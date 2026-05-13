# LangGraph 기반 Multi-Agent 운동 처방 시스템

이 프로젝트는 **LangGraph**의 상태 기반 설계(Stateful Workflow)를 활용하여 사용자의 건강 고민에 대해 단계별 분석 및 맞춤형 운동 솔루션을 제공하는 멀티 에이전트 시스템입니다. 단일 프롬프트 체인의 한계를 넘어, 각 단계별로 정제된 데이터를 상태(State)값으로 관리하며 최종 결과물의 품질을 높이는 데 초점을 맞췄습니다.

## 1. 프로젝트 배경 및 목적

기존의 단순 LLM 질의응답 방식은 복잡한 요구사항에서 정보의 누락이 발생하기 쉽습니다. 본 프로젝트는 이를 해결하기 위해 **LangGraph**를 도입하여 다음과 같은 이점을 확보했습니다.

* **관심사 분리(Separation of Concerns)**: 증상 추출, 운동 추천, 리포트 생성 등 각 작업을 독립적인 에이전트(Node)로 분리하여 관리.
* **상태 유지(State Management)**: `TypedDict` 기반의 `AgentState`를 통해 에이전트 간 데이터 전달 과정을 투명하게 관리.
* **확장성**: 추후 피드백 루프나 조건부 분기(Conditional Edge)를 쉽게 추가할 수 있는 구조 설계.

## 2. 기술 스택

* **Orchestration**: LangGraph (StateGraph, TypedDict)
* **LLM Framework**: LangChain (PromptTemplate, Chain)
* **Model**: EXAONE 3.5 2.4B (Running via Ollama)
* **Runtime**: Python 3.10+

## 3. 멀티 에이전트 아키텍처

본 시스템은 세 개의 독립적인 에이전트가 상태를 공유하며 협업합니다.

| 에이전트명 | 역할 설명 |
| --- | --- |
| **Extractor Agent** | 사용자 질문 내 자연어 텍스트에서 운동 처방의 근거가 될 '증상(Symptoms)'을 선별적으로 추출 |
| **Candidate Agent** | 추출된 증상을 분석하여 의학적/생체역학적 관점에서 도움이 되는 운동 3종을 선정 |
| **Answer Agent** | 이전 에이전트들이 생성한 데이터를 취합하여 개조식 리포트 형태로 가공 및 출력 |

## 4. 그래프 구조 (Graph Visualization)

LangGraph로 설계된 에이전트 간의 워크플로우입니다.


<img width="115" height="432" alt="image" src="https://github.com/user-attachments/assets/466128f4-d279-4963-8774-d0af7da271e3" />


## 5. 주요 구현 코드 상세

LangGraph의 핵심인 `StateGraph`와 `Edge` 정의 부분입니다.

```python
# 1. 그래프 초기화
graph = StateGraph(AgentState)

# 2. 노드 배치 (에이전트 연결)
graph.add_node("extractor", extractor_agent)
graph.add_node("candidate", candidate_agent)
graph.add_node("answer", answer_agent)

# 3. 제어 흐름 정의
graph.set_entry_point("extractor")
graph.add_edge("extractor", "candidate")
graph.add_edge("candidate", "answer")
graph.add_edge("answer", END)

# 4. 컴파일
app = graph.compile()

```

## 6. 실행 결과 (Output Example)

실제 로컬 환경(Ollama)에서 EXAONE 3.5 모델을 통해 생성된 결과물입니다.

### 실제 출력 결과

<img width="1168" height="328" alt="image" src="https://github.com/user-attachments/assets/1d4ac59d-d4a1-4cfd-b1f5-e775486342e7" />


## 7. 향후 개선 사항

* **Human-in-the-loop**: 추천된 운동 중 사용자가 선호하지 않는 항목을 제외할 수 있는 인터럽트 기능 추가.
* **Conditional Routing**: 증상의 심각도에 따라 '병원 방문 권고' 노드로 분기하는 로직 구현.
* **Database Integration**: 사용자의 과거 운동 기록을 Vector DB에 저장하여 개인화된 추천 강화.

---
