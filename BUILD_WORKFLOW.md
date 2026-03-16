_# RAG Pipeline Construction Workflow

This document outlines a complete, step-by-step workflow for building a production-ready RAG (Retrieval-Augmented Generation) system. It is designed to be followed by an AI code assistant or a developer to construct the pipeline from scratch.

## Prerequisites

Before starting, ensure a Python environment is set up. It is highly recommended to use a virtual environment to manage dependencies.

```bash
# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate

# Install core dependencies
pip install llama-index fastapi uvicorn python-dotenv

# Install dependencies for specific components shown in the architecture
pip install llama-index-vector-stores-opensearch llama-index-llms-ollama llama-index-embeddings-jinaai langfuse llama-index-callbacks-langfuse
```

## Core Workflow

Building the RAG pipeline involves five sequential steps. Follow them in order. For detailed instructions and code examples for each step, refer to the corresponding markdown file in the `references/` directory.

### 1. Step 1: Data Ingestion & Processing

**Goal**: Load raw documents, parse them, split them into text chunks (nodes), and generate vector embeddings.

**Action**: Read and execute the instructions in **[references/01_ingestion.md](references/01_ingestion.md)**.

### 2. Step 2: Storage

**Goal**: Store the processed text chunks and their vector embeddings in a specialized database for efficient retrieval.

**Action**: Read and execute the instructions in **[references/02_storage.md](references/02_storage.md)**.

### 3. Step 3: Retrieval

**Goal**: Given a user query, retrieve the most relevant text chunks from the vector store using hybrid search.

**Action**: Read and execute the instructions in **[references/03_retrieval.md](references/03_retrieval.md)**.

### 4. Step 4: Generation

**Goal**: Use a Large Language Model (LLM) to generate a coherent answer based on the retrieved text chunks.

**Action**: Read and execute the instructions in **[references/04_generation.md](references/04_generation.md)**.

### 5. Step 5: API & Observability

**Goal**: Expose the RAG pipeline as a web API and integrate observability tools to trace and monitor the system's behavior.

**Action**: Read and execute the instructions in **[references/05_api_and_observability.md](references/05_api_and_observability.md)**.
