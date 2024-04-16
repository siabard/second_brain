Text Splitter는 토큰 제한이 있는 LLM이 여러 문장을 참고해 답변할 수 있도록 문서를 분할하는 역할을 한다.

![[langchain_textsplitter.png]]

Document를 Embedding 하는 과정에서 Chunking을 하는 역할을 한다. 이렇게 Chunk를 Vector Store에 매칭해야하는데, Vector Store에 질문 Embedding과 유사도가 높은 청크를 찾아 이를 LLM에 보내야하기 때문이다.
Chunk가 너무 크면 LLM의 토큰 제한을 넘어갈 수 있기 때문에, Chunk를 잘 나누어야한다. 이를 담당하는 것이 Text Splitter이다. 

# 설치하기

`pip` 를 통해 다음과 같이 설치한다.

```
pip install -qU langchain-text-splitters
```

# 동작원리

![[langchain_embedding.png]]
문서를 읽고 이를 일정 크기로 나눈 Chunk를 임베딩 벡터로 분류한다. 이 후 입력된 사용자 질문역시 임베딩 벡터로 변환한 후, 이를 기존 저장한 임베딩 벡터와 유사도 검사를 해서 높은 항목의 Chunk를 최종 프로그램트에 넣어 답을 도출한다.

사용자 질문과 Chunk 의 크기만큼 LLM 토큰 제한이 걸리기때문에 Chunk 크기를 지정하는 것은 상당히 중요하다.

# RecursiveCharacterTextSplitter
`CharacterTextSplitter`는 구분자 1개 기준으로 분할한다. 그렇기때문에 구분자를 무엇으로 설정했는지에 따라 `max_token`으로 할당된 제한을 지키지 못할 수 있다.
반면에 `RecursiveCharacterTextSplitter`는 구분자 여러 개를 재귀적으로 사용하면서 분할하기 때문에 `max_token` 값을 잘 지킬 수 있다. 보통 줄바꿈, 마침표, 쉼표 순으로 재귀적으로 분할한다.

그러므로 대부분의 경우에는 `RecursiveCharacterTextSplitter`를 사용해서 구분한다.

## Chunk Overlap
각 Chunk 마다 일정정도 서로 겹치게끔 오버랩을 할 수 있다. 왜냐하면 Chunk 의 맥락을 해당 Chunk 내용만 보고는 알 수 없을 수 있기때문에, 이전 Chunk 의 일정 정도 내용을 가져와 맥락을 가져올 수 있도록 한다.

# 실제 코드

## CharacterTextSplitter

```python
with open('./content/drive/My Drive/state_of_the_union.txt') as f:
	state_of_the_union = f.read()

from langchain.text_splitter import CharacterTextSplitter
text_splitter = CharacterTextSplitter(
	separator = "\n\n",
	chunk_size = 1000,    # 한 청크의 크기
	chunk_overlap = 100,  # 청크간에 텍스츠가 겹쳐질 크기
	length_function = len,
)

splitted_texts = text_splitter.split_text(state_of_the_union)

# print(text[0])

char_list = []
for i in range(len(splitted_texts)):
	char_list.append(len(splitted_texts[i]))
# print(char_list)

texts = text_splitter.create_document([state_of_the_union])
```

일반 텍스트가 아닌 PDF 파일은 `PyPDFLoader`를 통해 읽어들인다.

```python

from langchain.document_loader import PyPDFLoader

loader = PyPDFLoader('./content/drive/My Drive/혁신.pdf')
pages = loader.load_and_split()
# print(pages[1].page_content)

from langchain.text_splitter import CharacterTextSplitter
text_splitter = CharacterTextSplitter(
	separator = "\n",
	chunk_size = 1000,
	chunk_overlap = 100,
	lengh_function = len,
)

texts = text_splitter.split_documents(pages)
# print(texts[1].page_content)

```

## RecursiveCharacterTextSplitter

위의 일반 텍스트 예제는 아래와 같다.
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
	chunk_size = 1000,
	chunk_overlap = 100,
	length_function = len,
)

texts = text_splitter.create_documents([state_of_the_union])
# print(texts[0].content)

```

PDF예제는 다음과 같다.
```python
from langchain.document_loader import PyPDFLoader

loader = PyPDFLoader('./content/drive/My Drive/혁신.pdf')
pages = loader.load_and_split()
# print(pages[1].page_content)

from langchain.text_splitter import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
	chunk_size = 1000,
	chunk_overlap = 100,
	lengh_function = len,
)

texts = text_splitter.split_documents(pages)
# print(texts[1].page_content)
```

## 그외 Splitter
코드나 LateX 같은 문서는 특별한 Splitter가 필요하다.

```python
from langchain.text_splitter import (
	RecursiveCharacterTextSplitter,
	Language,
)

RecursiveCharacterTextSplitter.get_separators_for_language(Language.PYTHON)

PYTHON_CODE = """
def hello_world():
  print("Hello World")

# Call the function
hello_world()
"""

python_splitter = RecursiveCharacterTextSplitter.from_language(
	language = Language.PYTHON,
	chunk_size = 50,
	chunk_overlap = 0
)
python_docs = python_splitter.create_document([PYTHON_CODE])

```

## 토큰 단위 텍스트 분할

텍스트 분할의 목적은 LLM이 소화할 수 있을 정도의 텍스트만 호출하도록 만드는 것이다. 따라서 LLM 이 소화할 수 있는 양으로 청크를 제한해야한다. 토큰은 Transformer에서 처리하는 방식에 따라 그 수가 달라질 수 있다. 그러므로, 앱을 얹힐 LLM의 토큰 제한을 파악하고, 해당 LLM이 사용하는 Embedder를 기반으로 토큰 수를 계산해야한다. 
아래 예는 OpenAI의 tiktoken 을 기반으로 개발하는 예이다.
tiktoken은 다음과 같이 설치한다.

```bash
pip install tiktoken
```

```python
import tiktoken
tokenizer = tiktoken.get_encoding("cl100k_base") # GPT 모델의 임베딩 모델

def tiktoken_len(text):
  tokens = tokenizer.encode(text)
  return len(tokens)

# print(tiktoken_len(texts[1].page_content))

text_splitter = RecursiveCharacterTextSplitter(
	chunk_size = 1000, chunk_overlapping = 0, length_function = tiktoken_len
)
texts = text_splitter.split_documens(pages)
```

