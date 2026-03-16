# Step 5: API 계층 및 관찰 가능성 (API & Observability)

이 단계의 목표는 지금까지 만든 RAG 파이프라인을 외부 애플리케이션에서 사용할 수 있도록 API로 노출하고, 전체 시스템의 동작을 추적하고 분석할 수 있는 관찰 가능성(Observability)을 확보하는 것입니다. 다이어그램의 `API Layer`, `Client Interface`, `Observability Layer`에 해당합니다.

## 1. API 계층 (API Layer)

`FastAPI`는 현대적이고 빠른 파이썬 웹 프레임워크로, AI/ML 모델 서빙에 널리 사용됩니다. RAG 쿼리 엔진을 `FastAPI` 엔드포인트로 감싸면 다른 서비스나 클라이언트에서 쉽게 호출할 수 있습니다.

**예시: FastAPI를 사용한 RAG 앤드포인트**

```python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel
# Step 1~4에서 설정한 쿼리 엔진을 로드한다고 가정합니다.
# from rag_pipeline import query_engine

app = FastAPI()

class QueryRequest(BaseModel):
    query: str

class QueryResponse(BaseModel):
    answer: str
    sources: list

@app.post("/ask", response_model=QueryResponse)
async def ask_question(request: QueryRequest):
    """사용자 질문을 받아 RAG 파이프라인을 통해 답변과 출처를 반환합니다."""
    response = query_engine.query(request.query)
    
    # 출처 정보 가공
    source_list = []
    for node in response.source_nodes:
        source_list.append({
            "file_name": node.metadata.get("file_name"),
            "page": node.metadata.get("page_label"),
            "score": node.score
        })

    return QueryResponse(answer=response.response, sources=source_list)

# 서버 실행: uvicorn main:app --reload
```

> **[팁]** 다이어그램의 `Redis`는 `FastAPI`의 비동기 작업을 관리하거나, 자주 묻는 질문에 대한 답변을 캐싱(Caching)하여 응답 속도를 높이고 비용을 절감하는 데 사용될 수 있습니다.

## 2. 클라이언트 인터페이스 (Client Interface)

API가 준비되면, 사용자가 실제로 상호작용할 수 있는 UI가 필요합니다. 다이어그램은 `Telegram` 봇과 `Gradio` 웹 UI를 예시로 보여줍니다.

-   **Telegram**: 텔레그램 봇 API를 사용하여 채팅을 통해 RAG 시스템과 대화할 수 있습니다.
-   **Gradio**: 몇 줄의 코드만으로 간단한 웹 기반 데모 UI를 빠르게 만들 수 있어 프로토타이핑에 매우 유용합니다.

## 3. 관찰 가능성 계층 (Observability Layer)

`LangFuse`는 LLM 애플리케이션을 위한 오픈소스 관찰 가능성 도구입니다. RAG 파이프라인의 모든 단계를 추적(Trace)하여 어떤 문서가 검색되었고, 어떤 프롬프트가 사용되었으며, LLM이 최종적으로 어떤 답변을 생성했는지 시각적으로 분석할 수 있게 해줍니다.

**예시: LlamaIndex와 LangFuse 연동**

```python
# 필요 라이브러리 설치
# pip install langfuse llama-index-callbacks-langfuse

from llama_index.core.callbacks import CallbackManager
from llama_index.callbacks.langfuse import LangfuseCallbackHandler
from llama_index.core import Settings

# LangFuse 연동을 위한 핸들러 생성
# LANGFUSE_SECRET_KEY, LANGFUSE_PUBLIC_KEY, LANGFUSE_HOST 환경변수 설정 필요
langfuse_callback_handler = LangfuseCallbackHandler()

# LlamaIndex 전역 콜백 매니저에 핸들러 추가
Settings.callback_manager = CallbackManager([langfuse_callback_handler])

# 이제부터 LlamaIndex를 통해 실행되는 모든 작업(검색, 생성 등)은
# 자동으로 LangFuse에 기록됩니다.
```

> **[중요]** 관찰 가능성을 확보하는 것은 시스템을 테스트하고, 문제를 진단하며, 성능을 개선하는 데 필수적입니다. 복잡한 시스템일수록 초기 단계부터 `LangFuse`와 같은 도구를 도입하는 것이 장기적으로 큰 도움이 됩니다.

---

이것으로 RAG 파이프라인 구축의 모든 단계가 완료되었습니다. 이 스킬의 메인 파일인 **[SKILL.md](../SKILL.md)** 로 돌아가 전체 워크플로우를 다시 한번 확인하고, 실제 프로젝트에 적용해보세요.
