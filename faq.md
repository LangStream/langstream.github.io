---
title: FAQ
description: Frequently Asked Questions
---

## New to Gen AI and LangStream

### What is LangStream?
LangStream is an open-source project designed to facilitate the creation of streaming Gen AI applications. It combines event-based architectures with the latest Gen AI technologies. With LangStream, developers can declare their Gen AI components, and it will handle deploying them as an event-based streaming application.

### How does LangStream help in the development of Gen AI applications?
LangStream allows you to build applications that capitalize on Large Language Models (LLMs). It ensures that data from various sources, both static and streaming, is available for constructing high-quality prompts for the LLMs, making the AI response more accurate and contextually relevant.

### How is LangStream different than LangChain or LlamaIndex?

 LangStream is complementary to LangChain and LlamaIndex, but is different in several ways:
  * Includes a run-time environment for your applications leveraging proven tech like Kubernetes
  * Is event-driven, which provides advantages around scaling, fault tolerance, and extensibility
  * Includes no-code, config-driven agents for common tasks, such as calling an embedding model API, in addition to supporting Python agents using LangChain and LlamaIndex


### What are LangStream application agents?
LangStream application agents operate on event data. They modify or transform this data before passing it on to the next agent in the pipeline. LangStream comes with several pre-built agents for tasks like AI actions, data transformation, text processing, and input/output.

### Can I customize LangStream to my needs?
Absolutely! While LangStream offers pre-built agents for common tasks, you can also write custom agents using Python, providing a mix-and-match capability to meet your unique requirements.

### What's the role of Kubernetes and Kafka in LangStream?
When you deploy an application using LangStream, it leverages Kubernetes and Apache Kafka for the backend infrastructure. Kafka manages the data flow through topics, and Kubernetes handles the application's deployment and scaling. Both tools offer proven stability and reliability.

### What kind of observability does LangStream offer for my applications?
LangStream is designed with a high level of abstraction but ensures that logs and metrics are available for debugging purposes. It supports Prometheus metrics and offers tooling for accessing Kubernetes logs.

