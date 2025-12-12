# Readme for classic RAG quickstarts

We offer simple, easy to follow quickstarts for learning the fundamentals of a pattern or feature. This collection of quickstarts consists of code samples for the classic RAG pattern in Azure AI Search, where classic RAG uses the original search engine pipeline instead of agentic retrieval. 

Key points about classic RAG include:

- LLM integration is used only for answer formulation over search results. There's no preliminary LLM query planning step (querydecomposition or rewriting), or built-in answer synthesis.
- Queries target a single search index on a search service.
- Queries are issued using the [Search Documents API](https://learn.microsoft.com/rest/api/searchservice/documents/search-post) (or SDK equivalent).
- You can use hybrid search (one text query with one or more vector queries) but the entire query payload must stay under the [API limits for a request](https://learn.microsoft.com/azure/search/search-limits-quotas-capacity#api-request-limits).

We now recommend [**agentric retrieval**](https://learn.microsoft.com/azure/search/agentic-retrieval-overview) for modern RAG workloads. However, if you already have an existing solution built on Azure AI Search, a classic RAG pattern might meet your application requirements with fewer code changes. For a comparison of classic RAG and agentic retrieval, see [RAG in Azure AI Search](https://learn.microsoft.com/azure/search/retrieval-augmented-generation-overview).

## In this series

The code in this series shows you how to send search results to a chat completion model. It assumes hotels sample data. You can use a different index, but we recommend that you [create your own hotels-sample index](https://learn.microsoft.com/azure/search/search-get-started-portal) if you want to use the code as-is.

- [REST](/quickstarts/rest/readme.md)

- [Python](/quickstarts/python/Quickstart-rag.ipynb)

- [.NET (C#)](/quickstarts/csharp/README.md)

- [Java](/quickstarts/java/README.md)

- [JavaScript](/quickstarts/javascript/README.md)

- [TypeScript](/quickstarts/typescript/README.md)