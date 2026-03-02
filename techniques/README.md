# AI 핵심 기법 모음

> 실제 서비스에서 사용되는 AI 기법들을 정리한 레퍼런스

## 목차

| 기법 | 카테고리 | 핵심 개념 |
|------|----------|----------|
| [RAG](#rag-retrieval-augmented-generation) | 지식 확장 | 외부 지식 검색 후 생성 |
| [Agentic RAG](#agentic-rag) | 지식 확장 | 에이전트가 자율 검색 전략 결정 ⭐ |
| [GraphRAG](#graphrag) | 지식 확장 | 지식 그래프 기반 관계 검색 ⭐ |
| [RLHF](#rlhf-reinforcement-learning-from-human-feedback) | 학습 방법 | 인간 피드백으로 모델 정렬 |
| [ReAct](#react-reasoning--acting) | 추론 방법 | 사고 + 행동 반복 루프 |
| [Chain-of-Thought (CoT)](#chain-of-thought-cot) | 프롬프팅 | 단계적 추론 유도 |
| [Tree of Thoughts (ToT)](#tree-of-thoughts-tot) | 추론 방법 | 다중 경로 탐색 |
| [Inference-Time Scaling](#inference-time-scaling--test-time-compute) | 추론 방법 | 추론 시 컴퓨팅 증가로 성능 향상 ⭐ |
| [Self-RAG](#self-rag) | 지식 확장 | 검색 필요성 자체 판단 |
| [Constitutional AI](#constitutional-ai-cai) | 안전성 | 원칙 기반 자기 교정 |
| [Function Calling / Tool Use](#function-calling--tool-use) | 도구 연동 | 외부 함수 호출 |
| [Agentic Loop](#agentic-loop) | 에이전트 | 자율 행동 루프 |
| [Multi-Agent](#multi-agent-시스템) | 에이전트 | 여러 에이전트 협력 |
| [Context Engineering](#context-engineering) | 에이전트 | 동적 컨텍스트 조립 ⭐ |
| [MoE (Mixture of Experts)](#mixture-of-experts-moe) | 아키텍처 | 전문가 라우팅으로 효율화 ⭐ |
| [Speculative Decoding](#speculative-decoding) | 최적화 | 초안 모델로 추론 속도 2-3배 향상 ⭐ |
| [Prompt Engineering](#prompt-engineering) | 기초 | 프롬프트 설계 방법론 |
| [Fine-tuning](#fine-tuning) | 학습 방법 | 도메인 특화 모델 학습 |
| [Embeddings & Vector DB](#embeddings--vector-db) | 데이터 | 의미 기반 검색 |

> ⭐ 2025~2026 신규/주요 업데이트 기법

---

## RAG (Retrieval-Augmented Generation)

### 개념

모델의 한계(지식 컷오프, 환각)를 외부 지식 검색으로 보완하는 기법.

```
사용자 질문
    ↓
[Retriever] → 관련 문서 검색 (Vector DB)
    ↓
[Generator] → 검색된 문서 + 질문으로 답변 생성
    ↓
최종 답변
```

### 기본 RAG 구현

```python
from anthropic import Anthropic
import chromadb
from sentence_transformers import SentenceTransformer

client = Anthropic()
embedding_model = SentenceTransformer("all-MiniLM-L6-v2")
chroma_client = chromadb.Client()
collection = chroma_client.create_collection("knowledge_base")

def add_documents(docs: list[str]):
    """문서를 Vector DB에 저장"""
    embeddings = embedding_model.encode(docs).tolist()
    collection.add(
        embeddings=embeddings,
        documents=docs,
        ids=[f"doc_{i}" for i in range(len(docs))]
    )

def rag_query(question: str, n_results: int = 3) -> str:
    # 1. 질문을 벡터로 변환
    query_embedding = embedding_model.encode([question]).tolist()

    # 2. 관련 문서 검색
    results = collection.query(
        query_embeddings=query_embedding,
        n_results=n_results
    )
    context = "\n\n".join(results["documents"][0])

    # 3. 검색된 컨텍스트로 답변 생성
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": f"""다음 컨텍스트를 바탕으로 질문에 답해줘.

컨텍스트:
{context}

질문: {question}

컨텍스트에 없는 내용은 추측하지 말고 "정보가 없습니다"라고 답해줘."""
        }]
    )
    return response.content[0].text
```

### 고급 RAG 패턴

#### Hybrid Search (키워드 + 의미 검색)
```python
from rank_bm25 import BM25Okapi

def hybrid_search(query: str, docs: list[str], alpha: float = 0.5):
    # 키워드 검색 (BM25)
    tokenized_docs = [doc.split() for doc in docs]
    bm25 = BM25Okapi(tokenized_docs)
    keyword_scores = bm25.get_scores(query.split())

    # 의미 검색 (Vector)
    query_vec = embedding_model.encode([query])
    doc_vecs = embedding_model.encode(docs)
    semantic_scores = cosine_similarity(query_vec, doc_vecs)[0]

    # 두 점수 결합
    combined = alpha * keyword_scores + (1 - alpha) * semantic_scores
    return [docs[i] for i in combined.argsort()[::-1][:3]]
```

#### Re-ranking
```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

def rerank(query: str, candidates: list[str]) -> list[str]:
    pairs = [(query, doc) for doc in candidates]
    scores = reranker.predict(pairs)
    ranked = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)
    return [doc for doc, _ in ranked]
```

### 실제 기업 RAG 활용

| 기업 | 활용 방식 | 효과 |
|------|----------|------|
| **Notion AI** | 사용자 노트베이스 RAG | 개인 지식 기반 답변 |
| **GitHub Copilot** | 현재 오픈 파일 + 레포지토리 코드 검색 | 프로젝트 맥락 인식 |
| **Salesforce Einstein** | CRM 데이터 RAG | 고객별 맞춤 응답 |
| **Bloomberg GPT** | 금융 문서 RAG | 정확한 금융 데이터 기반 분석 |
| **법무법인** | 판례 데이터베이스 RAG | 유사 판례 자동 검색 |

---

## Agentic RAG

### 개념

기존 RAG의 한계(단일 검색, 고정 파이프라인)를 에이전트가 극복. 에이전트가 검색 전략을 스스로 결정하고, 중간 결과를 반영해 다음 검색을 조정.

```
[사용자 질문]
      ↓
[에이전트: 검색 계획 수립]
      ↓
[Step 1] 넓은 범위 검색 → 중간 결과 분석
      ↓
[Step 2] 결과가 불충분하면 → 다른 쿼리로 재검색
      ↓
[Step 3] 여러 소스 결과 통합 → 교차 검증
      ↓
[최종 답변 생성]
```

### 기존 RAG vs Agentic RAG

| 구분 | 기존 RAG | Agentic RAG |
|------|---------|-------------|
| 검색 횟수 | 1회 고정 | 동적 (필요에 따라 반복) |
| 쿼리 결정 | 사용자 입력 그대로 | 에이전트가 쿼리 재작성 |
| 전략 | 단일 벡터 검색 | 벡터+키워드+그래프 혼합 |
| 실패 처리 | 없음 | 실패 시 전략 변경 |

### 구현

```python
import anthropic

client = anthropic.Anthropic()

# 검색 도구들
retrieval_tools = [
    {
        "name": "vector_search",
        "description": "의미 기반 벡터 검색으로 관련 문서 찾기",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "n_results": {"type": "integer", "default": 5}
            },
            "required": ["query"]
        }
    },
    {
        "name": "keyword_search",
        "description": "정확한 키워드로 문서 검색",
        "input_schema": {
            "type": "object",
            "properties": {"keywords": {"type": "string"}},
            "required": ["keywords"]
        }
    },
    {
        "name": "refine_query",
        "description": "검색 결과가 불충분할 때 쿼리 재작성",
        "input_schema": {
            "type": "object",
            "properties": {
                "original_query": {"type": "string"},
                "missing_info": {"type": "string"}
            },
            "required": ["original_query", "missing_info"]
        }
    }
]

def agentic_rag(question: str) -> str:
    messages = [{
        "role": "user",
        "content": f"""다음 질문에 답하기 위해 필요한 정보를 검색해줘.
충분한 정보가 모일 때까지 검색 전략을 바꿔가며 반복 검색해줘.

질문: {question}"""
    }]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=retrieval_tools,
            messages=messages,
            system="당신은 정보 검색 전문가입니다. 여러 검색 전략을 활용해 충분한 근거를 수집하세요."
        )

        if response.stop_reason == "end_turn":
            return response.content[0].text

        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = execute_retrieval_tool(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result
                })

        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})
```

### 실제 기업 활용

| 기업 | 활용 방식 | 효과 |
|------|----------|------|
| **Perplexity AI** | 멀티스텝 웹 검색 + 재쿼리 | 복잡한 조사 질문 처리 |
| **Contextual AI** | 엔터프라이즈 문서 에이전트 검색 | 법률/금융 문서 정확도 향상 |
| **Glean** | 회사 내부 지식 에이전트 RAG | 업무 맥락 인식 검색 |

---

## GraphRAG

### 개념

텍스트 대신 **지식 그래프(Knowledge Graph)**로 데이터를 구조화. 엔티티 간 관계를 그래프로 표현해 다중 홉 추론이 가능.

```
일반 RAG:
"스티브 잡스가 설립한 회사?" → 벡터 검색 → Apple

GraphRAG:
"스티브 잡스의 동료가 나중에 설립한 경쟁사?" →
  [잡스] → [동료] → [워즈니악] → [공동창업] → [Apple]
          → [동료] → [스컬리] → [이후] → [관계없음]
  그래프 탐색으로 관계 체인 추적
```

### 구현 (Neo4j + Claude)

```python
from neo4j import GraphDatabase
import anthropic

client = anthropic.Anthropic()
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))

def graph_rag(question: str) -> str:
    # 1. Claude가 Cypher 쿼리 생성
    cypher_query = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{
            "role": "user",
            "content": f"""다음 질문에 답하기 위한 Neo4j Cypher 쿼리를 작성해줘.
그래프 스키마: (Person)-[:WORKS_AT]->(Company)-[:COMPETES_WITH]->(Company)

질문: {question}

Cypher 쿼리만 출력해줘."""
        }]
    ).content[0].text

    # 2. 그래프 DB 쿼리 실행
    with driver.session() as session:
        result = session.run(cypher_query)
        graph_data = [dict(record) for record in result]

    # 3. 그래프 결과로 최종 답변 생성
    return client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": f"""그래프 데이터: {graph_data}

질문: {question}

위 데이터를 바탕으로 답해줘."""
        }]
    ).content[0].text

# Microsoft GraphRAG 오픈소스 활용
# pip install graphrag
# graphrag index --root ./my_data
# graphrag query --root ./my_data --method global "회사 전략은?"
```

### RAG 진화 단계

```
Naive RAG → Advanced RAG → Self-RAG → Agentic RAG → GraphRAG
  단순 검색    하이브리드     자기평가      멀티스텝       관계 추론
```

---

## RLHF (Reinforcement Learning from Human Feedback)

### 개념

인간의 선호도를 학습 신호로 사용해 모델 동작을 정렬하는 학습 방법론.

```
단계 1: Supervised Fine-Tuning (SFT)
  인간이 작성한 고품질 예시로 기본 학습

단계 2: Reward Model 학습
  인간이 두 응답 중 더 나은 것을 선택 → 선호도 데이터 수집
  수집된 선호도로 보상 모델 학습

단계 3: PPO (Proximal Policy Optimization)
  보상 모델을 신호로 삼아 언어 모델 강화학습
  KL divergence로 과도한 변화 방지
```

### RLHF의 변형들

#### RLAIF (RL from AI Feedback)
인간 대신 AI가 피드백 제공 — 확장성 문제 해결

```python
# RLAIF 패턴: AI가 스스로 응답 평가
def rlaif_evaluate(question: str, response: str) -> float:
    evaluation = client.messages.create(
        model="claude-opus-4-6",  # 더 강한 모델이 평가
        max_tokens=100,
        messages=[{
            "role": "user",
            "content": f"""다음 질문에 대한 응답을 평가해줘.

질문: {question}
응답: {response}

다음 기준으로 1-10점 평가 (숫자만 출력):
- 정확성, 유용성, 안전성, 간결성"""
        }]
    )
    return float(evaluation.content[0].text.strip())
```

#### DPO (Direct Preference Optimization)
PPO 없이 직접 선호도 최적화 — 더 안정적, 최근 주류

```
RLHF 과정:
SFT → Reward Model 학습 → PPO 강화학습 (복잡, 불안정)

DPO 과정:
SFT → Direct Preference Optimization (간단, 안정적)
```

#### Constitutional AI (CAI)
원칙 목록으로 자기 비판 및 수정 — Anthropic의 핵심 기법

### 실제 적용 현황

| 기업/모델 | RLHF 변형 | 특징 |
|----------|----------|------|
| **OpenAI GPT-4** | RLHF + PPO | InstructGPT 기반 |
| **Anthropic Claude** | RLHF + Constitutional AI | 안전성 중심 |
| **Meta LLaMA 2** | RLHF + DPO | 오픈소스 |
| **Google Gemini** | RLHF + RLAIF | 멀티모달 |

---

## ReAct (Reasoning + Acting)

### 개념

"생각(Reasoning)"과 "행동(Acting)"을 번갈아 수행하며 복잡한 문제를 해결하는 에이전트 패턴.

```
[Thought] 무엇을 해야 할지 추론
[Action] 도구 실행 (검색, 계산, API 호출 등)
[Observation] 결과 관찰
[Thought] 관찰 결과로 다시 추론
[Action] 다음 행동 결정
... 반복 ...
[Final Answer] 최종 답변
```

### ReAct 구현

```python
import anthropic

client = anthropic.Anthropic()

tools = [
    {
        "name": "web_search",
        "description": "웹에서 최신 정보 검색",
        "input_schema": {
            "type": "object",
            "properties": {"query": {"type": "string"}},
            "required": ["query"]
        }
    },
    {
        "name": "calculator",
        "description": "수학 계산 수행",
        "input_schema": {
            "type": "object",
            "properties": {"expression": {"type": "string"}},
            "required": ["expression"]
        }
    },
    {
        "name": "code_executor",
        "description": "Python 코드 실행",
        "input_schema": {
            "type": "object",
            "properties": {"code": {"type": "string"}},
            "required": ["code"]
        }
    }
]

def react_agent(task: str):
    messages = [{"role": "user", "content": task}]

    print(f"[Task] {task}\n")

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages,
            system="""당신은 ReAct 에이전트입니다.
도구를 활용해 단계적으로 문제를 해결하세요.
각 단계에서 생각(Thought)을 먼저 표현하고 필요한 도구를 호출하세요."""
        )

        if response.stop_reason == "end_turn":
            final_answer = next(b.text for b in response.content if hasattr(b, 'text'))
            print(f"[Final Answer] {final_answer}")
            return final_answer

        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                print(f"[Action] {block.name}({block.input})")
                result = execute_tool(block.name, block.input)
                print(f"[Observation] {result[:200]}...")
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result)
                })

        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})
```

### ReAct vs 다른 패턴

| 패턴 | 특징 | 적합한 상황 |
|------|------|-----------|
| **ReAct** | 사고 + 행동 통합 | 복잡한 멀티스텝 작업 |
| **Chain-of-Thought** | 사고만 (행동 없음) | 추론 문제, 수학 |
| **Act-only** | 행동만 (사고 없음) | 단순 도구 호출 |
| **Plan-and-Execute** | 먼저 전체 계획 수립 | 장기 프로젝트 |

---

## Chain-of-Thought (CoT)

### 개념

중간 추론 단계를 명시적으로 생성하게 해서 정확도를 높이는 프롬프팅 기법.

### Zero-shot CoT

```python
# 단순히 "단계적으로 생각해줘" 추가
prompt = """문제: 15명이 8일 동안 하는 일을 6명이 하려면 며칠이 필요한가?

단계적으로 생각해줘."""

# 효과: 단순 질문보다 정확도 약 40% 향상
```

### Few-shot CoT

```python
prompt = """예시:
Q: 사과 5개에서 3개를 팔고, 7개를 더 샀다. 몇 개인가?
A: 처음 5개 → 3개 팔면 2개 → 7개 더 사면 2+7=9개. 답: 9개

Q: 15명이 8일 할 일을 6명이 하면?
A:"""
```

### 자동 CoT (Auto-CoT)

```python
def auto_cot(question: str) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=[{
            "role": "user",
            "content": f"{question}\n\nLet's think step by step:"
        }],
        thinking={  # Extended Thinking 활성화
            "type": "enabled",
            "budget_tokens": 10000
        }
    )
    return response.content[-1].text
```

### Extended Thinking (Claude 특화)

```python
# Claude의 내부 추론 토큰 활성화
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # 추론에 사용할 최대 토큰
    },
    messages=[{"role": "user", "content": "복잡한 알고리즘 문제..."}]
)

# thinking 블록과 text 블록이 함께 반환됨
for block in response.content:
    if block.type == "thinking":
        print(f"[내부 추론]\n{block.thinking}")
    elif block.type == "text":
        print(f"[최종 답변]\n{block.text}")
```

---

## Inference-Time Scaling / Test-Time Compute

### 개념

**2025년 AI 분야 가장 중요한 패러다임 전환.** 학습 데이터/파라미터 확장 대신, 추론(inference) 시 컴퓨팅을 더 사용해 성능을 높이는 방법.

```
기존 패러다임:
성능 향상 = 더 큰 모델 + 더 많은 데이터 (학습 비용↑↑)

새 패러다임:
성능 향상 = 추론 시 더 많이 생각 (추론 비용↑, 학습 비용 동일)

비유: 시험 전날 벼락치기(학습) vs 시험 중 꼼꼼히 검토(추론)
```

### 핵심 방법들

#### 1. Best-of-N Sampling
```python
def best_of_n(question: str, n: int = 8) -> str:
    """N개 답변 생성 후 가장 좋은 것 선택"""
    candidates = []
    for _ in range(n):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=2048,
            messages=[{"role": "user", "content": question}],
            temperature=0.8   # 다양성을 위해 온도 높임
        )
        candidates.append(response.content[0].text)

    # 보상 모델로 최고 답변 선택
    best = reward_model_select(question, candidates)
    return best
```

#### 2. Self-Consistency (다수결)
```python
def self_consistency(question: str, n: int = 10) -> str:
    """같은 질문을 여러 번 물어보고 가장 많이 나온 답변 선택"""
    answers = []
    for _ in range(n):
        answer = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=512,
            messages=[{"role": "user", "content": f"{question}\n\n최종 답만 출력해줘."}]
        ).content[0].text
        answers.append(answer)

    # 가장 많이 나온 답변 반환
    from collections import Counter
    return Counter(answers).most_common(1)[0][0]
```

#### 3. Extended Thinking (Claude 특화)
```python
# 추론 토큰 예산 설정 — 클수록 더 깊이 생각
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000   # 내부 추론에 사용할 최대 토큰
    },
    messages=[{"role": "user", "content": "복잡한 수학 증명 문제..."}]
)

for block in response.content:
    if block.type == "thinking":
        print(f"[내부 추론 과정]\n{block.thinking}")
    elif block.type == "text":
        print(f"[최종 답변]\n{block.text}")
```

#### 4. Monte Carlo Tree Search (MCTS) + LLM
```python
# 가장 복잡한 형태: 트리 탐색으로 최적 추론 경로 탐색
# AlphaGo의 MCTS를 언어 모델 추론에 적용
def mcts_reasoning(problem: str, simulations: int = 100):
    root = ThoughtNode(problem)
    for _ in range(simulations):
        # 1. Selection: 유망한 노드 선택 (UCB1)
        node = select_node(root)
        # 2. Expansion: 새 추론 경로 생성
        child = expand_with_llm(node)
        # 3. Simulation: 끝까지 추론
        score = simulate_rollout(child)
        # 4. Backpropagation: 점수 역전파
        backpropagate(child, score)
    return best_path(root)
```

### 추론 모델 현황 (2025~)

| 모델 | 방법 | 특징 |
|------|------|------|
| **OpenAI o3** | Chain-of-thought + MCTS | 코딩/수학 SOTA |
| **Claude (Extended Thinking)** | Streaming reasoning tokens | 투명한 사고 과정 |
| **DeepSeek-R1** | 순수 RL로 추론 능력 획득 | o1 대비 70% 저렴 |
| **Gemini 2.0 Flash Thinking** | 추론 토큰 + 빠른 응답 | 속도/성능 균형 |

### 언제 사용하나

```
일반 추론으로 충분:     빠른 요약, 번역, 간단한 코드 작성
Inference-Time Scaling: 수학 증명, 복잡한 알고리즘, 전략 수립, 코드 디버깅
```

---

## Tree of Thoughts (ToT)

### 개념

여러 추론 경로를 동시에 탐색하고 가장 유망한 경로를 선택하는 방법.

```
질문
 ├── 접근법 A ── 단계 A1 ── 평가: 8/10 ✓ (탐색 계속)
 │              └── 단계 A2 ── 최종 답변 A
 ├── 접근법 B ── 단계 B1 ── 평가: 3/10 ✗ (가지치기)
 └── 접근법 C ── 단계 C1 ── 평가: 6/10 △ (병렬 탐색)
                └── 단계 C2 ── 평가: 9/10 ✓ (우선 탐색)
```

### ToT 구현

```python
def tree_of_thoughts(problem: str, branching_factor: int = 3, depth: int = 3):
    def generate_thoughts(state: str, n: int) -> list[str]:
        """현재 상태에서 n개의 다음 생각 생성"""
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{
                "role": "user",
                "content": f"""문제: {problem}
현재 추론 상태: {state}

이 상태에서 가능한 {n}가지 다른 접근법/다음 단계를 제시해줘.
각각을 <thought> 태그로 구분해줘."""
            }]
        )
        # 태그로 분리
        text = response.content[0].text
        thoughts = re.findall(r'<thought>(.*?)</thought>', text, re.DOTALL)
        return thoughts[:n]

    def evaluate_thought(thought: str) -> float:
        """추론 경로의 유망도 평가 (0-10)"""
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=100,
            messages=[{
                "role": "user",
                "content": f"""문제: {problem}
추론 경로: {thought}

이 추론이 문제 해결에 얼마나 유망한지 0-10점으로 평가해줘 (숫자만)."""
            }]
        )
        return float(response.content[0].text.strip())

    # BFS로 탐색
    current_states = [""]
    for _ in range(depth):
        all_thoughts = []
        for state in current_states:
            thoughts = generate_thoughts(state, branching_factor)
            all_thoughts.extend(thoughts)

        # 평가 후 상위 k개만 유지
        scored = [(t, evaluate_thought(t)) for t in all_thoughts]
        scored.sort(key=lambda x: x[1], reverse=True)
        current_states = [t for t, _ in scored[:branching_factor]]

    return current_states[0]  # 최고 경로 반환
```

---

## Self-RAG

### 개념

모델이 "검색이 필요한가?"를 스스로 판단하고, 검색 후 결과의 유용성도 스스로 평가하는 RAG 변형.

```
질문 입력
    ↓
[Retrieve?] 검색 필요 여부 판단 → 불필요하면 바로 답변
    ↓ (필요한 경우)
[Search] 문서 검색
    ↓
[IsREL?] 검색 결과 관련성 판단 → 관련 없으면 재검색
    ↓
[IsSUP?] 답변이 문서에 의해 지지되는가 확인
    ↓
[IsUSE?] 답변이 유용한가 최종 평가
    ↓
최종 답변
```

### Self-RAG 구현

```python
def self_rag(question: str) -> str:
    # 1. 검색 필요성 판단
    need_retrieval = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=10,
        messages=[{
            "role": "user",
            "content": f"""질문: "{question}"
이 질문에 답하기 위해 외부 정보 검색이 필요한가?
yes/no만 답해줘."""
        }]
    ).content[0].text.strip().lower()

    if need_retrieval == "no":
        # 검색 없이 바로 답변
        return direct_answer(question)

    # 2. 검색 수행
    docs = vector_search(question)

    # 3. 관련성 평가
    relevant_docs = []
    for doc in docs:
        is_relevant = client.messages.create(
            model="claude-haiku-4-5",
            max_tokens=10,
            messages=[{
                "role": "user",
                "content": f"질문: {question}\n문서: {doc}\n관련있나? yes/no"
            }]
        ).content[0].text.strip().lower()

        if is_relevant == "yes":
            relevant_docs.append(doc)

    # 4. 최종 답변 생성
    return generate_with_context(question, relevant_docs)
```

---

## Constitutional AI (CAI)

### 개념

Anthropic이 개발한 AI 안전성 기법. 원칙(Constitution) 목록으로 모델이 스스로 응답을 비판하고 수정.

```
원칙 목록 예시:
1. 해롭거나 위험한 정보를 제공하지 말 것
2. 솔직하고 정직할 것
3. 인간의 자율성을 존중할 것
...

과정:
1. 초기 응답 생성
2. 원칙 중 하나를 무작위 선택
3. "이 원칙에 따라 응답을 비판해줘" → 비판 생성
4. "비판을 반영해서 응답을 수정해줘" → 수정된 응답
5. 반복
6. RLHF로 최종 정렬
```

### CAI 구현 패턴

```python
CONSTITUTION = [
    "응답이 사용자에게 해를 끼치지 않는지 확인해줘",
    "응답이 편향되거나 차별적이지 않은지 확인해줘",
    "응답이 사실에 기반하고 정확한지 확인해줘",
    "응답이 프라이버시를 침해하지 않는지 확인해줘"
]

def constitutional_revision(prompt: str, response: str) -> str:
    current_response = response

    for principle in random.sample(CONSTITUTION, k=2):
        # 비판 단계
        critique = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=512,
            messages=[{
                "role": "user",
                "content": f"""원칙: {principle}

응답: {current_response}

위 원칙에 따라 이 응답의 문제점을 찾아줘."""
            }]
        ).content[0].text

        # 수정 단계
        current_response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{
                "role": "user",
                "content": f"""원본 응답: {current_response}

비판: {critique}

비판을 반영해서 응답을 개선해줘."""
            }]
        ).content[0].text

    return current_response
```

---

## Function Calling / Tool Use

### 개념

LLM이 외부 함수/API를 호출할 수 있게 해주는 기능. Claude에서는 "Tool Use"라고 부른다.

### 기본 Tool Use

```python
import anthropic
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "get_weather",
        "description": "특정 도시의 현재 날씨 조회",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "도시 이름 (영어)"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "온도 단위"
                }
            },
            "required": ["city"]
        }
    },
    {
        "name": "send_email",
        "description": "이메일 전송",
        "input_schema": {
            "type": "object",
            "properties": {
                "to": {"type": "string"},
                "subject": {"type": "string"},
                "body": {"type": "string"}
            },
            "required": ["to", "subject", "body"]
        }
    }
]

def process_tool_call(name: str, inputs: dict) -> str:
    if name == "get_weather":
        # 실제 날씨 API 호출
        return f"{inputs['city']}의 날씨: 맑음, 22°C"
    elif name == "send_email":
        # 이메일 전송 로직
        return f"이메일 전송 완료: {inputs['to']}"

def tool_use_agent(user_message: str):
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "end_turn":
            return response.content[0].text

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = process_tool_call(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })

            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
```

### 병렬 도구 호출

Claude는 여러 도구를 동시에 호출할 수 있다.

```python
# Claude가 자동으로 병렬 실행 결정
# 예: "서울, 도쿄, 뉴욕 날씨 모두 알려줘"
# → get_weather(Seoul), get_weather(Tokyo), get_weather(New York) 동시 실행
```

---

## Agentic Loop

### 개념

에이전트가 목표 달성까지 자율적으로 계획-실행-평가-재계획을 반복하는 패턴.

```
목표 설정
    ↓
[Plan] 작업 분해 및 계획
    ↓
[Execute] 도구로 실행
    ↓
[Observe] 결과 관찰
    ↓
[Evaluate] 목표 달성 여부 평가
    ↓ (미달성)
[Replan] 계획 수정 → 다시 Execute
    ↓ (달성)
완료
```

### 실용적 Agentic Loop

```python
def agentic_loop(goal: str, max_iterations: int = 20):
    messages = [{
        "role": "user",
        "content": f"""목표: {goal}

이 목표를 달성하기 위해 필요한 도구들을 사용해서 단계적으로 작업해줘.
목표를 달성하면 "GOAL_ACHIEVED: [결과 요약]"으로 완료를 알려줘."""
    }]

    for i in range(max_iterations):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=all_available_tools,
            messages=messages
        )

        # 목표 달성 체크
        text_content = "".join(
            b.text for b in response.content
            if hasattr(b, 'text')
        )
        if "GOAL_ACHIEVED" in text_content:
            return text_content

        # 도구 실행
        if response.stop_reason == "tool_use":
            tool_results = execute_all_tools(response.content)
            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
        else:
            messages.append({"role": "assistant", "content": response.content})

    return "최대 반복 횟수 도달"
```

---

## Multi-Agent 시스템

### 개념

여러 전문화된 에이전트가 협력하여 복잡한 작업을 처리.

### 오케스트레이터 패턴

```python
class AgentOrchestrator:
    def __init__(self):
        self.agents = {
            "researcher": ResearchAgent(),    # 정보 수집 전문
            "coder": CodingAgent(),           # 코드 작성 전문
            "reviewer": ReviewAgent(),         # 코드 리뷰 전문
            "tester": TestingAgent()          # 테스트 전문
        }

    def execute_task(self, task: str) -> str:
        # 1. 오케스트레이터가 태스크 분석
        plan = self.create_plan(task)

        results = {}
        for step in plan:
            agent_name = step["agent"]
            agent_task = step["task"]
            context = {k: results[k] for k in step.get("needs", [])}

            # 2. 적절한 에이전트에게 위임
            result = self.agents[agent_name].run(agent_task, context)
            results[step["id"]] = result

        return self.synthesize(results)
```

### 실제 Multi-Agent 활용

| 시스템 | 에이전트 구성 | 활용 |
|--------|-------------|------|
| **AutoGPT** | 계획→실행→평가 에이전트 | 자율 작업 수행 |
| **CrewAI** | 역할 기반 팀 에이전트 | 협업 시뮬레이션 |
| **LangGraph** | 상태 기반 그래프 에이전트 | 복잡한 워크플로우 |
| **Claude Code** | 탐색/계획/실행 서브에이전트 | 코딩 작업 |

---

## Context Engineering

### 개념

2025년 하반기 등장한 개념. 단순한 프롬프트 작성을 넘어, **에이전트가 어떤 정보를 컨텍스트 윈도우에 넣을지 동적으로 조립하는 기술**.

```
기존 접근:
[전체 문서] + [질문] → LLM → 답변
  (컨텍스트 낭비, 노이즈 많음)

Context Engineering:
[작업 관련 핵심 정보만 선별]
+ [이전 대화 요약]
+ [현재 상태]
+ [사용 가능한 도구 목록]
→ LLM → 최적 행동
```

### 핵심 구성 요소

```
Context Window =
  Instructions   (무엇을 해야 하는가)
+ Memory         (과거 무슨 일이 있었나)
+ State          (현재 상태는 무엇인가)
+ Tools          (어떤 도구를 쓸 수 있나)
+ Background     (도메인 지식)
+ Conversation   (현재 대화 내용)
```

### 구현 패턴

```python
class ContextEngineer:
    def __init__(self, max_tokens: int = 100_000):
        self.max_tokens = max_tokens
        self.memory_store = MemoryStore()
        self.token_budget = TokenBudget(max_tokens)

    def build_context(self, task: str, conversation: list) -> list:
        """작업에 최적화된 컨텍스트 조립"""

        # 1. 토큰 예산 배분
        budget = {
            "instructions": 2000,
            "memory": 5000,
            "tools": 3000,
            "conversation": 10000,
            "background": 4000
        }

        # 2. 관련 메모리 검색 (전체 아님, 관련된 것만)
        relevant_memories = self.memory_store.search(
            query=task,
            max_tokens=budget["memory"]
        )

        # 3. 대화 압축 (오래된 것은 요약)
        compressed_conv = self.compress_conversation(
            conversation,
            max_tokens=budget["conversation"]
        )

        # 4. 현재 작업에 필요한 도구만 선택
        relevant_tools = self.select_tools(task)

        # 5. 컨텍스트 조립
        system_context = f"""
# 지시사항
{self.get_instructions(task)}

# 관련 기억
{relevant_memories}

# 배경 지식
{self.get_background(task)}
"""
        return [
            {"role": "system", "content": system_context},
            *compressed_conv
        ]

    def compress_conversation(self, conv: list, max_tokens: int) -> list:
        """긴 대화를 요약해서 토큰 절약"""
        if self.count_tokens(conv) <= max_tokens:
            return conv

        # 오래된 메시지 요약
        old_messages = conv[:-10]  # 최근 10개는 유지
        summary = self.summarize(old_messages)

        return [
            {"role": "system", "content": f"[이전 대화 요약]\n{summary}"},
            *conv[-10:]
        ]
```

### Memory 계층 설계

```python
class HierarchicalMemory:
    """
    Working Memory    → 현재 세션 컨텍스트 (빠름, 제한적)
    Episodic Memory   → 과거 작업/대화 요약 (중기)
    Semantic Memory   → 도메인 지식, 사실 (장기)
    Procedural Memory → 해결 패턴, 베스트 프랙티스 (장기)
    """

    def retrieve(self, query: str, memory_types: list = None) -> str:
        results = []
        for mem_type in (memory_types or ["working", "episodic", "semantic"]):
            retrieved = self.stores[mem_type].search(query, k=3)
            results.extend(retrieved)

        # 관련성 순 정렬 후 토큰 예산 내로 자르기
        return self.rerank_and_trim(results, max_tokens=5000)
```

### 실제 기업 활용

| 기업 | 활용 방식 |
|------|----------|
| **Cursor** | 코드베이스 전체 대신 현재 파일 + 관련 파일만 동적 로드 |
| **Claude Code** | CLAUDE.md + 관련 파일 + 메모리 파일 계층적 조립 |
| **Devin** | 작업 히스토리 + 현재 상태 + 도구 목록 동적 구성 |

---

## Mixture of Experts (MoE)

### 개념

하나의 거대 모델 대신, **여러 전문가(Expert) 네트워크**를 만들고 입력에 따라 일부만 활성화. 2025년 기준 프론티어 모델의 60% 이상이 MoE 채택.

```
기존 Dense 모델:
입력 → [모든 파라미터 활성화] → 출력
(비효율: 수학 문제에도 시 작성 파라미터 활성화)

MoE:
입력 → [Router: 어떤 전문가?] → [전문가 2, 7 활성화] → 출력
(효율: 관련 전문가만 활성화, 동일 품질을 훨씬 빠르게)
```

### 아키텍처

```python
import torch
import torch.nn as nn

class MixtureOfExperts(nn.Module):
    def __init__(self, num_experts: int = 8, top_k: int = 2, d_model: int = 512):
        super().__init__()
        self.num_experts = num_experts
        self.top_k = top_k  # 매 토큰마다 활성화할 전문가 수

        # 라우터: 어느 전문가를 쓸지 결정
        self.router = nn.Linear(d_model, num_experts)

        # 전문가 네트워크들 (각자 다른 가중치)
        self.experts = nn.ModuleList([
            FeedForwardNetwork(d_model) for _ in range(num_experts)
        ])

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        batch_size, seq_len, d_model = x.shape

        # 1. 라우터가 각 토큰에 전문가 점수 계산
        router_logits = self.router(x)  # (batch, seq, num_experts)
        router_weights = torch.softmax(router_logits, dim=-1)

        # 2. 상위 K개 전문가만 선택
        top_k_weights, top_k_indices = torch.topk(router_weights, self.top_k, dim=-1)
        top_k_weights = top_k_weights / top_k_weights.sum(dim=-1, keepdim=True)

        # 3. 선택된 전문가만 실행 (나머지는 건너뜀)
        output = torch.zeros_like(x)
        for expert_idx in range(self.num_experts):
            # 이 전문가를 선택한 토큰만 처리
            mask = (top_k_indices == expert_idx).any(dim=-1)
            if mask.any():
                expert_out = self.experts[expert_idx](x[mask])
                weight = top_k_weights[mask][top_k_indices[mask] == expert_idx]
                output[mask] += expert_out * weight.unsqueeze(-1)

        return output
```

### 실제 MoE 모델들

| 모델 | 전문가 수 | 활성 전문가 | 특징 |
|------|---------|-----------|------|
| **DeepSeek-V3** | 256 | 8 | 세분화된 전문가 분할 |
| **Mistral Large 3** | 8 | 2 | 오픈소스 고성능 |
| **Grok-2** | 비공개 | 비공개 | xAI 개발 |
| **GPT-4** (추정) | 비공개 | 비공개 | OpenAI |

### MoE 장단점

```
장점:
✓ 동일 품질 대비 추론 속도 3-10배 빠름
✓ 파라미터는 많지만 활성 파라미터는 적음 (메모리 효율)
✓ 전문화로 다양한 도메인 처리 유리

단점:
✗ 학습이 불안정 (전문가 붕괴 현상)
✗ 분산 추론 시 통신 오버헤드
✗ 로드 밸런싱 필요 (특정 전문가 과부하)
```

---

## Speculative Decoding

### 개념

LLM의 병목인 "한 번에 한 토큰 생성"을 극복. **작은 초안 모델(Draft Model)**이 여러 토큰을 빠르게 예측하면, **큰 검증 모델(Target Model)**이 한 번에 검증.

```
일반 디코딩:
LLM: "The" → "cat" → "sat" → "on" → "the" → "mat"
       6번 forward pass

Speculative Decoding:
Draft: "The cat sat on the mat" (한 번에 6토큰 제안, 빠름)
Target: "The cat sat on the mat" ✓✓✓✓✓✓ (검증, 1번 병렬 처리)
결과: 2-3배 빠름 (품질 동일 보장)
```

### 구현

```python
def speculative_decode(
    target_model,
    draft_model,
    prompt: str,
    max_new_tokens: int = 100,
    speculation_len: int = 4  # 한 번에 초안 토큰 수
) -> str:
    input_ids = tokenizer.encode(prompt, return_tensors="pt")
    generated = input_ids.clone()

    while generated.shape[1] - input_ids.shape[1] < max_new_tokens:
        # 1. Draft 모델이 K개 토큰 빠르게 생성
        draft_tokens = []
        draft_probs = []
        draft_input = generated.clone()

        for _ in range(speculation_len):
            with torch.no_grad():
                draft_out = draft_model(draft_input)
                next_token_probs = torch.softmax(draft_out.logits[:, -1, :], dim=-1)
                next_token = torch.multinomial(next_token_probs, 1)
                draft_tokens.append(next_token)
                draft_probs.append(next_token_probs[0, next_token.item()])
                draft_input = torch.cat([draft_input, next_token], dim=1)

        # 2. Target 모델이 초안 K개 토큰을 한 번에 검증 (병렬)
        candidate = torch.cat([generated] + draft_tokens, dim=1)
        with torch.no_grad():
            target_out = target_model(candidate)
            target_probs = torch.softmax(target_out.logits[:, -1:, :], dim=-1)

        # 3. 수락/거부 결정 (rejection sampling)
        accepted = 0
        for i, (draft_token, draft_prob) in enumerate(zip(draft_tokens, draft_probs)):
            target_prob = target_probs[0, generated.shape[1] + i - 1, draft_token.item()]
            # 확률 비율로 수락 여부 결정
            acceptance_prob = min(1.0, target_prob / draft_prob)
            if torch.rand(1).item() < acceptance_prob:
                generated = torch.cat([generated, draft_token], dim=1)
                accepted += 1
            else:
                break  # 거부된 토큰부터 target 모델로 재생성

        if accepted == 0:
            # 모두 거부 → target 모델로 1토큰 생성
            next_token = torch.multinomial(target_probs[0, generated.shape[1] - 1], 1)
            generated = torch.cat([generated, next_token.unsqueeze(0)], dim=1)

    return tokenizer.decode(generated[0], skip_special_tokens=True)
```

### 성능 비교

| 방법 | 속도 | 품질 | 비용 |
|------|------|------|------|
| 일반 디코딩 | 1x | 기준 | 기준 |
| Speculative Decoding | 2-3x | 동일 | +10% (draft 모델) |
| 배치 Speculative | 3-5x | 동일 | +15% |

### 실제 적용

```
Google: Gemini 서빙에 내부 speculative decoding 적용 → 2배 빠른 응답
NVIDIA: TensorRT-LLM에 내장 → A100에서 3.7x 속도 향상
Together AI: 오픈소스 추론 서버에 적용
```

---

## Prompt Engineering

### 핵심 원칙

#### 1. 명확한 지시
```python
# 나쁜 예
prompt = "이 코드 고쳐줘"

# 좋은 예
prompt = """다음 Python 함수를 수정해줘:
1. 타입 힌트 추가
2. docstring 추가
3. 입력 검증 추가 (None 체크)

수정할 코드:
```python
def calculate_average(numbers):
    return sum(numbers) / len(numbers)
```"""
```

#### 2. 역할 부여 (Role Prompting)
```python
system = """당신은 10년 경력의 시니어 보안 엔지니어입니다.
코드 리뷰 시 OWASP Top 10을 기준으로 취약점을 분석합니다."""
```

#### 3. 출력 형식 지정
```python
prompt = """분석 결과를 다음 JSON 형식으로 출력해줘:
{
  "vulnerabilities": [
    {"line": 10, "type": "SQL Injection", "severity": "HIGH", "fix": "..."}
  ],
  "summary": "...",
  "risk_score": 8.5
}"""
```

#### 4. Few-shot 예시
```python
prompt = """번역해줘 (한국어 IT 용어 스타일):

Input: "The deployment pipeline failed"
Output: "배포 파이프라인이 실패했습니다"

Input: "Please review my pull request"
Output: "제 PR을 리뷰해 주세요"

Input: "The API endpoint is returning 500 errors"
Output: """
```

### 고급 프롬프팅 기법

| 기법 | 설명 | 효과 |
|------|------|------|
| **CoT** | "단계적으로 생각해줘" | 추론 정확도 향상 |
| **Self-Consistency** | 동일 질문 여러 번 → 다수결 | 안정성 향상 |
| **Least-to-Most** | 쉬운 것부터 어려운 것 순서로 | 복잡한 문제 해결 |
| **Emotional Prompting** | "이건 매우 중요해" 추가 | 집중도 향상 (논란 있음) |
| **Metacognitive** | "확신도를 %로 표현해줘" | 불확실성 인식 |

---

## Fine-tuning

### 언제 사용하나

```
Fine-tuning이 필요한 경우:
✓ 특정 도메인 어휘/스타일 학습 (의료, 법률, 금융)
✓ 일관된 출력 형식 강제
✓ 반복적인 few-shot 예시 대체 (비용 절감)
✓ 프롬프트로 달성 불가능한 특수 동작

Fine-tuning이 불필요한 경우:
✗ RAG로 해결 가능한 지식 업데이트
✗ 프롬프트 엔지니어링으로 해결 가능
✗ 데이터가 부족한 경우 (< 100개)
```

### Claude Fine-tuning (Anthropic API)

```python
import anthropic

client = anthropic.Anthropic()

# 파인튜닝 데이터 형식
training_data = [
    {
        "messages": [
            {"role": "user", "content": "서울 날씨 어때?"},
            {"role": "assistant", "content": "현재 서울의 날씨 정보를 확인하겠습니다..."}
        ]
    }
]

# Fine-tuning 작업 생성 (API 통해)
# 현재 Claude는 제한적 파인튜닝 지원
# 주로 프롬프트 캐싱과 RAG로 대체
```

### LoRA (Low-Rank Adaptation)

오픈소스 모델 파인튜닝의 주류 방법.

```python
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b")

lora_config = LoraConfig(
    r=16,           # 랭크 (낮을수록 파라미터 적음)
    lora_alpha=32,  # 스케일링 팩터
    target_modules=["q_proj", "v_proj"],  # 적용할 레이어
    lora_dropout=0.1,
    bias="none"
)

model = get_peft_model(model, lora_config)
# 전체 파라미터의 0.1-1%만 학습 → 빠르고 저렴
```

---

## Embeddings & Vector DB

### 개념

텍스트를 의미론적 벡터로 변환하여 의미 기반 검색을 가능하게 하는 기술.

```
"강아지가 뛰어논다" → [0.23, -0.15, 0.87, ...]  (1536차원)
"개가 달린다"      → [0.21, -0.13, 0.89, ...]  (의미 유사 → 벡터 유사)
"주식 시장 하락"   → [-0.45, 0.67, -0.12, ...] (의미 다름 → 벡터 다름)
```

### Anthropic Embeddings API

```python
client = anthropic.Anthropic()

def get_embedding(text: str) -> list[float]:
    response = client.embeddings.create(
        model="voyager-3",  # Anthropic 임베딩 모델
        input=text
    )
    return response.embedding

def cosine_similarity(v1: list[float], v2: list[float]) -> float:
    import numpy as np
    v1, v2 = np.array(v1), np.array(v2)
    return np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))
```

### Vector DB 비교

| DB | 특징 | 적합한 규모 |
|----|------|-----------|
| **Chroma** | 로컬, 개발용, Python 네이티브 | 소규모 |
| **Pinecone** | 완전 관리형, 빠름 | 중~대규모 |
| **Weaviate** | 멀티모달, 그래프 | 중규모 |
| **Qdrant** | Rust, 빠름, 오픈소스 | 중~대규모 |
| **pgvector** | PostgreSQL 확장, 기존 DB 통합 | 소~중규모 |
| **Milvus** | 분산, 초대규모 | 대규모 |

### 실용 벡터 검색 파이프라인

```python
import chromadb
from anthropic import Anthropic

client = Anthropic()
chroma = chromadb.PersistentClient(path="./vector_db")
collection = chroma.get_or_create_collection(
    "company_docs",
    metadata={"hnsw:space": "cosine"}
)

class KnowledgeBase:
    def add(self, docs: list[dict]):
        """문서 추가 (id, content, metadata 포함)"""
        embeddings = [get_embedding(d["content"]) for d in docs]
        collection.add(
            ids=[d["id"] for d in docs],
            embeddings=embeddings,
            documents=[d["content"] for d in docs],
            metadatas=[d.get("metadata", {}) for d in docs]
        )

    def search(self, query: str, n: int = 5, filter: dict = None) -> list[str]:
        results = collection.query(
            query_embeddings=[get_embedding(query)],
            n_results=n,
            where=filter  # 메타데이터 필터링
        )
        return results["documents"][0]

    def answer(self, question: str) -> str:
        context = self.search(question)
        return client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{
                "role": "user",
                "content": f"컨텍스트:\n{chr(10).join(context)}\n\n질문: {question}"
            }]
        ).content[0].text
```

---

*마지막 업데이트: 2026-03-02*
