# Step 3: Retrieval

The goal of this step is to quickly and accurately find the most relevant document chunks from the database built in [Step 2](02_storage.md) when a user query is received. This corresponds to the `Hybrid Retrieval Pipeline` section of the diagram.

## Core Process

1.  **Create a Query Engine**: Build a query engine from the stored index to perform searches.
2.  **Set up Hybrid Search**: Combine keyword-based search and semantic vector search to improve retrieval accuracy.
3.  **Execute Search and Review Results**: Perform a search with a sample query and check the results.

## 1. Creating a Query Engine

The `VectorStoreIndex` object can easily create a query engine via its `as_query_engine()` method. This engine is the primary component that takes a user query and performs the search.

```python
# Assume the index object from Step 2 is available.
# from index import index

query_engine = index.as_query_engine()
```

## 2. Setting Up Hybrid Search

One of the key features of the architecture is **hybrid search**. This strategy combines two different search methods to compensate for their weaknesses and maximize their strengths.

-   **Vector Search**: Excels at abstract, meaning-focused queries like "the future of AI." It works by converting the query itself into a vector and comparing its cosine similarity with the stored document vectors.
-   **Keyword Search (BM25)**: Strong for queries containing specific terms or proper nouns, such as "LangFuse paper." It's an algorithm that improves upon TF-IDF, calculating the frequency and importance of keywords within a document.

`OpenSearch` supports both types of search. The `OpensearchVectorStore` in `LlamaIndex` allows you to easily enable hybrid search mode.

**Example: Creating a query engine with hybrid search**

```python
from llama_index.core import VectorStoreIndex
from llama_index.vector_stores.opensearch import OpensearchVectorStore

# OpenSearch connection details
endpoint = "http://localhost:9200"
index_name = "arxiv_papers"

# Set hybrid_search=True
vector_store = OpensearchVectorStore(
    endpoint=endpoint, 
    index_name=index_name, 
    hybrid_search=True
)

# Reload the existing index
index = VectorStoreIndex.from_vector_store(vector_store=vector_store)

# Create a query engine that performs hybrid search
# similarity_top_k: The final number of documents to return
query_engine = index.as_query_engine(similarity_top_k=5)
```

> **[Tip]** The `RRF (Reciprocal Rank Fusion)` in the diagram is an effective method for combining the rankings from both search results (BM25 and Vector) to produce a final, more robust ranking. Recent versions of `LlamaIndex` support this fusion logic internally within the `VectorStoreQueryEngine` or allow for custom implementations.

## 3. Executing a Search

Now you can ask a question using the created query engine.

```python
query = "What are the latest advancements in large language models?"
response = query_engine.query(query)

# The response object contains the answer text and the source nodes.
print("Response:", response)

# Check the source documents that were the basis for the answer
for node in response.source_nodes:
    print("Source Node Score:", node.score)
    print(node.get_content())
```

The retrieved `source_nodes` are the key pieces of information that will be provided as context to the LLM in the next step, **[04_generation.md](04_generation.md)**.
