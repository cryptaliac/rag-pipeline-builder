---
name: rag-pipeline-builder
description: Build a production-ready RAG (Retrieval-Augmented Generation) pipeline from scratch. Use this skill to create a question-answering system based on your own documents, following a step-by-step process from data ingestion to API deployment.
---

# RAG Pipeline Builder

This skill provides a complete, step-by-step workflow for building a production-ready RAG (Retrieval-Augmented Generation) system based on the architecture diagram provided by the user. It guides you from processing raw documents to deploying a fully functional, observable API.

## Prerequisites

Before starting, ensure the following core Python libraries are installed. You may need additional libraries for specific data loaders or vector stores.

```bash
sudo pip3 install llama-index fastapi uvicorn python-dotenv
# For specific components, e.g., OpenSearch, Ollama, Jina
sudo pip3 install llama-index-vector-stores-opensearch llama-index-llms-ollama llama-index-embeddings-jinaai langfuse llama-index-callbacks-langfuse
```

## Core Workflow

Building the RAG pipeline involves five sequential steps. Follow them in order. For detailed instructions and code examples for each step, refer to the corresponding markdown file in the `references/` directory.

### 1. Step 1: Data Ingestion & Processing

**Goal**: Load raw documents, parse them, split them into text chunks (nodes), and generate vector embeddings.

**Instructions**: Read **[references/01_ingestion.md](references/01_ingestion.md)** for a complete guide on data loading, splitting, and embedding.

### 2. Step 2: Storage

**Goal**: Store the processed text chunks and their vector embeddings in a specialized database for efficient retrieval.

**Instructions**: Read **[references/02_storage.md](references/02_storage.md)** for instructions on setting up a vector store like OpenSearch and indexing your data.

### 3. Step 3: Retrieval

**Goal**: Given a user query, retrieve the most relevant text chunks from the vector store using hybrid search (a combination of vector and keyword search).

**Instructions**: Read **[references/03_retrieval.md](references/03_retrieval.md)** to learn how to build a query engine and implement an effective retrieval strategy.

### 4. Step 4: Generation

**Goal**: Use a Large Language Model (LLM) to generate a coherent answer based on the retrieved text chunks (context) and the user's query.

**Instructions**: Read **[references/04_generation.md](references/04_generation.md)** for guidance on prompt engineering and integrating an LLM (like one from Ollama) for response generation.

### 5. Step 5: API & Observability

**Goal**: Expose the RAG pipeline as a web API and integrate observability tools to trace and monitor the system's behavior.

**Instructions**: Read **[references/05_api_and_observability.md](references/05_api_and_observability.md)** to learn how to build a FastAPI endpoint and connect it to monitoring tools like LangFuse.
