# Ollama 설치하기

Python 에서 아래와 같은 코드로 Ollama 에 연결한다.

```python
from langchain_community.llms import Ollama
from langchain_community.chat_models import ChatOllama

llm = Ollama(model="llama2")
chat_model = ChatOllama()
```

`llm`과 `chat_model` 은 설정하는 방법과 추가적인 파라미터가 매우 비슷하다. 다른 점은 입/출력 형식이다. LLM에서는 문자열로 입출력이 일어나고, ChatModel 에서는 메시지의 리스트를 입력으로 받아 메시지를 출력한다.

이 두 모델의 차이는 아래 코드에서 볼 수 있다.

```python
from langchain_core.messages import HumanMessage

text = "What would be a good company name for a company that makes colorful socks?"
messages = [HumanMessage(content=text)]

llm.invoke(text)
# >> Feetful of Fun

chat_model.invoke(messages)
# >> AIMessage(content="Socks O'Color")
```

LLM에서는 문자열을 반환하지만, ChatModel 에서는 메시지 객체를 반환하는 것을 볼 수 있다.

# 프롬프트

대부분의 LLM 어플리케이션에서는 사용자 입력을 바로 LLM에 전달하지 않는다. 대신에 사용자의 입력을 프롬프트 템플릿이라고하는 더 큰 텍스트 조각에 포함시키는 방식을 취한다. 

이전 예제에서 회사의 이름을 질의하는 문장에서, 회사의 상품에 대한 부분만 사용자가 지정한다면 매우 편리할 것이다. PromptTemplate 은 이런 상황에서 사용한다. 위 코드를 PromptTemplate를 써서 프롬프트를 만들도록 하면 다음과 같다.

```python
from langchain.prompts import PromptTemplate

prompt = PromptTemplate.from_template("What is a good name for a company that makes {product}?")
prompt.format(product="colorful socks")
```

