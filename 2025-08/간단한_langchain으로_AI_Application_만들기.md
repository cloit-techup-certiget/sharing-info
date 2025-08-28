---

# 간단한 AI application을 만들기 위한 개념들

참고: 해당 문서는 `랭체인과 랭그래프를 활용한 RAG-AI 에이전트 실전 입문`를 많이 참고 했습니다.

---

## AI application을 구성하는 요소들

*(참조: 아래의 요소들은 LangChain에서의 요소입니다.)*

### Prompt

* **정의**: LLM에 입력으로 주어지는 텍스트 템플릿
* LLM에 지시사항, 역할 정의, 예시 등을 명시하여 고품질 답변을 유도할 수 있음

### Message

* **정의**: 입력 단위
* 역할(SystemMessage, HumanMessage, AIMessage)을 포함
* 메시지를 추가/삭제하는 것으로 history를 관리할 수 있음
* Prompt와의 차이: Prompt는 텍스트 기반의 구조화된 입력이나, Message는 역할을 포함

### Output Parser

* **정의**: 모델이 반환한 텍스트 응답을 구조화된 데이터로 변환
* LLM 출력을 애플리케이션에서 쓸 수 있는 형식으로 바꿀 때 필요

#### 기본 파서의 종류

* `StrOutputParser`: 텍스트를 그대로 반환
* `CommaSeparatedListOutputParser`: 문자열을 리스트로 변환
* `PydanticOutputParser`: Pydantic 모델 스키마 기반 파싱

### LLM

* **정의**: LLM의 추상화 계층을 제공하는 LangChain의 기본 클래스
* LLM 공급자(OpenAI, Anthropic 등)와 직접 통신하는 역할을 함

---

## 예제 코드

```python
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field


# Output Parser
class BookRecommend(BaseModel):
    title: str = Field(description="책 제목")
    reason: str = Field(description="책을 선택한 이유")


parser = PydanticOutputParser(pydantic_object=BookRecommend)

# 메시지
messages = [
    (
        "system",
        "당신은 사용자의 요청에 맞는 책을 추천하는 assistant입니다.{format_instruction}",
    ),
    (
        "human",
        "context :'''{context}''' 위의 context의 내용을 이용하여 {question}에 대한 지식을 얻기 적합한 책을 추천해주세요.",
    ),
    ("system", "출력은 반드시 JSON 하나로만. 다음 스키마를 따를 것:\n"),
]

# 프롬프트
prompt = ChatPromptTemplate.from_messages(messages).partial(
    format_instruction=parser.get_format_instructions()
)

# LLM
model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# LCEL
chain = prompt | model | parser

result = chain.invoke(
    {
        "context": """
이하는 현재 판매중인 책 이름입니다.
1.처음부터 제대로 배우는 도커/쿠버네티스 컨테이너 개발과 운영
2.도커로 구축한 랩에서 혼자 실습하며 배우는 네트워크 프로토콜 입문
3.시작하세요! 도커/쿠버네티스
4.그림과 실습으로 배우는 도커 & 쿠버네티스
5.개발자를 위한 쉬운 도커
6.컨테이너 인프라 환경 구축을 위한 쿠버네티스/도커
""",
        "question": "네트워크 지식",
    }
)
print(result)

# 출력 예시
# title='도커로 구축한 랩에서 혼자 실습하며 배우는 네트워크 프로토콜 입문'
# reason='이 책은 네트워크 프로토콜에 대한 기초 지식을 실습을 통해 배우는 데 중점을 두고 있어, 네트워크 지식 습득에 적합합니다.'
```

---

## AI application 심화

### RAG

* 위의 예제 코드에서 `context`에 DB에서 적합한 내용을 찾아 채워 넣으면 더 좋은 응답을 얻을 수 있음.
* 예를 들어, 사용자가 "Python에 관련된 책"을 검색했을 때, `context`로 "python 주제의 책 목록"을 넣어주면 LLM에 그냥 검색했을 때보다 더 나은 답변이 나올 것임.

### Langsmith

* LLM 애플리케이션 개발 운영 플랫폼

### LangGraph

* 그래프, 노드 개념을 사용하여 좀 더 복잡한 AI application 구현에 사용

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field
import operator
from typing import Annotated, TypedDict
from langchain.schema import BaseMessage
from langchain_core.messages import HumanMessage, AIMessage


# AI Application이 사용하는 상태 정의
class State(TypedDict):
    # messages: Annotated[list[BaseMessage], operator.add]  # 대화 히스토리 누적
    messages: list[BaseMessage]
    source_lang: str
    title: str
    reason: str


# Output Parser
class BookRecommend(BaseModel):
    title: str = Field(description="책 제목")
    reason: str = Field(description="책을 선택한 이유")


llm = ChatOpenAI(model="gpt-4o-mini")
parser = PydanticOutputParser(pydantic_object=BookRecommend)

# 간단하게 text의 언어를 확인하는 함수, LLM이 판단하게 대체할 수 있음
def detect_language(state: State):
    text = state["messages"][-1].content
    if any("가" <= ch <= "힣" for ch in text):
        state["source_lang"] = "Korean"
    else:
        state["source_lang"] = "English"
    return state

# 책을 text에 맞는 언어로 추천
def recommend_book(state: State):
    text = state["messages"][-1].content

    prompt = ChatPromptTemplate.from_messages(
        [
            ("system", "당신은 책을 추천하는 사람입니다. "),
            (
                "human",
                "책의 주제:{topic} \n\n 원하는 책의 언어: {source_lang} "
                "{format_instructions}",
            ),
        ]
    )

    chain = prompt | llm | parser
    result = chain.invoke(
        {
            "source_lang": state["source_lang"],
            "topic": text,
            "format_instructions": parser.get_format_instructions(),
        }
    )

    state["title"] = result.title
    state["reason"] = result.reason
    return state


from langgraph.graph import StateGraph, END

# 그래프 정의
graph = StateGraph(State)

# 노드 추가
graph.add_node("detect_language", detect_language)
graph.add_node("recommend_book", recommend_book)

# 시작 노드 → 언어 감지
graph.set_entry_point("detect_language")

# 감지 후 무조건 번역으로
graph.add_edge("detect_language", "recommend_book")

# 종료 지점
graph.add_edge("recommend_book", END)

# 그래프 컴파일
app = graph.compile()

state = {
    "messages": [HumanMessage(content="데이터 엔지니어링")],
    "title": "",
    "reason": "",
}

result = app.invoke(state)
print(result)

# 출력 예시
# {'messages': [HumanMessage(content='데이터 엔지니어링', ...)],
#  'source_lang': 'Korean',
#  'title': '데이터 엔지니어링: 설계와 구축',
#  'reason': '이 책은 데이터 엔지니어링의 기본 개념부터 최신 기술까지 폭넓게 다루고 있어,
#             데이터 파이프라인 구축에 필요한 이론과 실전 사례를 제공합니다. 또한,
#             데이터 모델링, ETL 프로세스, 클라우드 데이터 웨어하우스 등의 주제를 깊이 있게 설명하여
#             데이터 엔지니어링을 배우고자 하는 분들에게 유용합니다.'}
# 영어로 입력한 경우의 예시
# {'messages': [HumanMessage(content='Data Engineering', additional_kwargs={}, response_metadata={})], 'source_lang': 'English', 'title': 'Designing Data-Intensive Applications', 'reason': 'This book provides a comprehensive overview of the principles and practices of data engineering, focusing on data storage, processing, and transmission. It covers essential concepts in distributed systems, database design, and data integration, making it a valuable resource for both beginners and experienced data engineers.'}

```

