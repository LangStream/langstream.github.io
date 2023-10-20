---
title: LangStream Changelog
description: Notes from releases
---
[Releases](https://github.com/LangStream/langstream/releases){:target="_blank"}

### 0.3.1 - Oct 20, 2023

Bugfixes:
* Fix CLI docker run on MacOS and Linux


### 0.3.0 - Oct 19, 2023


New features:
* **Amazon Bedrock Support**: LangStream now works seamlessly with Amazon Bedrock, allowing you to generate embeddings and perform text completions effortlessly.
* **Expanded Database Support**: Now, you can seamlessly integrate Apache Solr and OpenSearch as Vector Databases. This means you can efficiently index and retrieve embeddings.
* **Local Hugging Face Model Execution**: Running your favorite Hugging Face model on the LangStream executor is now a breeze. Just a few lines of YAML code and you're good to go.
* **Flexible SQL Queries for JDBC Databases**: You can now perform both read and write operations on JDBC databases using the new query agent. This adds a new level of flexibility to your pipelines.
* **New Flow Control Agents**: We've added three new agents - timer-source, trigger-event, and log-event - that help streamline data flow in your pipelines. These agents enhance the precision and control of your workflows.

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.3.0](https://github.com/LangStream/langstream/releases/tag/v0.3.0)



### 0.2.0 - Oct 12, 2023


New features:

* **Support for FLARE pattern**: FLARE is an emerging pattern, similar to RAG, that iteratively uses a prediction of the upcoming sentence to anticipate future content, which is then utilized as a query to retrieve relevant documents to regenerate the sentence if it contains low-confidence tokens. Read more in the [paper](https://arxiv.org/abs/2305.06983).
* **New agent 'dispatch'**: you can implement message routing easily by defining condition on the record.
* **New agent 'http-request'**: run http requests to enrich your data model from any public service or call webhooks when an event happens on the pipeline.
* **New agent 'azure-blob-storage-source'**: read and process files from Azure Blob Storage containers.
* **New UI for apps monitoring and testing**: you can now spins up a local UI webserver to monitor and test your application in a friendly way. 
* **CLI support for Windows**: CLI now is now installable on Windows.
* **Export your application as Mermaid graph**: run `langstream apps get myapp -o mermaid` to generate a [MermaidJS](https://mermaid.js.org/) graph that shows you the entire application workflow.

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.2.0](https://github.com/LangStream/langstream/releases/tag/v0.2.0)

### 0.1.0 - Oct 4, 2023

New features:

* **MMR based re-ranking agent**: read more details in the [documentation](https://docs.langstream.ai/pipeline-agents/text-processors/rerank) about the new `re-rank` agent.
* **API Reference**: all the agents, resources and assets have a new [API Reference](https://docs.langstream.ai/building-applications/api-reference) page.
* **Default global values**: you can now specify application default values for the global variables.
* **Embedded Vector Database HerdDB in mini-langstream**: you can run vector-based applications in mini-langstream without the need for an external vector database.

Breaking changes:
* **Refactor Python agent API**: the Python API is now more handy and structured. Checkout the [documentation](https://docs.langstream.ai/pipeline-agents/custom-agents/python-function) for getting started.
* **New templating system for globals and secrets**: for referring to a secret you should now use use *${secrets.my-secret.key}* instead of *{{{ secrets.my-secret.key }}}* . Note that the old syntax will continue to work but it's highly recommended to switch to the new one.   

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.1.0](https://github.com/LangStream/langstream/releases/tag/v0.1.0)



### 0.0.22 - Sep 27, 2023

Langstream 0.0.22 new features:


* **Topics replication factor**: when declaring a topic, you can now specify the replication factor with the property `topics.<TOPIC>.options.replication-factor: 3`.
* **Batched AI Embeddings**: the `compute-embeddings` is now fully async and uses batches to improve throughtput.
* **mini-langstream**: mini-langstream is a brand new command that creates a local LangStream cluster, based on Minikube. Try it with brew (MacOS): `brew install LangStream/langstream/mini-langstream && mini-langstream start`.
* **Milvus vector database support**: open source project [Milvus](https://milvus.io/) and his cloud version - Zillis - is now pluggable as vector database in your pipeline.
* **New agent `ai-text-completions`**: for autonomus agents you can now use the OpenAI and VertexAI text models. Have you heard about `gpt-3.5-turbo-instruct` or `text-bison`? Here you go. 
* **JDBC and vectors**: you can now write to vector databases that are JDBC compliants. Ready to store vectors on PostGRE? 
* **Multiple AI services in the same application**: now you can mix different AI service types/configuration to use different models in the same application.
* **JDBC table assets**: leverage `assets` to let LangStream handling the JDBC tables for you.
* **Embedded vector database in `langstream docker run`**: when you test the application on Docker, you can now startup an in-memory vector databases to verify your vector-based pipelines in a couple of seconds.
* **CLI**: allow updates to the default profile.
* **GRPC Python agents**: the python runtime has been rewritten to be faster, more stable and composable with Java agents.

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.0.22](https://github.com/LangStream/langstream/releases/tag/v0.0.22)


### 0.0.21 - Sep 19, 2023

Langstream 0.0.21 is a maintenance release and contains only core bugfixes about OpenAI client and Cassandra/Astra sink.

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.0.21](https://github.com/LangStream/langstream/releases/tag/v0.0.21)


### 0.0.20 - Sep 19, 2023

Langstream 0.0.20 new features:
* Web crawler agent has been improved and it can now works better with sitemaps, robots.txt and other nuances
* Introduce deletion-mode for assets. Assets can be deleted when the application is deleted.
* Core improvements like CLI chat output for streaming messages, usage of the OpenAI client

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.0.20](https://github.com/LangStream/langstream/releases/tag/v0.0.20)


### 0.0.19 - Sep 15, 2023

Langstream 0.0.19 new features:
* Support local run without passing secrets
* Fix CLI dependencies download and enable Cassandra Sink e2e test
* Introduce deletion-mode for topics. Topics can be deleted when the application is deleted.

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.0.19](https://github.com/LangStream/langstream/releases/tag/v0.0.19)


### 0.0.18 - Sep 14, 2023

Langstream 0.0.18 new features:
* `langstream run docker` command improvements
* Support writing messages with different schemas on Kafka topics

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.0.18](https://github.com/LangStream/langstream/releases/tag/v0.0.18)
### 0.0.17 - Sep 14, 2023

LangStream 0.0.17 new features:

* New chat gateway which simplifies the usage of the gateways
* Introduced support for Azure Blob Storage as code storage
* New tool for testing the application locally without Kubernetes

Full release notes: [https://github.com/LangStream/langstream/releases/tag/v0.0.17](https://github.com/LangStream/langstream/releases/tag/v0.0.17)

