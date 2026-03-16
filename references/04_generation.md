# Step 4: 답변 생성 (Generation)

이 단계의 목표는 [Step 3](03_retrieval.md)에서 검색된 관련성 높은 문서 조각(컨텍스트)들을 활용하여, 사용자의 질문에 대한 최종 답변을 생성하는 것입니다. 다이어그램의 `LLM Generation Layer`에 해당합니다.

## 핵심 프로세스

1.  **프롬프트 템플릿(Prompt Template) 정의**: 검색된 컨텍스트와 사용자 질문을 LLM에게 전달할 양식을 정의합니다.
2.  **LLM(Large Language Model) 설정**: 답변 생성을 수행할 LLM을 지정합니다.
3.  **응답 생성 및 후처리(Post-processing)**: LLM으로부터 받은 응답을 정제하고 출처 정보를 포함시킵니다.

## 1. 프롬프트 템플릿 정의

프롬프트 템플릿은 RAG 시스템의 성능에 가장 큰 영향을 미치는 요소 중 하나입니다. LLM이 제공된 컨텍스트만을 기반으로, 충실하게 답변을 생성하도록 유도하는 것이 핵심입니다.

**예시: LlamaIndex의 프롬프트 템플릿**

```python
from llama_index.core import PromptTemplate

# QA(질의응답)를 위한 프롬프트 템플릿
# {context_str} 에는 Step 3에서 검색된 문서 조각들이 들어갑니다.
# {query_str} 에는 사용자의 원본 질문이 들어갑니다.

qa_prompt_tmpl_str = (
    "Context information is below.\n"
    "---------------------\n"
    "{context_str}\n"
    "---------------------\n"
    "Given the context information and not prior knowledge, "
    "answer the query. If the context does not provide an answer, "
    "just say that you couldn't find an answer in the provided documents.\n"
    "Query: {query_str}\n"
    "Answer: "
)
qa_template = PromptTemplate(qa_prompt_tmpl_str)

# 생성된 템플릿을 쿼리 엔진에 업데이트합니다.
# query_engine은 Step 3에서 생성되었다고 가정합니다.
query_engine.update_prompts({"response_synthesis:text_qa_template": qa_template})
```

> **[팁]** "prior knowledge"를 사용하지 말라고 명시하고, 컨텍스트에 답이 없을 때의 행동을 정의해주는 것이 LLM의 환각(Hallucination)을 줄이는 데 매우 효과적입니다.

## 2. LLM 설정

다이어그램은 `Ollama`를 통해 로컬에서 LLM을 실행하는 방식을 보여줍니다. `Ollama`는 Llama 3, Mistral 등 다양한 오픈소스 LLM을 쉽게 설치하고 실행할 수 있게 해주는 도구입니다. 또는 `OpenAI`의 GPT 모델과 같은 API 기반 모델을 사용할 수도 있습니다.

**예시: Ollama를 LlamaIndex와 연동**

```python
# 필요 라이브러리 설치
# pip install llama-index-llms-ollama

from llama_index.llms.ollama import Ollama
from llama_index.core import Settings

# 로컬에서 실행 중인 Ollama에 연결하고 사용할 모델 지정
llm = Ollama(model="llama3", request_timeout=120.0)

# LlamaIndex 전역 설정에 LLM 지정
Settings.llm = llm

# 이제부터 모든 생성 작업은 이 LLM을 사용합니다.
```

## 3. 응답 생성 및 후처리

이제 모든 준비가 끝났습니다. 쿼리 엔진에 질문을 던지면, 내부적으로 **검색(Retrieval) → 프롬프트 구성 → 생성(Generation)** 의 전체 과정이 실행됩니다.

```python
# Step 3에서 설정한 하이브리드 검색 엔진을 사용합니다.
response = query_engine.query("What is RAG?")

# LLM이 생성한 답변 텍스트
print(response.response)

# 답변의 근거가 된 소스 노드(문서)의 메타데이터 확인
# 이를 통해 사용자에게 "출처 보기" 기능을 제공할 수 있습니다.
for node in response.source_nodes:
    print(f"Source: {node.metadata.get('file_name')}, Page: {node.metadata.get('page_label')}")
```

이제 RAG의 핵심 파이프라인이 완성되었습니다. 마지막으로 **[05_api_layer.md](05_api_layer.md)** 에서는 이 파이프라인을 외부에서 호출할 수 있는 API로 만들고 사용자 인터페이스와 연결하는 방법을 알아봅니다.
