LangChain 에는 다수의 모듈이 있으므로, 해당 모듈을 모두 설치해야한다.

우선 최소로 설치하려면 다음과 같이 한다.
```bash
pip install langchain
```

`langchain-core`는 LangChain 생태계를 이용하기위한 기본 추상화와 LCEL(LangChain Expression Language)를 지원한다. 기본적으로 `langchain` 과 함께 설치되지만, 다음과 같이 단독으로 설치할 수도 있다.

```bash
pip install langchain-core
```

`langchain-community`는 서드파티 통합관련 패키지를 담고 있다. `langchain`에 포함되어있지만, 다음과 같이 단독으로 설치할 수도 있다.
```bash
pip install langchain-community
```

`langchain-exxperimental` 패키지는 실험적인 모듈을 담고 있으며, 아래와 같이 설치할 수 있다.

```bash 
pip install langchain-experimental
```

`langgraph`는 LLM으로 상태보관형 다중 액터 시스템을 만들 때 사용한다.

```bash
pip install langgraph
```

`langserve`는 개발자가 LangChain 실행물을 REST API로 구성할 수 있도록 해준다. `langchain` 설치시에 자동으로 설치되지만, 단독으로 설치할 수 있다.

```bash
pip install "langserve[all]"
```

LangChain CLI는 LangChain 템플릿으로 작업하거나 LangServe 프로젝트와 작업할 때 쓰인다.

```bash
pip install langchain-cli
```

LangSmith SDK는 LangChain 설치시 자동으로 설치된다.

```bash
pip install langsmith
```