로컬에 모델을 구성해서 서비스를 구축해보려는 노력이 늘어나고 있다. 민감한 데이터의 외부 유출을 막음은 물론, 전체 시스템의 통제권을 가지고 올 수 있기 때문이다.
그렇지만, 로컬에 LLM 모델을 실행시키기위해서는 막대한 자원이 필요하다. 보통 데스크탑에서 돌릴 수 있는 모델 크기의 상한은 대략 10B (10 Billion)이 한계일 것이다
문제는 작은 모델의 경우에는 성능이 그다지 좋지 않다는 점이다. 이런 이유로 GPT4나 Claude 같은 유명 LLM 정도의 성능은 기대할 수 없다. 
다만 적절한 크기의 LLM 모델 중 잘 구성된 것을 찾아낸다면 로컬에서도 어느 정도 타협할만한 성능을 낼 수 있기에 Huggingface 에 올라온 EEVE 모델을 이용하여 RAG 를 결합해보도록 하겠다.

# Huggingface/GGUF 모델을 Ollama에 올리기
GGUF 파일은 Huggingface 에서 내려받을 수 있는 모델 파일이다. 우선 Huggingface Hub를 설치한다.

```bash
pip install huggingface-hub
```

다음에 필요한 모델의 GGUF 파일을 다운 받는다.

```bash
huggingface-hub download \
  heegyu/EEVE-Korean-Instruct-10.8B-v1.0-GGUF \
  ggml-model-Q5_K_M.gguf \
  --local-dir ~/ollama/gguf \
  --local-dir-use-symlinks False
```

## Modelfile 생성
Ollama 에 GGUF 파일을 사용하려면 Modelfile 을 구성해줘야한다.

```
FROM ggml-model-Q5_K_M.gguf

TEMPLATE """{{- if .System }}
<s>{{ .System }}</s>
{{- end }}
<s>Human:
{{ .Prompt }}</s>
<s>Assistant:
"""

SYSTEM """A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. """

TEMPERATURE 0
PARAMETER stop <s>
PARAMETER stop </s>
```

템플릿이 가이드되지않는 경우에는 해당 모델의 Base를 찾아가서 TEMPLATE을 찾아 넣어준다. Huggingface 링크에 보면 파이썬 스크립트에 아래와 같은 줄이 있다.

```python
prompt_template = "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions.\nHuman: {prompt}\nAssistant:\n"
text = '한국의 수도는 어디인가요? 아래 선택지 중 골라주세요.\n\n(A) 경성\n(B) 부산\n(C) 평양\n(D) 서울\n(E) 전주'
```
```
```
이 값을 토대로 유추해볼 때, `prompt_template`의 첫번째 줄이 system 메세지이며, 그 이후로 정해진 형태를 쓴 것을 알 수 있다.

이제 Ollama 에서 읽을 수 있는 모델로 변환한다.

```bash
ollama create EEVE-Korean-10.8B -f EEVE-Korean-Instruct-10.8B-v1.0-GGUF/Modelfile
```
위의 명령을 넣으면 신규로 모델이 생성되어 등록된다.
이제 ollama에서 해당파일을 실행한다.

```bash
ollama run EEVE-Korean-10.8B:latest
```

## LangServe


## GPU 사용량 확인
Apple Silicon 에서는 asitop 을 설치해서 확인이 가능하다.

```bash
pip install asitop
```

이제 실행시킬 때 관리자 권한으로 실행시킨다.

```bash
sudo asitop
```

## RAG
RAG를 구성하기위한 한글 Embedding은 여러가지가 있지만 별도의 API 키 없이도 가능한 것은 Huggingface Embedding이다.
```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_google_genai import ChatGoogleGenerativeAI

hf = HuggingFaceEmbeddings(model_name='jhgan/ko-sroberta-multitask')

docs = [
    "오늘은 정말 바쁜 하루였다. 아침에 일찍 일어나서 친구와 만나기로 한 약속을 지켰다.",
    "최근 인공지능 기술의 발전으로 많은 산업 분야에서 혁신이 일어나고 있다. 특히 자연어 처리 기술의 진보가 눈부시다.",
    "제주도는 한국에서 가장 인기 있는 관광지 중 하나다. 아름다운 해변과 푸른 자연이 매력적인 곳이다.",
    "한국의 김치는 세계적으로 유명한 발효 음식이다. 매콤하고 시원한 맛이 특징이며 다양한 요리에 활용된다.",
    "정기적인 운동과 균형 잡힌 식단은 건강을 유지하는 데 필수적이다. 매일 조금씩이라도 몸을 움직이는 습관을 들이자.",
    "온라인 학습 플랫폼의 발전으로 어디서나 다양한 지식을 접할 수 있게 되었다. 평생교육의 중요성이 점점 더 커지고 있다.",
    "지속 가능한 생활을 위해서는 일회용품 사용을 줄이고 재활용을 적극적으로 실천해야 한다.",
    "축구는 전 세계적으로 사랑받는 스포츠 중 하나다. 팀워크와 개인 기술이 조화를 이루어야 승리할 수 있다.",
    "한국의 전통 문화 중 하나인 한복은 그 아름다움으로 많은 사람들에게 사랑받고 있다. 특별한 날에 한복을 입는 것은 큰 의미가 있다.",
    "최근 글로벌 경제는 여러 도전에 직면해 있다. 변동성이 큰 시장에서 기업과 개인은 더욱 신중한 결정을 내려야 한다."
]
 
vectorstore = FAISS.from_texts(docs, embedding=hf)
retriever = vectorstore.as_retriever()

template = """Answer the question based only on the following context:
{context}
 
Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

model = ChatGoogleGenerativeAI(model="gemini-pro")

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

chain.invoke("건강을 유지하려면 어떻게 해야하나요?")
```
위와 같이 RAG를 구성할 수 있다.