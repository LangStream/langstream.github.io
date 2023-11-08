---
title: Building Scalable Vectorization Pipelines in LangStream
categories:
image: /images/vectorization-pipeline.png
author_staff_member: enrico
date: November 8, 2023
---

The most fundamental component of a retrieval augmented generation (RAG)-based application is your data.
You store your data in a vector database as documents and query it to retrieve the most relevant documents for a given input text.

A typical pipeline that brings your data to the vector database is like this:
- Read data from a source (e.g. an S3 bucket, a WebSite, a Kafka topic, etc.)
- Process the data to extract the text to vectorise
- Split the text into chunks of a given size
- Compute vector embeddings for each chunk
- Write the chunks to the vector database
- Clean up obsolete data from the vector database


Now, let’s look at an example of all these steps in a LangStream pipeline:

```yaml
name: "Extract and manipulate text"
assets:
  - name: "documents-table"
    asset-type: "jdbc-table"
    creation-mode: create-if-not-exists
    config:
      table-name: "documents"
      datasource: "JdbcDatasource"
      create-statements:
        - |
          CREATE TABLE documents (
          filename TEXT,
          chunk_id int,
          num_tokens int,
          text TEXT,
          embeddings_vector FLOATA,
          PRIMARY KEY (filename, chunk_id));
pipeline:
  - name: "Read from S3"
    type: "s3-source"
    configuration:
      ....
  - name: "Extract text"
    type: "text-extractor"
  - name: "Normalise text"
    type: "text-normaliser"
  - name: "Split into chunks"
    type: "text-splitter"
    configuration:
      splitter_type: "RecursiveCharacterTextSplitter"
      chunk_size: 400
      chunk_overlap: 100
      length_function: "cl100k_base"
  - name: "Convert to structured data"
    type: "document-to-json"
    configuration:
        text-field: text
        copy-properties: true
  - name: "prepare-structure"
    type: "compute"
    configuration:
      fields:
         - name: "value.filename"
           expression: "properties.name"
           type: STRING
         - name: "value.chunk_id"
           expression: "properties.chunk_id"
           type: STRING
         - name: "value.chunk_num_tokens"
           expression: "properties.chunk_num_tokens"
           type: STRING
  - name: "compute-embeddings"
    id: "step1"
    type: "compute-ai-embeddings"
    configuration:
      model: "${secrets.open-ai.embeddings-model}"
      embeddings-field: "value.embeddings_vector"
      text: "{{ value.text }}"
      batch-size: 10
      flush-interval: 500
  - name: "Delete stale chunks"
    type: "query"
    configuration:
      datasource: "JdbcDatasource"
      when: "fn:toInt(properties.text_num_chunks) == (fn:toInt(properties.chunk_id) + 1)"
      mode: "execute"
      query: "DELETE FROM documents WHERE filename = ? AND chunk_id > ?"
      output-field: "value.delete-results"
      fields:
        - "value.filename"
        - "fn:toInt(value.chunk_id)"
  - name: "Write"
    type: "vector-db-sink"
    configuration:
      datasource: "JdbcDatasource"
      table-name: "documents"
      fields:
        - name: "filename"
          expression: "value.filename"
          primary-key: true
        - name: "chunk_id"
          expression: "value.chunk_id"
          primary-key: true
        - name: "embeddings_vector"
          expression: "fn:toListOfFloat(value.embeddings_vector)"
        - name: "text"
          expression: "value.text"
        - name: "num_tokens"
          expression: "value.chunk_num_tokens"
```


Let's look at all the key components of this pipeline.

## Assets

LangStream allows you to define assets that can be used by the pipeline. Assets are stored in the LangStream database and can be shared across pipelines. Assets can be of different types, such as JDBC tables, Cassandra Keyspaces, Milvus collections, etc.

When you deploy your application the LangStream runtime will create the assets if they don't exist yet, and when you uninstall the application the assets will be deleted. These behaviors can be configured using the `creation-mode` and `deletion-mode` parameters.

## Reading from an S3 Bucket

In the example above we read the data from an S3 bucket. The `s3-source` agent reads the data from the bucket and emits a document for each file.
LangStream handles recovery automatically and guarantees at-least once processing of each document.

Even if the document is split into chunks down the pipeline, LangStream will guarantee that when all the chunks of a document have been written to the vector database
the source is notified and the document is marked as processed.
This way, even if you have a failure in the middle of the pipeline and you have to restart, the source will only emit the documents that have not been processed yet. This allows you to deal with failures and restarts.

This is an IO-intensive operation, but it normally doesn't use much compute resources.

## Extracting the text

From the documents that are emitted by the source we extract the text using the `text-extractor` agent. This agent uses the Apache Tika library to extract the text from the document. The text is stored in a field called `value.text` together with other metadata, which is stored in the `properties` of the record.

