임베딩작업은 분할된 텍스트를 데이터 수치화하여 벡터 저장소에 보관하도록 하는 것으로 문장 간의 유사성을 비교할 수 있도록 한다.
문장 간의 유사성은 문장의 벡터 공간상에서의 거리를 계산함으로써 알 수 있다. 대용량의 문서를 이미 Embedding Model 훈련을 했기 때문에, 이미 만들어진 모델을 사용하면 학습없이 Embedding 수행이 가능하다.

# 의존성 추가  (OpenAI 이용)

```bash
pip install openai langchain pypdf tiktoken
```

```python
from langchain.embeddings import OpenAIEmbeddings

embeddings_model = OpenAIEmbeddings(openai_api_key=[.....])
embeddings = embeddings_model.embed_documents(
	[
		"안녕하세요",
		"제 이름은 홍길동입니다.",
		"이름이 무엇인가요?",
		"랭체인은 유용합니다.",
		"Hello World!"
	]
)

# print(len(embeddings), len(embeddings[0]))

# 아래는 임베딩 테스트를 해본 것이다.
embedded_query_q = embeddings_model.embed_query("이 대화에서 언급된 이름은 무엇입니까")
embedded_query_a = embeddings_model.embed_query("이 대화에서 언급한 이름은 홍길동입니다.")
# print(len(embedded_query_q_, len(embedded_query_a))

from numpy import dot
from numpy.linalg import norm
import numpy as np

def cos_sim(A, B):
  return dot(A, B)/(norm(A)*norm(B))

# 이제 문장간의 유사도를 계산해볼 수 있다.
# print(cos_sim(embedded_query_q, embedded_query_a))
# print(cos_sim(embedded_query_q, embeddings[1]))
# print(cos_sim(embedded_query_q, embeddings[3]))
```


## Huggingface embedding

의존성 설치

```bash
pip install sentence_transformers
```

임베딩을 할 때 정규화가 되어야 유사도 검사가 제대로 된다.

```python
from langchain.embeddings import HuggingFaceBgeEmbeddings

model_name = "BAAI/bge-small-en"
model_kwargs = {'device': 'cpu'}
encode_kwargs = {'normalize_embeddings': True}
hf = HuggingFaceBgeEmbeddings(
	model_name = model_name,
	model_kwargs = model_kwargs,
	encode_kwargs = encode_kwargs
)

embeddings = hf.embed_documents([
	"today is monday",
	"weather is nice today",
	"what's the problem?",
	"lanchain is useful",
	"Hello World!",
	"my name is morris"
])
```

실제 사용은 아래와 같다.

```python

BGE_query_q = hf.embed_query("Hello? who is this?")
BGE_query_a = hf.embed_query("Hi, this is Json")

# print(cos_sim(BGE_query_q, BGE_query_a))
```

## 한국어 임베딩 ko-sbert-nli

```python
from langchain.embeddings import HuggingFaceEmbeddings

model_name = "jhgan/ko-sbert-nli"
model_kwargs = {'device': 'cpu'}
encode_kwargs = {'normalize_embeddings': True}
ko = HuggingFaceEmbeddings(
	model_name = model_name,
	model_kwargs = model_kwagrs,
	encode_kwargs = encode_kwargs,
)


sentences = [
	"안녕하세요",
	"제 이름은 홍길동입니다",
	"랭체인은 유용합니다",
	"홍길동 아버지의 이름은 홍상직입니다."
]

ko_embeddings = ko.embed_document(sentenses)
q = "홍길동은 아버지를 아버지라 부르지 못하였습니다. 홍길동 아버지의 이름은 무엇입니까?"
a = "홍길동의 아버지는 엄했습니다."

ko_query_q = ko.embed_query(q)
ko_query_a = ko.embed_query(a)

# print(cos_sim(ko_query_q), cos_sim(ko_query_a))
# print(cos_sim(ko_query_q), cos_sim(ko_embeddings[3]))
```

위의 예제를 돌려보면 `ko_query_a`보다 `ko_embeddings[3]` 쪽이 `ko_query_q`와의 벡터 연관성이 높음을 알 수 있다.