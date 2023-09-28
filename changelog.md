---
title: LangStream Changelog
description: Notes from releases
---

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

