# Step 3: 정보 검색 (Retrieval)

이 단계의 목표는 사용자의 질문(Query)이 들어왔을 때, [Step 2](02_storage.md)에서 구축한 데이터베이스에서 가장 관련성 높은 문서 조각(청크)들을 빠르고 정확하게 찾아내는 것입니다. 다이어그램의 `Hybrid Retrieval Pipeline` 부분에 해당합니다.

## 핵심 프로세스

1.  **쿼리 엔진(Query Engine) 생성**: 저장된 인덱스로부터 검색을 수행할 쿼리 엔진을 만듭니다.
2.  **하이브리드 검색(Hybrid Search) 설정**: 키워드 기반 검색과 의미 기반 벡터 검색을 결합하여 검색 정확도를 높입니다.
3.  **검색 실행 및 결과 확인**: 실제 질문으로 검색을 수행하고 결과를 확인합니다.

## 1. 쿼리 엔진 생성

`VectorStoreIndex` 객체는 `as_query_engine()` 메소드를 통해 간단하게 쿼리 엔진을 생성할 수 있습니다. 이 엔진이 사용자 질문을 받아 검색을 수행하는 주체입니다.

```python
# Step 2에서 생성한 index 객체가 있다고 가정합니다.
# from index import index

query_engine = index.as_query_engine()
```

## 2. 하이브리드 검색 설정

다이어그램의 가장 중요한 특징 중 하나는 **하이브리드 검색**입니다. 이는 두 가지 다른 검색 방식을 결합하여 단점을 보완하고 장점을 극대화하는 전략입니다.

-   **벡터 검색 (Vector Search)**: "AI의 미래"와 같이 추상적이고 의미 중심적인 질문에 강합니다. 질문 자체를 벡터로 변환하여 저장된 문서 벡터들과 코사인 유사도(cosine similarity)를 비교합니다.
-   **키워드 검색 (BM25)**: "LangFuse 논문"과 같이 특정 용어나 고유명사가 포함된 질문에 강합니다. TF-IDF를 개선한 알고리즘으로, 문서 내 키워드의 빈도와 중요도를 계산합니다.

`OpenSearch`는 이 두 가지 검색을 모두 지원합니다. `LlamaIndex`의 `OpensearchVectorStore`를 사용하면 하이브리드 검색 모드를 쉽게 활성화할 수 있습니다.

**예시: 하이브리드 검색을 사용하는 쿼리 엔진 생성**

```python
from llama_index.core import VectorStoreIndex
from llama_index.vector_stores.opensearch import OpensearchVectorStore

# OpenSearch 연결 정보
endpoint = "http://localhost:9200"
index_name = "arxiv_papers"

# hybrid_search=True로 설정
vector_store = OpensearchVectorStore(
    endpoint=endpoint, 
    index_name=index_name, 
    hybrid_search=True
)

# 기존 인덱스를 다시 로드
index = VectorStoreIndex.from_vector_store(vector_store=vector_store)

# 하이브리드 검색을 수행하는 쿼리 엔진
# similarity_top_k: 최종적으로 반환할 문서의 수
query_engine = index.as_query_engine(similarity_top_k=5)
```

> **[팁]** 다이어그램의 `RRF (Reciprocal Rank Fusion)`는 두 검색(BM25, Vector)의 결과 순위를 종합하여 최종 순위를 매기는 효과적인 방법입니다. `LlamaIndex`의 최신 버전에서는 `VectorStoreQueryEngine` 내부적으로 이러한 퓨전 로직을 지원하거나 커스텀하게 구현할 수 있습니다.

## 3. 검색 실행

이제 생성된 쿼리 엔진으로 실제 질문을 던져볼 수 있습니다.

```python
query = "What are the latest advancements in large language models?"
response = query_engine.query(query)

# response 객체에는 답변 텍스트와 소스 노드가 포함됩니다.
print("Response:", response)

# 답변의 근거가 된 소스 문서 확인
for node in response.source_nodes:
    print("Source Node Score:", node.score)
    print(node.get_content())
```

검색된 `source_nodes`가 바로 다음 단계인 **[04_generation.md](04_generation.md)** 에서 LLM에게 컨텍스트로 제공될 핵심 정보입니다.
