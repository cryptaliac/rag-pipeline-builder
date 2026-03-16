_# Step 2: 데이터 저장 (Storage)_

이 단계의 목표는 [Step 1](01_ingestion.md)에서 생성된 노드(텍스트 청크와 벡터 임베딩)를 검색이 가능한 데이터베이스에 영구적으로 저장하는 것입니다. 다이어그램의 `Storage` 계층에 해당하며, `PostgreSQL`과 `OpenSearch`를 사용합니다.

## 핵심 프로세스

1.  **벡터 저장소(Vector Store) 설정**: 벡터 데이터를 저장하고 검색할 데이터베이스를 준비합니다.
2.  **인덱스 생성 및 데이터 주입**: 준비된 노드를 벡터 저장소에 주입(insert)하여 인덱스를 구축합니다.

## 1. 벡터 저장소 선택 및 설정

다이어그램은 `OpenSearch`를 벡터 저장소로 사용합니다. `OpenSearch`는 전문 검색 엔진 `Elasticsearch`에서 파생된 오픈소스로, 키워드 기반의 텍스트 검색과 벡터 유사도 검색(k-NN)을 모두 지원하여 **하이브리드 검색**에 특히 강력합니다.

-   **PostgreSQL**: 문서의 원본 텍스트나 메타데이터(e.g., 출처, 생성일)를 저장하는 관계형 데이터베이스 역할을 합니다.
-   **OpenSearch**: 텍스트 청크의 벡터 임베딩 값을 저장하고, 빠른 유사도 검색을 수행하는 벡터 데이터베이스 역할을 합니다.

`LlamaIndex`는 다양한 벡터 저장소와의 연동을 지원합니다. `OpenSearch`를 사용하기 위해서는 관련 라이브러리를 설치해야 합니다.

```bash
# 필요 라이브러리 설치
pip install llama-index-vector-stores-opensearch
```

## 2. 인덱스 생성 및 데이터 주입

`VectorStoreIndex`는 노드를 벡터 저장소에 연결하고, 데이터를 주입하여 검색 가능한 상태로 만드는 핵심 객체입니다.

**예시: OpenSearch에 인덱스 생성**

```python
from llama_index.core import VectorStoreIndex, StorageContext
from llama_index.vector_stores.opensearch import OpensearchVectorStore

# Step 1에서 생성한 노드(nodes)가 준비되어 있다고 가정합니다.
# from nodes import nodes

# OpenSearch 클라이언트 엔드포인트 설정
# 로컬에 Docker로 설치했거나, 클라우드 서비스를 이용할 수 있습니다.
endpoint = "http://localhost:9200"

# 데이터를 저장할 인덱스 이름 설정
index_name = "arxiv_papers"

# 벡터 저장소 객체 생성
vector_store = OpensearchVectorStore(
    endpoint=endpoint, 
    index_name=index_name
)

# 스토리지 컨텍스트에 벡터 저장소 지정
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# 인덱스 생성
# 이 시점에 nodes의 텍스트가 임베딩 모델을 통해 벡터로 변환되고,
# 텍스트와 벡터가 함께 OpenSearch에 저장됩니다.
index = VectorStoreIndex(nodes, storage_context=storage_context)

print(f"'{index_name}' 인덱스에 데이터 저장을 완료했습니다.")
```

> **[팁]** `VectorStoreIndex.from_documents()`를 사용하면 문서 로딩, 분할, 임베딩, 저장을 한 번에 처리할 수도 있습니다. 하지만 각 단계를 명시적으로 나누는 것이 파이프라인을 이해하고 디버깅하는 데 더 유리합니다.

이제 데이터가 검색 가능한 상태로 저장되었습니다. 다음 단계인 **[03_retrieval.md](03_retrieval.md)** 에서는 저장된 데이터를 활용하여 사용자의 질문과 가장 관련성 높은 문서를 찾는 방법을 알아봅니다.
