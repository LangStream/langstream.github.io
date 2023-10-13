---
title: Building a Q&A RAG chatbot with conversational awareness
categories:
image: /images/chatbot.jpeg
author_staff_member: chris
date: October 13, 2023
---

You want to build a chatbot that answers questions about private data. For example, your chatbot taps into your company's documentation and other knowledge sources to help support agents answer questions from your customers. Or maybe it’s a support chatbot to directly answer questions from your users.

For this type of chatbot, you are going to want to use the [Retrieval Augmented Generation (RAG)](https://www.datastax.com/guides/what-is-retrieval-augmented-generation) pattern. LLMs have been trained on public data up to a training date and have a limited context window, or how much additional data you can provide them. RAG works around these limitations by storing all the context in a database, which can scale well beyond the context window limits. You put your private documents or updated versions of the public data in a database, index that information using vector embeddings and then perform similarity search to find the most relevant documents to include in the LLM context. Using this technique, you can narrow down your large set of context data to the most relevant set that will fit in the LLM context window.

The RAG pattern is a proven technique that allows LLMs to provide fresh answers about topics it was not trained on. With the proper prompt engineering, it also reduces hallucinations by grounding the LLM response in data relevant to the question. This is all good, but your users are interacting with ChatGPT on a daily basis and they expect the chatbot to remember the details of the conversation. They are using pronouns in their questions or referring to specific details in the last answer. They are conditioned to expect the chatbot to understand these references because this is how normal human conversation works and how ChatGPT works.

However, this conversational awareness doesn’t come for free. LLMs are stateless and have no native awareness of the conversation. You need to provide the chat history data to the LLM so it can become conversationally aware.

In this blog post, I am going to show you two different ways to add conversational awareness to a chatbot that uses the RAG pattern. I am going to use LangStream to illustrate how to do this. However, conversational awareness is supported in other frameworks, such as LangChain (see the [RetrievalQAChain](https://js.langchain.com/docs/api/chains/classes/RetrievalQAChain) and the  [ConversationalRetrievalQAChain](https://js.langchain.com/docs/modules/chains/popular/chat_vector_db)) and the concepts are the same.

## Requirements for conversational awareness

Before diving into the solution, let’s consider the requirements for conversational awareness. First, since the LLMs will not remember the conversation, you will need to store the history of the conversation to give to the LLM. You could store the history in memory, but it will be lost if the process restarts, so it makes sense to save the chat history in some sort of database.

With the conversation history stored in a database, you will need to retrieve it so you can include it in the prompt when you call the LLM. This can be a simple query from the database where the results get added to the prompt. Keep in mind that the conversation history will consume some of the available context window of the LLM. Also keep in mind that you need to insert relevant data into the context window as part of the RAG pattern, so you can’t afford to include a lot of conversation history, especially for LLMs that have small context windows (for example, 4096 tokens). 

One solution to this problem is to summarize the conversation history using the LLM or to summarize the oldest part of the conversation and include the most recent history. Describing those techniques would require their own blog post on handling conversation history. For the purposes of this blog post, we are simply going to insert only the last N instances of the conversation history to try and control how much of the context window is used by conversation history.

The last consideration of conversation memory is how to prevent the history from growing indefinitely. This is particularly important if you have a large number of users and you need to keep track of the conversation history for each (typically done by user session). If you don’t control the size, query times will eventually become slow and the storage requirements can become quite large.

If you are using memory for history, you can simply restart the process to clear the memory, but you will lose all conversation history, old and new. If you are using a database, you will need some mechanism to remove the oldest part of the conversation history from the database. In our solutions, we are using Cassandra (Astra DB) to store the history and take advantage of its time-to-live (TTL) feature, which will remove rows from the database after a certain time.

## Configuring the conversation history table

We will use LangStream assets to configure the Cassandra table we will use for storing the conversation history.  Assets are a handy feature of LangStream. They represent configuration as code and make the dependencies clear. When deploying a LangStream application, it will create any assets that don’t already exist.

Here is the asset configuration for the conversation history table:

``` yaml
assets:
  - name: "convo-memory-table"
    asset-type: "cassandra-table"
    creation-mode: create-if-not-exists
    config:
      table-name: "${globals.chatTable}"
      keyspace: "${globals.vectorKeyspace}"
      datasource: "AstraDatasource"
      create-statements:
        - >
          CREATE TABLE IF NOT EXISTS ${globals.vectorKeyspace}.${globals.chatTable} (
            sessionId TEXT,
            timestamp TIMEUUID,
            question TEXT,
            answer TEXT,
            prompt TEXT,
            PRIMARY KEY (sessionId, timestamp)
          )
          WITH CLUSTERING ORDER BY (timestamp DESC)
          AND default_time_to_live = 3600;
```

Notice that we are creating the table with a composite key including the session ID for the conversation and a timestamp. This will allow for a LIMIT query to return the most recent exchanges in the conversation. Each row of the table includes the question asked by the user and the answer provided by the LLM. We also include the details of the prompt sent to the LLM. This is for debugging and diagnostic purposes and it is not necessary to include it. 

Since Cassandra supports TTL by column, we are setting the default TTL on all columns to 3600 seconds (1 hour). As long as none of the columns of a row are changed, after 1 hour the row will be deleted. This configuration ensures that our Cassandra table doesn’t grow indefinitely large. If you have a large number of user sessions, you may want to use an even smaller TTL value.

## Writing the history

Once the table is created, we can write the conversation history to it. Obviously this occurs at the end of the pipeline after the exchange with the LLM but it makes sense to discuss it now. On the first question of the session, the history returned will necessarily be empty, so it only comes into play after the chatbot has responded to at least one question.

Here is the section of the LangStream pipeline that writes the conversation result to the Cassandra table:

``` yaml
  - name: "Write conversation history to Astra"
    type: "vector-db-sink"
    input: "${globals.logTopic}"
    configuration:
      datasource: "AstraVector"
      table: "${globals.vectorKeyspace}.${globals.chatTable}"
      mapping: "sessionid=value.sessionid,question=value.questionNoContext,answer=value.answer,prompt=value.prompt,timestamp=now()"
```

The `vector-db-sink` agent maps the field from the message to the database columns (sessionid, question, answer, prompt). It also generates a timestamp when writing the row using the now() function. This is a feature of the Cassandra `vector-db-sink` (which is based on the [Kafka Connect connector](https://docs.datastax.com/en/astra-serverless/docs/manage/upload/streaming-data-with-the-datastax-apache-kafka-connector.html)).

## Simple solution to make the chatbot conversationally aware

Now that we have the conversation history stored in a Cassandra table, we can use it to make the chatbot conversationally aware. The simple solution is to retrieve the conversation history from the database before constructing the prompt, add it to the prompt, and instruct the LLM to consider the history when responding . Here is part of a LangStream pipeline that does just that:

``` yaml
{% raw %}
pipeline:
  - name: "Query Chat History"
    id: query-chat-history
    type: "query"
    configuration:
      datasource: "AstraDatasource"         
      query: "select question,answer from ${globals.vectorKeyspace}.${globals.chatTable} where sessionid = ? limit 3"
      output-field: "value.history"
      fields:
        - "value.sessionid"    
  - name: "compute-embeddings"
    id: "compute-embeddings"
    type: "compute-ai-embeddings"
    configuration:
      model: "text-embedding-ada-002"
      embeddings-field: "value.question_embeddings"
      text: "{{ value.question }}"
  - name: "lookup-related-documents-in-llm"
    type: "query"
    configuration:
      datasource: "AstraDatasource"
      query: "SELECT text FROM ${globals.vectorKeyspace}.${globals.vectorTable} ORDER BY embeddings_vector ANN OF ? LIMIT 4"
      fields:
        - "value.question_embeddings"
      output-field: "value.related_documents"
  - name: "ai-chat-completions"
    type: "ai-chat-completions"
    output: "${globals.logTopic}"
    configuration:
      model: "${globals.chatModelName}"
      completion-field: "value.answer"
      log-field: "value.prompt"
      stream-to-topic: "${globals.answersTopic}"
      stream-response-completion-field: "value"
      min-chunks-per-message: 20
      messages:
        - role: system
          content: |
              You are a helpful assistant for ${globals.assistantType}. 

              A user is going to ask a question. Refer to the Related Documents below 
              when answering their question. Use them as much as possible
              when answering the question. If you do not know the answer, say so.

              Do not answer questions not related to ${globals.assistantType}.

              When answering questions, take into consideration the history of the 
              chat converastion, which is listed below under Chat History. The chat history 
              is in reverse chronological order, so the most recent exhange is at the top.
              
              Related Documents:
              ==================

              {{# value.related_documents}}
              {{ text}}
              {{/ value.related_documents}}

              Chat History:
              =============
              {{# value.history}}
              User: {{ question }}  Assistant: {{ answer}}
              -----------------------------------------------
              {{/ value.history}}
        - role: user
          content: "{{ value.question}}"
{% endraw %}

```


In the `query-chat-history` agent, we query the Cassandra table for the last 3 rows from the conversation history using a LIMIT query. Because the table has a composite key consisting of the session ID and timestamp, we get the most recent 3 results from the table for that session. We then include those results, which are stored in the `history` field, in the prompt using the mustache templating notation:

``` yaml
{% raw %}
              Chat History:
              =============
              {{# value.history}}
              User: {{ question }}  Assistant: {{ answer}}
              -----------------------------------------------
              {{/ value.history}}
{% endraw %}
```


We also include instructions in the prompt instruction (“When answering questions, take into consideration the history…”) the LLM to consider the chat history when responding to the question.

## RAG problem

This solution will work and provide conversational awareness to your chatbot. However, our chatbot is using the RAG pattern. In the pipeline snippet above there is also a query of the vector database for documents that are semantically similar to the question. This is done using a vector embedding of the question which was computed earlier in the pipeline.

To compute the vector embedding, we are using the question as it was entered by the user. Let’s imagine what would happen if the user question was:

> Tell me more about that?

This question likely refers to something in the last response from the LLM. Because we include the chat history in the prompt for the LLM and chat models like GPT-4 have been fine tuned for chat conversations, it will likely figure out what “that” refers to and respond correctly.

However, the similarity search is not that sophisticated. Without knowing what “that” refers to, it is hard to get relevant results from the vector database using similarity search. In fact, you will probably just get random matches (text is that is semantically similar to the word "that"), since there is very little explicit semantic meaning in the question. We will then insert these random results in the prompt as “related documents”. In this scenario, the RAG pattern doesn’t improve the response. In fact, it may make it worse, by confusing the LLM with random text that we have told it is relevant.

## Better solution to make the chatbot conversationally aware

So how do we fix this? The answer is to do another call to the LLM, one that creates a new version of the question that takes into account the conversation history. You then use this context aware question to compute a vector embedding and use that to search the vector database. 

In the scenario above, “that” will be resolved to whatever it refers to in the prior conversation by this additional LLM call. Even if the LLM doesn’t get it exactly right–for example, it resolves “that” to the wrong concept–it will most likely replace it with some concept from the conversation history. That will get a contextually relevant result that may not be what the user wanted, but at a minimum will trigger the user to rephrase the question with something less ambiguous than “that”. Exactly how, by the way, this would go in a conversation with an actual person. 

Here’s the part of a LangStream pipeline that will ask the LLM to rephrase the question from the user taking the conversation history into account:

``` yaml
{% raw %}
  - name: "Update question based on chat history"
    type: "ai-chat-completions"
    configuration:
      model: "${globals.chatModelName}" 
      completion-field: "value.question"
      log-field: "value.chatHistoryPrompt"
      stream: false
      messages:
        - role: system
          content: |
              You are a conversational interpreter for a conversation between a user and 
              a bot who is an expert on ${globals.assistantType}.
              
              The user will give you a question without context. You will reformulate the question
              to take into account the context of the conversation. You should assume the question
              is related to ${globals.assistantType}. You should also consult with the Chat History
              below when reformulating the question. For example,
              you will substitute pronouns for mostly likely noun in the conversation
              history. 
              
              When reformulating the question give higher value to the latest question and response
              in the Chat History. The chat history is in reverse chronological order, so the most 
              recent exchange is at the top.

              Only respond with the reformulated question. If there is no chat history, then respond 
              only with the question unchanged.

              Chat History:
              =============
              {{# value.history}}
              User: {{ question}}  Assistant: {{ answer}}
              -----------------------------------------------
              {{/ value.history}}
        - role: user
          content: "{{ value.questionNoContext}}"
{% endraw %}
```

This takes the question as input by the user (`questionNoContext`) and has the LLM reword it and store it in the question field. This updated question can then be used when querying the vector database for semantically similar information, resulting in higher quality results. This context-aware question can also be used for the final LLM call that provides the response to the end user. By using and improved question, the LLM will give an improved answer.

An additional advantage of this approach that I have found is that the chatbot is much less likely to provide an off-topic answer. Although you have prompted the LLM to only answer questions from a certain domain in the simple solution, it sometimes misinterprets the question and gives an off-topic answer. By adding the LLM call to reframe the question in the context of the conversation and the domain, you get a question that is less likely to induce an off-topic answer. 

Obviously, a downside to this approach is that it requires two LLM calls to answer the user's question. This increases the cost for LLM tokens and also the time it takes to generate an answer. The latency of the answer (as it appears to the user) can be mitigated by streaming the ultimate response from the LLM (which is enabled by default in LangStream) but additional time is unavoidable. The pipeline needs to wait for the full response from the LLM call to rephrase the question before it can proceed, which adds to the overall latency of the response.

## Conclusion

Adding conversation awareness to a RAG chatbot may seem like a simple thing to do. However, it involves many considerations. How are you going to store the history of the conversation? How are you going to retrieve the history of the conversation? How much of the conversation are you going to consider, given limited context windows for LLMs? How do you make sure the stored conversation history doesn’t grow forever, eventually slowing down the application?

Once you have these questions answered, you need to consider how the conversation history impacts the RAG retrieval. The question as input by the user may not contain a lot of semantic meaning, since the user may, naturally, refer back to the previous responses in the conversation. Users are conditioned to do this because that’s how human conversation works and how popular tools, such as ChatGPT, work. 

However, the raw user question can give poor semantic search results. This can lead to the RAG pattern sending random context to the LLM, which can confuse the LLM and lead to poor responses. A solution to this problem is to make an additional call to the LLM to rephrase the question, taking the conversational history into consideration. This will lead to better responses from your chatbot, but it has additional costs (for additional LLM tokens) and adds to the time-to-first-response to the user. 


You can see an example of a complete LangStream application using the better solution in the LangStream project GitHub repo [here](https://github.com/LangStream/langstream/tree/main/examples/applications/chatbot-rag-memory).
