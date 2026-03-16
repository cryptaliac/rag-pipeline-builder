# Step 1: 데이터 수집 및 처리 (Ingestion)

이 단계의 목표는 원본 데이터(e.g., PDF, Markdown, HTML)를 가져와 검색에 용이한 형태로 가공하여 저장하는 것입니다. 다이어그램의 `Data processing Pipeline (Ingestion)` 부분에 해당합니다.

## 핵심 프로세스

1.  **데이터 로드 (Fetch & Parse)**: 지정된 소스에서 데이터를 불러옵니다.
2.  **분할 (Chunk)**: 불러온 문서를 의미 있는 작은 단위(청크)로 나눕니다.
3.  **임베딩 (Generate daily report)**: 각 청크를 벡터(숫자 배열)로 변환합니다.

## 1. 데이터 로드 (Loaders)

다양한 형식의 데이터를 로드하기 위해 `LlamaIndex`나 `LangChain`에서 제공하는 데이터 로더(Loader)를 사용하는 것이 편리합니다.

**예시: PDF 파일 로드**

```python
# 필요 라이브러리 설치
# pip install llama-index

from llama_index.core import SimpleDirectoryReader

# "./data" 디렉토리 아래의 모든 문서를 로드합니다.
reader = SimpleDirectoryReader("./data")
documents = reader.load_data()

print(f"{len(documents)}개의 문서를 로드했습니다.")
```

| 로더 종류 | 사용 사례 |
| --- | --- |
| `SimpleDirectoryReader` | 로컬 디렉토리의 파일(PDF, MD, DOCX 등) |
| `BeautifulSoupWebReader` | 특정 웹 페이지의 콘텐츠 |
| `GithubRepositoryReader` | GitHub 레포지토리의 코드 및 문서 |

> **[팁]** 다이어그램의 `Airflow`는 이 로딩 스크립트를 주기적으로 (e.g., 매일 자정) 실행하여 새로운 데이터를 자동으로 수집하는 역할을 합니다. 초기에는 간단한 `cron` 작업으로 시작할 수 있습니다.

## 2. 문서 분할 (Splitters)

LLM의 컨텍스트 윈도우 크기 제한과 검색 정확도를 위해, 긴 문서는 반드시 작은 청크로 분할해야 합니다.

**예시: 문단 기준으로 분할**

```python
from llama_index.core.node_parser import SentenceSplitter

# 문단, 문장, 토큰 등 다양한 기준으로 분할 가능합니다.
# 청크 크기와 경계(overlap)를 설정하는 것이 중요합니다.
node_parser = SentenceSplitter(
    chunk_size=1024,  # 청크의 최대 토큰 수
    chunk_overlap=20   # 인접 청크 간의 중복 토큰 수
)

nodes = node_parser.get_nodes_from_documents(documents)

print(f"{len(nodes)}개의 노드(청크)로 분할되었습니다.")
```

> **[팁]** `chunk_size`는 사용하는 LLM의 컨텍스트 크기와 임베딩 모델을 고려하여 결정해야 합니다. 너무 크면 검색 결과에 노이즈가 많아지고, 너무 작으면 의미가 파편화될 수 있습니다. 일반적으로 512 또는 1024를 시작점으로 테스트하는 것이 좋습니다.

## 3. 임베딩 (Embeddings)

분할된 각 텍스트 청크를 의미를 나타내는 벡터로 변환합니다. 이 벡터는 나중에 질문 벡터와 비교하여 유사도를 계산하는 데 사용됩니다.

**예시: Jina AI 임베딩 모델 사용**

다이어그램에서는 `Jina`를 사용했지만, `OpenAI`, `HuggingFace` 등 다양한 임베딩 모델을 사용할 수 있습니다.

```python
# 필요 라이브러리 설치
# pip install llama-index-embeddings-jinaai

from llama_index.embeddings.jinaai import JinaEmbedding
from llama_index.core import Settings

# JINA_API_KEY 환경변수 설정 필요
embed_model = JinaEmbedding(model="jina-embeddings-v2-base-en")

# LlamaIndex 전역 설정에 임베딩 모델 지정
Settings.embed_model = embed_model

# 이제부터 생성되는 모든 노드는 자동으로 이 임베딩 모델을 사용합니다.
# (실제 임베딩 과정은 다음 단계인 Storage에서 인덱스를 생성할 때 일어납니다)
```

이제 데이터 수집 및 전처리가 완료되었습니다. 다음 단계인 **[02_storage.md](02_storage.md)** 에서는 이렇게 생성된 노드(청크와 벡터)를 데이터베이스에 저장하는 방법을 알아봅니다.
