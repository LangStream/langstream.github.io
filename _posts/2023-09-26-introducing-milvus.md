---
title: Support for Milvus as a Vector Database in LangStream 
categories:
image: 
author_staff_member: chris
date: September 26, 2023
---
We are happy to announce that as of release X LangStream now supports [Milvus](https://milvus.io) as a vector database. This integration broadens LangStream's support for vector database giving users more flexibility in their AI workflows.

Milvus is a popular open-source vector database. It is built for the cloud-native environment and is highly scalable. It supports vector similarity search and provides a wide range of similarity search algorithms. A cloud version of Milvus, called [Zilliz](https://zilliz.com/), is also available.

Here's a detailed walkthrough of how you can leverage Milvus/Zilliz with LangStream.

## Understanding Vector Databases in LangStream 


Vector databases form a important component of many Gen AI applications in LangStream. They store vector representations (embeddings) of various data, including text. By including search tools, vector databases enable similarity search on the vector representations, enabling the users to find semantically similar data in the database.

Vector databases are used in LangStream as part of the Retrieval Augmented Generation (RAG) applications. A LangStream application retrieves relevant documents or passages from a vector database based on their semantic relevance, providing context to the LLM for generating responses.

LangStream has native support for several databases, including Apache Cassandra, Datastax Astra DB, and Pinecone. Now, with support for Milvus, users have a broader range of options for their vector representation and similarity searching needs.

## Configuring Milvus as a Vector Database in LangStream

Suppose we want to use Milvus as a vector database in a LangStream application. Here is a simplified step-by-step process to configure it for use in LangStream. 

1. Create or update a configuration.yaml file to specify a resource of type `vector-database` with a service name of `milvus`. 

```
resources:
  - name: "MilvusDatasource"
    configuration:
      service: "milvus"
      ## OSS Milvus
      username: "{{{ secrets.milvus.username }}}"
      password: "{{{ secrets.milvus.password }}}"
      host: "{{{ secrets.milvus.host }}}"
      port: "{{{ secrets.milvus.port }}}"
      ## Set to "upsert" for OSS Milvus, on Zills use "delete-insert"
      write-mode: "{{{ secrets.milvus.write-mode }}}"
      ## Zillis
      url: "{{{ secrets.milvus.url }}}"
      token: "{{{ secrets.milvus.token }}}"
```
2. Update the secrets.yaml file to include the credentials for the Milvus or Zilliz service

    ```
    - name: milvus
      id: milvus
      data:
        username: ""
        password: ""
        host: ""
        port: ""
        write-mode: "upsert"  
        
    ```

In case you are using Zilliz service:


    ```
    - name: milvus
      id: milvus
      data:
        write-mode: "delete-insert"    
        url: ""
        token: ""

    ```

If you are on Milvus Cloud you need to set:
- url: the url of the Milvus Cloud instance (it should be something like https://milvus-cloud-xxxxx.milvuscloud.com)
- token: the token to use to authenticate

You can find the URL and the token in the Milvus Cloud console.

If you are on OSS Milvus you need to set:
- username: the username
- password: the password
- host: the host
- port: the port

## Example: Using Milvus for Querying and Writing Vector Data

With the vector-database configured for Milvus, you're ready to start performing semantic similarity queries across the vectors in the database with the help of `query-vector-db` agent and write vector embeddings using `vector-db-sink` agent. 

Here is how to set up your pipeline for vector querying, assuming the configuration above:

```
  - name: "lookup-related-documents-in-llm"
    type: "query-vector-db"
    configuration:
      datasource: "MilvusDatasource"
      query: |
        {
          "collection-name": "docs",
          "vectors": ?,
          "top-k": 10,
          "output-fields": ["text"]
        }
      fields:
        - "value.question_embeddings"
      output-field: "value.related_documents"
```

And this is how you can setup the pipeline for writing vectors:

```
  - name: "Write to Milvus"
    type: "vector-db-sink"
    input: chunks-topic
    configuration:
      datasource: "MilvusDatasource"
      collection-name: "docs"
      fields:
        - name: "filename_and_chunkid"
          expression: "fn:concat(value.filename, value.chunk_id)"
        - name: "vector"
          expression: "fn:toListOfFloat(value.embeddings_vector)"
        - name: "language"
          expression: "value.language"
        - name: "text"
          expression: "value.text"
        - name: "num_tokens"
          expression: "value.chunk_num_tokens"
```


Stay tuned for updates about more such integrations that provide more flexibility for building and running Gen AI applications. Please send us feedback on this new integration or LangStream in general in [Slack](https://join.slack.com/t/langstream/shared_invite/zt-21leloc9c-lNaGLdiecHuWU5N31L2AeQ) or [Linen](https://www.linen.dev/invite/langstream). If you find a bug, please open a [GitHub issue](https://github.com/LangStream/langstream/issues).
