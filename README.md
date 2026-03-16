> 이 문서는 Manus AI 에이전트에 의해 생성 및 리팩토링되었습니다.

# 범용 RAG 파이프라인 빌더 (Universal RAG Pipeline Builder)

이 레포지토리는 최신 RAG(Retrieval-Augmented Generation) 아키텍처를 기반으로, 프로덕션 수준의 질의응답 시스템을 처음부터 구축할 수 있는 상세한 단계별 워크플로우와 코드 예제를 제공합니다. 개발자 또는 AI 코딩 어시스턴트(예: Claude, Codex, Gemini)가 쉽게 따라 할 수 있도록 설계되었습니다.

## 핵심 아키텍처

본 프로젝트는 아래 다이어그램에 명시된 아키텍처를 기반으로 합니다. 검증된 오픈소스 도구들을 조합하여 강력하고 관찰 가능한 시스템을 구축하는 것을 목표로 합니다.

![RAG System Architecture](./assets/architecture.jpg)

## 이 레포지토리를 사용하는 방법

이 레포지토리는 AI 코딩 어시스턴트와 함께 사용할 때 가장 효과적입니다. 전체 프로젝트를 AI에게 제공하고, 고수준의 목표를 지시하여 개발을 자동화할 수 있습니다.

### AI 코딩 어시스턴트와 함께 사용하기 (Claude, Codex, Gemini 등)

1.  **레포지토리 다운로드 또는 클론**:
    ```bash
    git clone https://github.com/cryptaliac/rag-pipeline-builder.git
    cd rag-pipeline-builder
    ```

2.  **프로젝트 컨텍스트 제공**: 사용 중인 AI 어시스턴트에게 전체 프로젝트 폴더를 컨텍스트로 제공합니다. (예: VSCode의 Claude 플러그인, Cursor IDE 등)

3.  **작업 지시**: `BUILD_WORKFLOW.md` 파일을 참조하여 단계별로 작업을 지시합니다.

    > **예시 프롬프트**:
    > "이 프로젝트는 RAG 파이프라인을 구축하기 위한 워크플로우를 담고 있습니다. `BUILD_WORKFLOW.md` 파일을 읽고, 첫 번째 단계인 'Step 1: Data Ingestion & Processing'부터 시작해주세요. `references/01_ingestion.md` 파일의 지침에 따라 필요한 Python 스크립트를 작성하고 실행해주세요."

### 개발자가 직접 사용하는 경우

1.  **레포지토리 클론** 후, `BUILD_WORKFLOW.md` 파일을 엽니다.
2.  파일에 명시된 5가지 단계를 순서대로 따라갑니다.
3.  각 단계별로 `references/` 디렉토리에 있는 해당 마크다운 파일을 참고하여 코드 스니펫을 활용하고 설명을 따릅니다.

## 빌드 워크플로우 (`BUILD_WORKFLOW.md`)

이 프로젝트의 핵심은 `BUILD_WORKFLOW.md` 파일에 정의된 5단계 프로세스입니다.

| 단계 | 참조 파일 | 설명 |
| :--- | :--- | :--- |
| **1. 데이터 수집** | `references/01_ingestion.md` | 원본 문서를 로드하고, 검색에 용이하도록 텍스트 청크로 분할한 뒤 벡터로 변환합니다. |
| **2. 저장** | `references/02_storage.md` | 처리된 텍스트 청크와 벡터를 OpenSearch와 같은 벡터 데이터베이스에 저장합니다. |
| **3. 검색** | `references/03_retrieval.md` | 키워드 검색과 의미 기반 벡터 검색을 결합한 하이브리드 검색을 구현하여 사용자 질문에 가장 관련성 높은 문서를 찾습니다. |
| **4. 생성** | `references/04_generation.md` | 검색된 컨텍스트를 기반으로 Ollama를 통해 로컬에서 실행되는 LLM이 자연스러운 답변을 생성하도록 합니다. |
| **5. API 및 관찰** | `references/05_api_and_observability.md` | 전체 파이프라인을 FastAPI 엔드포인트로 노출하고, LangFuse를 연동하여 시스템 동작을 모니터링하고 디버깅합니다. |

## 디렉토리 구조

```
. (root)
├── BUILD_WORKFLOW.md                 # AI 또는 개발자를 위한 메인 워크플로우
├── README.md                         # 프로젝트 개요 및 사용법 (이 파일)
├── LICENSE                           # MIT 라이선스
├── assets/
│   └── architecture.jpg              # 아키텍처 다이어그램
└── references/                       # 각 단계별 상세 가이드 및 코드 예제
    ├── 01_ingestion.md
    ├── 02_storage.md
    ├── 03_retrieval.md
    ├── 04_generation.md
    └── 05_api_and_observability.md
```

## 실행 환경 요구사항

워크플로우의 코드 예제를 실행하기 위해서는 Python 3.8 이상 및 아래 라이브러리들이 필요합니다. 가상 환경 사용을 강력히 권장합니다.

```bash
# 가상 환경 생성 및 활성화
python -m venv venv
source venv/bin/activate

# 핵심 라이브러리 설치
pip install llama-index fastapi uvicorn python-dotenv

# 아키텍처 구성요소 라이브러리 설치
pip install llama-index-vector-stores-opensearch llama-index-llms-ollama llama-index-embeddings-jinaai langfuse llama-index-callbacks-langfuse
```

## 라이선스

이 프로젝트는 MIT 라이선스를 따릅니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 확인하세요.
