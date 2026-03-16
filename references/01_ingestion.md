# Step 1: Data Ingestion & Processing

The goal of this step is to fetch raw data (e.g., from PDFs, Markdown, HTML), process it, and prepare it for retrieval. This corresponds to the `Data processing Pipeline (Ingestion)` section of the architecture diagram.

## Core Process

1.  **Load Data (Fetch & Parse)**: Load data from the specified source.
2.  **Split (Chunk)**: Split the loaded documents into smaller, meaningful units (chunks).
3.  **Embed**: Convert each chunk into a vector (an array of numbers).

## 1. Data Loading (Loaders)

Using data loaders from libraries like `LlamaIndex` or `LangChain` is convenient for loading data from various formats.

**Example: Loading PDF files**

```python
# Required library
# pip install llama-index

from llama_index.core import SimpleDirectoryReader

# Load all documents from the "./data" directory.
reader = SimpleDirectoryReader("./data")
documents = reader.load_data()

print(f"Loaded {len(documents)} documents.")
```

| Loader Type | Use Case |
| --- | --- |
| `SimpleDirectoryReader` | Files in a local directory (PDF, MD, DOCX, etc.) |
| `BeautifulSoupWebReader` | Content of a specific web page |
| `GithubRepositoryReader` | Code and documents from a GitHub repository |

> **[Tip]** The `Airflow` component in the diagram is responsible for running this loading script periodically (e.g., daily at midnight) to automatically ingest new data. You can start with a simple `cron` job for this.

## 2. Document Splitting (Splitters)

Due to the context window size limits of LLMs and for retrieval accuracy, long documents must be split into smaller chunks.

**Example: Splitting by sentence**

```python
from llama_index.core.node_parser import SentenceSplitter

# You can split by various criteria like paragraph, sentence, or token.
# Setting the chunk size and overlap is crucial.
node_parser = SentenceSplitter(
    chunk_size=1024,  # Max tokens per chunk
    chunk_overlap=20   # Overlapping tokens between adjacent chunks
)

nodes = node_parser.get_nodes_from_documents(documents)

print(f"Split into {len(nodes)} nodes (chunks).")
```

> **[Tip]** The `chunk_size` should be determined based on the context window of your LLM and the embedding model. If it's too large, the retrieval results might be noisy. If it's too small, the semantic meaning can become fragmented. Starting with 512 or 1024 is a good practice.

## 3. Embedding

Each text chunk is converted into a vector that represents its semantic meaning. This vector is later used to calculate similarity with a query vector.

**Example: Using the Jina AI Embedding Model**

The diagram uses `Jina`, but you can use various other embedding models from `OpenAI`, `HuggingFace`, etc.

```python
# Required library
# pip install llama-index-embeddings-jinaai

from llama_index.embeddings.jinaai import JinaEmbedding
from llama_index.core import Settings

# Requires JINA_API_KEY environment variable
embed_model = JinaEmbedding(model="jina-embeddings-v2-base-en")

# Set the embedding model in the global LlamaIndex settings
Settings.embed_model = embed_model

# From now on, all created nodes will automatically use this embedding model.
# (The actual embedding process happens during index creation in the next step)
```

Data ingestion and preprocessing are now complete. In the next step, **[02_storage.md](02_storage.md)**, we will learn how to store these generated nodes (chunks and vectors) in a database.
