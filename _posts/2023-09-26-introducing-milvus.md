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
- type: "vector-database"
  name: "milvusdatasource"
  configuration:
    service: "milvus"
    clientid: "{{{ secrets.milvus.clientid }}}"
    secret: "{{{ secrets.milvus.secret }}}"
    database: "{{{ secrets.milvus.database }}}"
    environment: "{{{ secrets.milvus.environment }}}"
```
2. Update the secrets.yaml file to include the credentials for the Milvus or Zilliz service

    ```
    milvus:
        clientid: "YourMilvusClientID"
        secret: "YourMilvusSecret"
        database: "YourMilvusDatabase"
        environment: "YourMilvusEnvironment"
    ```

Please note: 
- `clientid` is the client ID provided by your Milvus service.
- `secret` is the secret provided by your Milvus service.
- `database` is the database name provided by Milvus.
- `environment` defines the environment you are working on. 

## Example: Using Milvus for Querying and Writing Vector Data

With the vector-database configured for Milvus, you're ready to start performing semantic similarity queries across the vectors in the database with the help of `query-vector-db` agent and write vector embeddings using `vector-db-sink` agent. 

Here is how to set up your pipeline for vector querying, assuming the configuration above:

```
- name: "execute query"
  type: "query-vector-db"
  configuration:
    datasource: "milvusdatasource"
    query: |
    {
    "vector": ?,
    "topk": 5,
    "filter": {
    "$or": [{"genre": "comedy"}, {"year":2019}]}
    }
```

And this is how you can setup the pipeline for writing vectors:

```
- name: "write to milvus"
  type: "vector-db-sink"
  input: "input-topic"
  configuration:
    datasource: "milvusdatasource"
    mapping: "id=value.id,description=value.description,name=value.name"
```


Stay tuned for updates about more such integrations that provide more flexibility for building and running Gen AI applications. Please send us feedback on this new integration or LangStream in general in [Slack](https://join.slack.com/t/langstream/shared_invite/zt-21leloc9c-lNaGLdiecHuWU5N31L2AeQ) or [Linen](https://www.linen.dev/invite/langstream). If you find a bug, please open a [GitHub issue](https://github.com/LangStream/langstream/issues).