This operation is CPU-intensive and may require some amount of memory, depending on the size of the documents.

## Text splitting

There are hard constraints on the number of tokens that the LLM can process. An what is a token depends on the algorithm used by the LLM.
With LangStream you can split the text into chunks of a given size. The `text-splitter` agent splits the text into chunks and emits a document for each chunk.
The algorimh selected in the example is the RecursiveCharacterTextSplitter, which splits the text into chunks of a given size, and then splits each chunk into smaller chunks of the same size, until the chunk size is less than the given size.
The size of the chunk is computed using the function `cl100k_base` that is the same used by OpenAI and measures the number of tokens in the chunk.

## Computing embeddings

When it is time to compute the embeddings you have two ways: call an external service or compute the vector locally.
If you call an external service, then you have to take into account a few things:

- the service is a remote service, so you have to take into account the latency and network failures
- most services allow to perform batch operations, so you can send multiple requests in a single call
- you can usually send multiple requests in parallel
- this operation is IO bound, and you don't need much CPU or memory locally

In case you use the local machine for computing the embeddings than the problem is different:
- you need machines with powerful CPUs or GPUs to compute the embeddings
- you may want to scale horizontally to increase the throughput

With LangStream, thanks to the streaming bus and to Kubernetes you can tune your pipeline and application at runtime and address all of the problems above:

- you can implement micro-batching
- you can deal with failures and retry
- you can scale horizontally to increase the throughput
- you can request to execute the agents on more powerful machines (e.g. with GPUs or more memory)

All of the feature can be tuned indepenently for each agent in the pipeline.

Some interesting characteristics of the pipeline above are:

- dealing with network failure to the external service is independent from writing to the vector database
- the size of batches sent to the external service in independent from the size of the batches written to the vector database
- the machines uses to compute embeddings can be different from the ones who perform text extraction and manipulation
- each step in the pipeline can be recovered independently and you don't need to replay the whole pipeline in case of failures (this could save you probably a lot of money if you are working with large data sets)

## Writing to the vector database

With LangStream you can connect natively to the most popular vector databases, from Astra DB to Milvus, from OpenSearch to Cassandra, from Pinecone to Elasticsearch.
But you can also leverage the Apache Kafka Connect ecosystem to integrate with your favourite vector database.

LangStream deals automatically with failures and retries, and it guarantees at-least once processing of each chunk and it is able to perform micro-batching to increase the throughput.

## Dealing with document updates

Another important fact to take into consideration in a vectorization pipeline is that the documents may change over time.
In the example above we are writing the chunks to a documents table, and you have two main cases:
- the new version is longer
- the new version of the document is shorter


If the new version is longer then it is easy as the new chunks will override the old ones.
But if the new version is shorter then you have to deal with the old chunks that are not part of the new version, and this is pretty easy to do as you can delete the chunks with an id that is greater than the number of chunks in the new version of the document.

This is pretty easy to implement in LangStream, as you can see in the example above.
Each agent can emit metadata that can be used by the next agents in the pipeline.

Therefore agents can emit events to the pipeline or trigger actions in other pipelines to deal with the changes in the documents, although this is not needed in this case.

## Metrics

It is important to monitor the performance of your pipeline, and LangStream provides a set of metrics that you can use to monitor the performance of your pipeline.
Metrics are exported to Prometheus and you can use Grafana to visualise them.

This is the result of running the vectorization pipeline over a corpus of HTML documents.
As you can see the source emits 106 documents and the sink receives 356 record, that means that each document has been split into 3 chunks on average.

![Pipeline](/images/pipeline_input_output.png)

In this image you can see the costs of calling the OpenAI embeding service.

The pipeline executed 96 calls to the OpenAI embedding service, to compute embeddings over 356 chunks of text, that means that the system automatically batched the calls to the service.
In total we sent 128148 tokens to OpenAI.

![OpenAI costs](/images/pipeline_openai_grafana.png)

With a streaming pipeline like this you could tune the batch size, and the parallelism of the calls to the OpenAI embeddings service in order to tune the pipeline.


## Conclusion

Writing scalable vectorization pipelines is not easy, but with LangStream you can focus on the business logic and let LangStream deal with the rest.
With an event-driven architecture you can scale your pipelines horizontally and you can tune the throughput of each agent independently.


Please send us feedback in [Slack](https://join.slack.com/t/langstream/shared_invite/zt-21leloc9c-lNaGLdiecHuWU5N31L2AeQ){:target="_blank"} or [Linen](https://www.linen.dev/invite/langstream){:target="_blank"}. If you find a bug, please open a [GitHub issue](https://github.com/LangStream/langstream/issues){:target="_blank"}.
