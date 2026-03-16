> This repository was created and refactored by the Manus AI agent.

# Universal RAG Pipeline Builder

This repository provides a detailed, step-by-step workflow and code examples for building a production-ready RAG (Retrieval-Augmented Generation) system from scratch, based on a modern RAG architecture. It is designed to be easily followed by a developer or an AI coding assistant (e.g., Claude, Codex, Gemini).

## Core Architecture

This project is based on the architecture outlined in the diagram below. It aims to build a powerful and observable system by combining proven open-source tools.

![RAG System Architecture](./assets/architecture.jpg)

## How to Use This Repository

This repository is most effective when used with an AI coding assistant. You can provide the entire project as context to the AI and automate the development by giving it high-level goals.

### With an AI Coding Assistant (Claude, Codex, Gemini, etc.)

1.  **Download or Clone the Repository**:
    ```bash
    git clone https://github.com/cryptaliac/rag-pipeline-builder.git
    cd rag-pipeline-builder
    ```

2.  **Provide Project Context**: Give your AI assistant the entire project folder as context (e.g., using a VSCode plugin for Claude, Cursor IDE, etc.).

3.  **Assign the Task**: Instruct the AI to follow the `BUILD_WORKFLOW.md` file to execute the steps.

    > **Example Prompt**:
    > "This project contains a workflow for building a RAG pipeline. Please read the `BUILD_WORKFLOW.md` file and start with the first step: 'Step 1: Data Ingestion & Processing'. Follow the instructions in `references/01_ingestion.md` to write and execute the necessary Python scripts."

### For Manual Development

1.  Clone the repository and open the `BUILD_WORKFLOW.md` file.
2.  Follow the five steps in the specified order.
3.  For each step, refer to the corresponding markdown file in the `references/` directory for code snippets and detailed explanations.

## The Build Workflow (`BUILD_WORKFLOW.md`)

The core of this project is the 5-step process defined in `BUILD_WORKFLOW.md`.

| Step | Reference File | Description |
| :--- | :--- | :--- |
| **1. Ingestion** | `references/01_ingestion.md` | Load source documents, split them into text chunks, and convert them into vector embeddings. |
| **2. Storage** | `references/02_storage.md` | Store the processed text chunks and vectors in a vector database like OpenSearch. |
| **3. Retrieval** | `references/03_retrieval.md` | Implement a hybrid search (combining keyword and semantic search) to find the most relevant documents for a user's query. |
| **4. Generation** | `references/04_generation.md` | Use a locally-run LLM (via Ollama) to generate a natural language answer based on the retrieved context. |
| **5. API & Observability** | `references/05_api_and_observability.md` | Expose the entire pipeline as a FastAPI endpoint and integrate LangFuse to monitor and debug the system's behavior. |

## Directory Structure

```
. (root)
├── BUILD_WORKFLOW.md                 # The main workflow for the AI or developer
├── README.md                         # Project overview and usage instructions (this file)
├── LICENSE                           # MIT License
├── assets/
│   └── architecture.jpg              # The architecture diagram
└── references/                       # Detailed guides and code examples for each step
    ├── 01_ingestion.md
    ├── 02_storage.md
    ├── 03_retrieval.md
    ├── 04_generation.md
    └── 05_api_and_observability.md
```

## Prerequisites

To run the code examples in this workflow, you need Python 3.8+ and the following libraries. Using a virtual environment is strongly recommended.

```bash
# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate

# Install core dependencies
pip install llama-index fastapi uvicorn python-dotenv

# Install dependencies for architectural components
pip install llama-index-vector-stores-opensearch llama-index-llms-ollama llama-index-embeddings-jinaai langfuse llama-index-callbacks-langfuse
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
