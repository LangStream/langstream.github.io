---
title: Re-Ranking with Maximal Marginal Relevance in LangStream
categories:
image: /images/rerank.jpeg
author_staff_member: enrico
date: October 4, 2023
---

When implementing the retrieval augmented generation (RAG) pattern, it’s necessary to query a vector database to get documents related to the input text and add them to the prompt as context for the LLM. Although the returned documents will be semantically similar to the input text, they may not be the optimal set of documents to add to the prompt. For example, if the set of documents have duplicate content, providing the same document multiple times in the prompt will not improve the result from the LLM. In this case, you want the documents to be similar and diverse. This is where the re-ranking the results comes into play. As of [release 0.1.0](/changelog) LangStream includes a [re-rank agent](https://docs.langstream.ai/pipeline-agents/text-processors/rerank){:target="_blank"} to improve the quality results that are sent to the LLM.

One of the most commonly used algorithms for re-ranking is **maximal marginal relevance** (MMR). The idea behind MMR is to find the most diverse set of documents, while also keeping the most relevant ones. This is achieved by assigning each document a score based on its similarity to the input text, as well as its distance from the documents that have already been selected.

The default implementation of MMR in LangStream is based on the CMU paper “[Maximal Marginal Relevance to Re-rank Results in Unsupervised Keyphrase Extraction](https://www.cs.cmu.edu/~jgc/publication/The_Use_MMR_Diversity_Based_LTMIR_1998.pdf)”. It uses the BM25 algorithm to compute the similarity between the input text and the document text, as well as the cosine similarity between the query vector and the document vector. The BM25 algorithm requires a couple of parameters, k1 and b, which you can configure in the agent.

Now, let’s look at an example of how to use the re-rank agent in a LangStream pipeline. Here’s the full pipeline:

```yaml
{% raw %}
pipeline:
- name: “convert-to-structure”
 type: “document-to-json”
 input: “questions-topic”
 configuration:
  text-field: “question”
- name: “compute-embeddings”
 type: “compute-ai-embeddings”
 configuration:
  model: “{{{secrets.open-ai.embeddings-model}}}”
  embeddings-field: “value.question_embeddings”
  text: “{{% value.question }}”
  flush-interval: 0
- name: “lookup-related-documents”
 type: “query-vector-db”
 configuration:
  datasource: “jdbcdatasource”
  query: “select text,embeddings_vector from documents order by cosine_similarity(embeddings_vector, cast(? as float array)) desc limit 20"
  fields:
   - “value.question_embeddings”
  output-field: “value.related_documents”
- name: “re-rank documents with mmr”
 type: “re-rank”
 configuration:
  max: 5 # keep only the top 5 documents, because we have a hard limit on the prompt size
  field: “value.related_documents”
  query-text: “value.question”
  query-embeddings: “value.question_embeddings”
  output-field: “value.related_documents”
  text-field: “record.text”
  embeddings-field: “record.embeddings_vector”
  algorithm: “mmr”
  lambda: 0.5
  k1: 1.2
  b: 0.75
{% endraw %}
```

Let’s break it down. We start by converting the input document to a JSON structure using the `convert-to-structure` agent. Next, we compute the embeddings of the input text using the `compute-ai-embeddings` agent and store them in a field called “value.question_embeddings”. We then use the `query-vector-db` agent to query the vector database for the top 20 documents related to the input text, and store them in a field called “value.related_documents”.
Finally, we use the `re-rank` agent to further filter the top 5 documents by re-ranking them using the MMR algorithm.

The input to the re-rank agent is the “value.related_documents” field from the previous agent, and the output is also stored in the same field. We also specify the input and output fields for the text and embeddings, as well as the values for the MMR algorithm parameters.

## Maximal Marginal Relevance
MMR is an algorithm that ranks the retrieved documents using a weighted combination of two criteria: relevance and diversity. The relevance criterion is based on the similarity between the query and the document embeddings, whereas the diversity criterion measures the dissimilarity between the retrieved documents.
MMR computes a score for each document based on the following formula:
```
score(d) = λ * similarity(query, d)
           - (1 - λ) * max(similarity(d, di))
           for di in retrieved_documents
```
where Lambda (`λ`) is a weight that balances the relevance and diversity criteria and `di` is a previously considered document that was already ranked.
The formula favors documents that are both similar to the query and dissimilar to the previously considered documents.

There are two parameters to tune BM25 algorithm used to calculate the similarity:

* **k1 - Term Frequency Saturation**: The \(k1\) parameter controls how quickly the term weight reaches saturation. The higher the value of \(k1\), the quicker a term in a document reaches its maximum weight. Common values for \(k1\) range between 1.2 and 2.0.


* **b - Length Normalization**: The \(b\) parameter controls the extent to which the BM25 algorithm normalizes term weights by the length of the document. A \(b\) value of 1.0 indicates full normalization, while a value of 0.0 indicates no normalization. Common values for \(b\) usually range around 0.75.


## Conclusion
Re-ranking is a powerful technique that can significantly improve the quality of search results. By using the MMR algorithm, it allows you to filter and rank documents based on their relevance and diversity with respect to the input text. This allows for better context to insert in the prompt which leads to better results from the LLM.

Please send us feedback on this new integration or LangStream in general in [Slack](https://join.slack.com/t/langstream/shared_invite/zt-21leloc9c-lNaGLdiecHuWU5N31L2AeQ){:target="_blank"} or [Linen](https://www.linen.dev/invite/langstream){:target="_blank"}. If you find a bug, please open a [GitHub issue](https://github.com/LangStream/langstream/issues){:target="_blank"}.
