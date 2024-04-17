
벡터 저장소는 임베딩 모델을 통해 수치화한 값을 모아두는 데이터베이스이다.
다양한 종류가 존재한다.  Pure Vector database 는 저장소안에 벡터값만 들어갈 수 있다. 데이터 베이스 연산이 매우 쉽다.

Vector libraries 의 경우에는 벡터 유사도를 계산하는데 특화되어있다. 그러다보니 Pure Vector Database 가 제공하는 기능에는 미치지 못한다.

보통은 Pure Vector Database 모델을 사용하며, 무료로 사용하는 버전으로는 Chroma 가 있다.

# Chroma
`Chroma`는 대표적인 오픈소스 벡터 저장소이다. 설치는 다음과 같다.

```bash
pip install chromadb tiktoken transformers sentence_transformers
```

기본적으로 VectorStore는 데이터를 일시적으로 저장하며, 텍스트와 임베딩 함수를 지정하여 `from_documents()` 함수에 보내면, 지정된 임베딩 함수를 통해 텍스트를 벡터로 변환하고, 이를 임시 DB로 생성시킨다.
`similarity_search()` 함수에 쿠러리를 지정해주면 이를 바탕으로 가장 벡터 유사도가 높은 벡터를 찾아 자연어 형태로 출력한다.

```python
import tiktoken
from lanchain.text_splitter import RecursiveCharacterTextSplitter

tokenizer = tiktoken.get_encoding("cl100k_base")

def tiktoken_len(text):
  tokens = tokenizer.encode(text)
  return len(tokens)

from langchain.embeddings.sentence_transformer import SentenceTransfomerEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain.document_loaders import (TextLoader, PyPDFLoader)

load = PyPDFLoader("...pdf")
pages = loader.load_and_split()

text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap = 0, length_function = tiktoken_len)
docs = text_splitter.split_documents(pagess)

# embedding
from langchain.embeddings import HuggingFaceEmbeddings

model_name = "jhgan/ko-sbert-nil"
model_kwargs = {'device': 'cpu'}
encode_kwargs = {'normalize_embeddings': True}
hf = HuggingFaceEmbeddings(
	model_name = model_name,
	model_kwargs = model_kwargs,
	encode_kwargs = encode_kwargs,
)

# load into Chroma
db = Chroma.from_documents(docs, hf)

# query
query = "좋은 이름은?"
docs = db.similarity_search(query)

# print(docs[0].page_content)
```

위에서 `db`라는 벡터 스토어에 임시로 보관되고 있다. 이를 디스크에 저장하려면 `persist()` 함수를 통해 벡터 저장소를 저장한다. 이 저장한 스토어는 Chroma 객체 선언시 경로를 넣어서 불러오면 된다.

유사도를 함께 가져오려면 `similarity_search_with_score()`를 사용한다. 이렇게하면 특정 score 이상의 답변만 가져올 수도 있다.

```python
docs = db.similarity_search_with_score(query)
```
## 디스크에 저장

```python
db2 = Chroma.from_documents(docs, hf, persist_directory="./chroma.db")
docs = db2.similarity_search(query)

```

## 디스크에서 읽기
```python
db3 = Chroma(persist_directory="./chroma.db", embedding_function=hf)
docs = db3.similarity_search(query)
```

# FAISS
페이지북에서 제공하는 Vector Libraries 이다. 효율적인 검색 및 클러스터링을 위한 라이브러리이며, 모든 크기의 벡터 집하에서 검색하는 알고리즘이 포함되어있다.

```bash
pip install faiss-cpu # CPU 이용
```

```python
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import FAISS
from langchain.document_loaders import PyPDFLoader

loader = PyPDFLoader("...pdf")
pages = loader.load_and_split()

text_splitter = RecursiveCharacterTextSplitter(chunk_size = 500, chunk_overlap = 0, length_function = tiktoken_len)
docs = text_splitter.split_documents(pages)

from langchain.embeddings import HuggingFaceEmbeddings

model_name = "jhgan/ko-sbert-nil"
model_kwargs = {'device': 'cpu'}
encode_kwargs = {'normalize_embeddings': True}
hf = HuggingFaceEmbeddings(
	model_name = model_name,
	model_kwargs = model_kwargs,
	encode_kwargs = encode_kwargs,
)

db = FAISS.from_documents(docs, hf)
query = "인공지능의 산업구조는?"
docs =  db.similarity_search(query)
docs_and_scores = db.similarity_search_and_score(query)

## DB 저장
db.save_local("faiss_index")

## DB 로딩
new_db = FAISS.load_local("faiss_index", hf)

# 질의한 문서에 대하여 가장 유사한 문서를 내놓으면서, 다양성을 염두해두고 가져온다.
docs = new_db.max_marginal_relevance_search(query, k=3)

```