### Is LangStream open-source?
Yes, LangStream is open source. You can find the project on [GitHub](https://github.com/LangStream/langstream) and even contribute to its development.

### I've built a prototype using a popular Gen AI library. Can I use LangStream to turn it into an event-driven application?
Definitely. LangStream supports popular Gen AI libraries like LangChain and LlamaIndex. You can easily convert your prototype into an event-driven application using LangStream.

### What are the advantages of an event-driven architecture for Gen AI applications?
Event-driven architectures offer scalability, real-time processing, loose coupling for easy evolution, and improved fault tolerance. Especially for Gen AI applications, this architecture can harness real-time data, enhance performance, and ensure the system can recover from failures efficiently.

### How can I get started with LangStream?
To begin with LangStream, you can check out its [GitHub repository](https://github.com/LangStream/langstream). There's also a [Visual Studio code extension](https://marketplace.visualstudio.com/items?itemName=DataStax.langstream)available for LangStream, which offers a seamless experience directly from your IDE.

### Where can I seek help or give feedback on LangStream?
The LangStream team is active on [Slack](https://join.slack.com/t/langstream/shared_invite/zt-21leloc9c-lNaGLdiecHuWU5N31L2AeQ). You can join the channel to ask questions, share feedback, or engage in discussions about the project.

## New to LangStream but not GenAI

### Does LangStream support RAG (retrieval augmented generation) techniques?
Yes. LLMs work with the knowledge embedded during their training and might not be updated with the latest information.  RAG combines the capabilities of powerful LLMs with external retrieval or database systems, enabling them to pull in up-to-date information or specific facts from vast datasets. LangStream makes it easy to bring all of your data to GenAI. See some examples in our docs [here](https://docs.langstream.ai/building-applications/rag-pattern). 

### My prompt doesn't yield the exact desired response from the LLM what can I try?

Fine-tuning the instructions given to the model can lead to more accurate or contextually relevant outputs. This is especially useful in task-specific applications where precision is required. 

Using LangStream you can combine the benefits of an event driven architecture with LLMs to do Instruction Fine-tuning to achieve better results. Here are some example use cases


* __Dynamic Task Automation__. In workflow tools, you can fine-tune instructions as tasks progress. For instance, in a content generation pipeline, initial drafts could be streamed and modified instructions sent back for revisions.


* __Feedback-driven Modeling__. As users interact with the generated content, their feedback can be streamed back to fine-tune the generation instructions, making the AI's output more aligned with user needs over time.

### What is least-to-most prompting and can I do it with LangStream?
Absolutely, LangStream makes this easy. Typically this technique is used to gradually refine prompts from being general to specific based on model responses. It allows for better control over the model's outputs by guiding it step-by-step to the desired answer. Here are some example use cases:

* __Adaptive User Support__. Based on the streamed user behavior, the model can decide the level of prompting required. If a user seems to struggle, the model can offer more detailed prompts or guidance.


* __Learning Applications__. In educational tools, real-time monitoring of student progress can help the system decide when to provide hints, prompts, or full solutions.

### What are some potential use cases that become possible with an event driven architecture and generative AI?

If you're leveraging models like RAG (Retrieval-Augmented Generation) and focusing on features like chain-of-thought reasoning and self-consistency, it could lead to more dynamic and responsive systems.

Here are some potential use cases and benefits of combining generative AI with event streaming:

* __Real-time Content Generation__. With the event stream, content can be generated in real-time based on the most recent data. For example, producing news summaries from a continuous stream of events or generating real-time analysis reports.

* __Dynamic Conversational Agents__. Agents that can adapt their responses based on real-time data. For instance, a customer support bot that provides real-time updates on issues.

* __Adaptive Learning Systems__. Systems that can adjust their content and teaching techniques in real-time based on feedback from students or changes in the curriculum.

* __Real-time Analytics and Reports__. Instead of just displaying numbers and graphs, have detailed textual explanations generated on-the-fly, providing deeper insights into the data.

* __Intelligent Monitoring__. Monitor logs, alerts, or any other event streams and generate human-readable summaries or detect anomalies.

* __Improved Chain-of-thought Reasoning__. By leveraging event streams, the model can be made aware of the "history" or "context" of previous interactions, allowing for more coherent and continuous conversations or analyses.

* __Self-consistency and Learning__. If the model can monitor its outputs (as a form of feedback loop through the event stream), it can attempt to maintain consistency in its answers or even adjust based on feedback.

### Does LangStream support FLARE?
Yes. FLARE leverages few-shot learning, allowing models to understand and adapt to new tasks with just a few examples. The retrieval component can also help in pulling specific information to assist with these tasks.

FLARE is easy to implement with LangStream with added benefits of providing real time updates to a retrieval database. Here are a few example use cases of FLARE + LangStream:

* __Event-driven Knowledge Retrieval__. If the retrieval model is connected to a live feed of news or updates, it can pull the most recent information to answer queries.

* __Few-shot Adaptations__. By streaming new examples in real-time, the model could adapt on-the-fly to specific tasks provided in a few-shot manner.

### How does LangStream enable Research and Revise techniques?
Sometimes, the initial answer provided by the model may not be optimal or might lack depth. The research and revise approach makes the model first generate an initial response, then "research" (re-evaluate its knowledge) and refine or revise its answer for better accuracy or depth.

The sequence in which you set up agents in your pipelines and ensure you’re writing out the right data enables research and revision. See this example in our documentation where one application uses the log output of another application to proactively reach out to you. Her are a few example use cases research and revise can enable using an event driven architecture with LangStream:

* __Iterative Content Generation__. Imagine a system that drafts content, then revises based on feedback streamed in real-time. Users could suggest revisions or clarifications, and the model could instantly re-draft.

* __Collaborative Research__. Multiple users could input data points, queries, or facts through the stream. The model then synthesizes these into comprehensive reports or answers.

### My chat bot is losing track of context in a more complex task. How can I address this?
Consider using the chain of thought reasoning method. This technique emphasizes maintaining a consistent line of reasoning across a multi-turn conversation or complex task. It helps the model keep context and build upon previous interactions. 

### Often my application is providing very different answers for semantically identical questions that are worded slightly differently, what can I do?
Consider using self-consistency techniques. LangStream’s architecture supports this well due to the benefits of it’s event driven architecture.  By promoting self-consistency, models are trained to ensure that they remain consistent in their responses regardless of slight variations in query phrasing.


