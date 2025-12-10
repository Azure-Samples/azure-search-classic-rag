# Readme for classic RAG in Azure AI Search

This series demonstrates the classic pattern for building RAG solutions on Azure AI Search. Classic RAG uses the original search engine pipeline instead of agentic retrieval, with no LLM integration except for at the end of the pipeline when you pass the search results to an LLM for answer formulation. In this series, you learn about the components, dependencies, and optimizations for maximizing relevance and minimizing costs in a classic RAG pattern.

We now recommend [**agentric retrieval**](https://learn.microsoft.com/azure/search/agentic-retrieval-overview) for modern RAG workloads. However, if you already have an existing solution built on Azure AI Search, a classic RAG pattern might meet your application requirements with fewer code changes.

Sample data is a [collection of PDFs](https://github.com/Azure-Samples/azure-search-sample-data/tree/main/nasa-e-book/earth_book_2019_text_pages) uploaded to Azure Storage. The content is from [NASA's Earth free e-book](https://www.nasa.gov/ebooks/earth/).

## Exercises in this series

- Set up your development environment. Choose your models for embeddings and chat. Learn how to structure an index for conversational search.

- Set up an indexing pipeline that loads, chunks, embeds, and ingests searchable content.

- Retrieve searchable content using queries and a chat model.

- Maximize relevance by adding semantic ranker and a scoring profile.

- Minimize vector storage and costs.

We omitted a few aspects of a RAG pattern to reduce complexity:

- No management of chat history and context. Chat history is typically stored and managed separately from your grounding data, which means extra steps and code. This exercise assumes atomic question and answers from the LLM and the default LLM experience.

- No per-user user security over results (what we refer to as "security trimming"). For more information and resources, start with [Document-level access overview](https://learn.microsoft.com/azure/search/search-document-level-access-overview) and make sure to review the links at the end of the article.
