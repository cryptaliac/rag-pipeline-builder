_# Step 2: Storage_

The goal of this step is to permanently store the nodes (text chunks and vector embeddings) created in [Step 1](01_ingestion.md) in a searchable database. This corresponds to the `Storage` layer in the diagram, which uses `PostgreSQL` and `OpenSearch`.

## Core Process

1.  **Set up a Vector Store**: Prepare a database to store and retrieve vector data.
2.  **Create Index and Ingest Data**: Inject the prepared nodes into the vector store to build a searchable index.

## 1. Choosing and Setting Up a Vector Store

The diagram uses `OpenSearch` as the vector store. `OpenSearch` is an open-source search engine derived from `Elasticsearch`. It supports both keyword-based text search and vector similarity search (k-NN), making it particularly powerful for **hybrid search**.

-   **PostgreSQL**: Serves as a relational database to store the original text of the documents or their metadata (e.g., source, creation date).
-   **OpenSearch**: Acts as a vector database to store the vector embeddings of the text chunks and perform fast similarity searches.

`LlamaIndex` supports integration with various vector stores. To use `OpenSearch`, you need to install the relevant library.

```bash
# Required library
pip install llama-index-vector-stores-opensearch
```

## 2. Creating an Index and Ingesting Data

`VectorStoreIndex` is the core object that connects your nodes to the vector store and ingests the data to make it searchable.

**Example: Creating an index in OpenSearch**

```python
from llama_index.core import VectorStoreIndex, StorageContext
from llama_index.vector_stores.opensearch import OpensearchVectorStore

# Assume the nodes created in Step 1 are ready.
# from nodes import nodes

# OpenSearch client endpoint
# You can install it locally via Docker or use a cloud service.
endpoint = "http://localhost:9200"

# Name of the index where data will be stored
index_name = "arxiv_papers"

# Create the vector store object
vector_store = OpensearchVectorStore(
    endpoint=endpoint, 
    index_name=index_name
)

# Assign the vector store to the storage context
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# Create the index
# At this point, the text in the nodes is converted to vectors by the embedding model,
# and both the text and vectors are stored in OpenSearch.
index = VectorStoreIndex(nodes, storage_context=storage_context)

print(f"Finished ingesting data into the '{index_name}' index.")
```

> **[Tip]** You can use `VectorStoreIndex.from_documents()` to handle document loading, splitting, embedding, and storage in a single call. However, explicitly separating each step makes the pipeline easier to understand and debug.

Now the data is stored and ready for retrieval. In the next step, **[03_retrieval.md](03_retrieval.md)**, we will learn how to find the most relevant documents for a user's query.
