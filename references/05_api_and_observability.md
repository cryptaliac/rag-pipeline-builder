# Step 5: API Layer & Observability

The goal of this final step is to expose the RAG pipeline as an API for external applications and to establish observability to trace and analyze the system's behavior. This corresponds to the `API Layer`, `Client Interface`, and `Observability Layer` in the diagram.

## 1. API Layer

`FastAPI` is a modern, fast Python web framework widely used for serving AI/ML models. By wrapping the RAG query engine in a `FastAPI` endpoint, you can easily call it from other services or clients.

**Example: RAG endpoint using FastAPI**

```python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel
# Assume the query engine from Steps 1-4 is loaded.
# from rag_pipeline import query_engine

app = FastAPI()

class QueryRequest(BaseModel):
    query: str

class QueryResponse(BaseModel):
    answer: str
    sources: list

@app.post("/ask", response_model=QueryResponse)
async def ask_question(request: QueryRequest):
    """Receives a user query and returns an answer and sources via the RAG pipeline."""
    response = query_engine.query(request.query)
    
    # Process source information
    source_list = []
    for node in response.source_nodes:
        source_list.append({
            "file_name": node.metadata.get("file_name"),
            "page": node.metadata.get("page_label"),
            "score": node.score
        })

    return QueryResponse(answer=response.response, sources=source_list)

# To run the server: uvicorn main:app --reload
```

> **[Tip]** The `Redis` component in the diagram can be used to manage `FastAPI`'s asynchronous tasks or to cache answers for frequently asked questions, which improves response times and reduces costs.

## 2. Client Interface

Once the API is ready, you need a user interface for interaction. The diagram shows a `Telegram` bot and a `Gradio` web UI as examples.

-   **Telegram**: Use the Telegram Bot API to create a chatbot that interacts with the RAG system.
-   **Gradio**: Quickly create a simple web-based demo UI with just a few lines of code, which is very useful for prototyping.

## 3. Observability Layer

`LangFuse` is an open-source observability tool for LLM applications. It allows you to trace every step of the RAG pipeline, enabling you to visually analyze which documents were retrieved, what prompt was used, and what final answer the LLM generated.

**Example: Integrating LangFuse with LlamaIndex**

```python
# Required libraries
# pip install langfuse llama-index-callbacks-langfuse

from llama_index.core.callbacks import CallbackManager
from llama_index.callbacks.langfuse import LangfuseCallbackHandler
from llama_index.core import Settings

# Create a handler for LangFuse integration
# Requires LANGFUSE_SECRET_KEY, LANGFUSE_PUBLIC_KEY, LANGFUSE_HOST environment variables
langfuse_callback_handler = LangfuseCallbackHandler()

# Add the handler to the global LlamaIndex callback manager
Settings.callback_manager = CallbackManager([langfuse_callback_handler])

# From now on, all operations executed via LlamaIndex (retrieval, generation, etc.)
# will be automatically logged to LangFuse.
```

> **[Important]** Establishing observability is essential for testing the system, diagnosing problems, and improving performance. For complex systems, integrating tools like `LangFuse` from the beginning is highly beneficial in the long run.

---

This completes all the steps for building the RAG pipeline. Return to the main workflow file, **[BUILD_WORKFLOW.md](../BUILD_WORKFLOW.md)**, to review the entire process and apply it to your project.
