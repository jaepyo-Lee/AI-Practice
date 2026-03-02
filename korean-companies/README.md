# 국내 기업 AI 자동화 사례

> 네이버, 카카오, 토스, 당근, 쿠팡, 배민 등 국내 주요 기업들의 실제 AI/LLM 도입 및 자동화 사례 모음
>
> 출처: 각 기업 기술 블로그 및 공식 발표 (2024~2025)

---

## 목차

| 기업 | 핵심 사례 |
|------|----------|
| [네이버](#네이버) | HyperCLOVA X, AI 코드리뷰, 쇼핑라이브 대본 자동화 |
| [카카오](#카카오) | Kanana 스팸 분류, 카카오페이 AI봇, 카카오스타일 리뷰 검수 |
| [토스](#토스) | Software 3.0, MCP 서버, 그래픽 자동 생성 |
| [당근](#당근) | 프롬프트 스튜디오, 비개발자 AI 툴, 매물 요약봇 |
| [쿠팡](#쿠팡) | 생성형 AI 광고 플랫폼, 프롬프트 캐싱, AI 코드 작성 50% |
| [배달의민족](#배달의민족) | LLMOps 2.0, AI 데이터 분석가, 허위 리뷰 필터 |
| [컬리](#컬리) | Vertex AI Search, Gemini 리뷰 분석 |
| [딜라이트룸 (알라미)](#딜라이트룸-알라미) | Claude Code로 IaC Drift 해결 — 8주 작업을 며칠로 ⭐ |
| [공통 패턴](#공통-패턴--aiops) | AIOps, LLMOps, Human-in-the-loop |

---

## 네이버

### 1. AI 코드 리뷰 자동화

**문제:** 수십 개 팀의 PR 코드 리뷰에 시니어 엔지니어 시간이 과도하게 소비됨

**해결책:** Llama3 기반 AI 코드 리뷰어 도입

```
개발자가 PR 생성
    ↓
AI가 변경 코드 자동 분석
    ↓
라인별 코멘트 자동 생성
예: "스레드 슬립을 사용한 리트라이 구현은
    예측 불가능한 지연을 초래합니다.
    exponential backoff 사용을 권장합니다"
    ↓
개발자가 피드백 검토 후 반영
```

**기술 스택:** Llama3 (자체 인프라 배포), GitLab CI 훅으로 PR 이벤트 트리거

---

### 2. 쇼핑라이브 AI 큐시트 헬퍼

**문제:** 중소 셀러들이 라이브 커머스 대본 작성에 평균 2~3시간 소요

**해결책:** HyperCLOVA X 기반 대본 자동 생성

```python
# 프롬프트 구조 (예시)
system = """당신은 쇼핑라이브 대본 전문 작가입니다.
주어진 상품 정보를 바탕으로 매력적인 라이브 대본을 작성하세요.
- 오프닝 30초, 상품 소개 3분, 혜택 강조 1분 구조
- 시청자 참여 유도 멘트 포함
- 자연스러운 구어체 사용"""

user = f"""
상품명: {product_name}
카테고리: {category}
주요 특징: {features}
가격/혜택: {price_info}
타겟 고객: {target}

위 정보로 라이브 대본을 작성해줘.
"""
```

**결과:** 대본 초안 작성 2~3시간 → **1분 이내**, 중소 셀러 라이브 진입 장벽 대폭 감소

---

### 3. CLOVA Studio — 프롬프트 기반 AI 서비스 플랫폼

코딩/AI 지식 없이도 HyperCLOVA X를 프롬프트만으로 커스터마이징하고 API로 서비스에 바로 적용할 수 있는 플랫폼.

**활용 사례:**
- 공공기관 '서울데이터허브' AI 챗봇
- 건강검진 결과 안내 챗봇 '에스크미'
- 지원사업 검색 플랫폼 '경기기업비서'

```
CLOVA Studio 워크플로우:
[프롬프트 작성] → [모델 파라미터 조정]
      ↓
[테스트 & 평가] → [API 자동 발급]
      ↓
[서비스에 연동] (코드 몇 줄만으로)
```

**참고:** [HyperCLOVA X](https://clova.ai/hyperclova) | [CLOVA Studio](https://clova.ai/clova-studio)

---

## 카카오

### 1. Kanana — 자체 LLM으로 스팸 콘텐츠 분류

**문제:** 카카오 플랫폼 내 스팸/불법 콘텐츠 급증, 외부 LLM 사용 시 보안/규제 문제

**해결책:** 자체 개발 Kanana 모델로 스팸 분류기 구축

```python
# 카카오 내부 분류 구조 (공개 정보 기반 재구성)
def classify_content(content: str) -> dict:
    response = kanana_client.messages.create(
        model="kanana-essence",   # 카카오 자체 모델
        messages=[{
            "role": "user",
            "content": f"""다음 콘텐츠가 스팸/불법인지 판정해줘.

콘텐츠: {content}

판정 결과를 다음 JSON 형식으로 출력해줘:
{{
  "is_spam": true/false,
  "confidence": 0.0~1.0,
  "category": "광고성/사기/불법정보/정상",
  "reason": "판정 이유 한 문장"
}}"""
        }]
    )
    return json.loads(response.content[0].text)
```

**핵심:** 판정 **이유까지 자동 생성** → 운영팀 검토 효율 대폭 향상, 규제 준수(데이터 외부 유출 없음)

**참고:** [Kanana Model Family 소개](https://www.kakaocorp.com/page/detail/11334)

---

### 2. 카카오페이 — 업무도우미 AI봇 "춘식이"

**문제:** 사내 정책/규정 문서가 방대해 직원들이 정보 찾는 데 시간 낭비

**해결책:** AWS Bedrock + Claude 기반 RAG 사내 챗봇

```python
# 카카오페이 AI봇 아키텍처 (기술블로그 기반)
import boto3
from anthropic import AnthropicBedrock

# AWS Bedrock을 통해 Claude 호출 (서울 리전)
bedrock_client = AnthropicBedrock(
    aws_region="ap-northeast-2"
)

def answer_policy_question(question: str) -> str:
    # 1. 사내 문서 벡터 검색
    relevant_docs = vector_store.search(question, k=5)

    # 2. Claude로 답변 생성 (Converse API 사용)
    response = bedrock_client.messages.create(
        model="anthropic.claude-3-sonnet-20240229-v1:0",
        max_tokens=1024,
        system="""당신은 카카오페이 사내 정책 전문가입니다.
주어진 문서를 기반으로만 답변하세요.
문서에 없는 내용은 '확인이 필요합니다'라고 답하세요.""",
        messages=[{
            "role": "user",
            "content": f"""참고 문서:
{chr(10).join(relevant_docs)}

질문: {question}"""
        }]
    )
    return response.content[0].text
```

**기술 스택:** AWS Bedrock (Claude 3 Sonnet), Amazon OpenSearch (벡터 DB), AWS Converse API

**결과:** 직원 정책 문의 처리 시간 단축, 24/7 자동 응답

**참고:** [카카오페이 기술블로그 - 업무도우미 AI봇](https://tech.kakaopay.com/post/choonsiri/)

---

### 3. 카카오스타일(지그재그) — AI 리뷰 검수 자동화

**문제:** 하루 수만 건 상품 리뷰(텍스트+이미지) 수동 검수 — 인력 비용 폭증, 정책 변경 시 재교육 필요

**해결책:** Amazon Bedrock + Claude 3 Sonnet으로 멀티모달 1차 검수 자동화

```python
# 지그재그 리뷰 검수 파이프라인
def review_inspection(review_text: str, review_images: list) -> dict:
    # Claude 3 Sonnet: 텍스트 + 이미지 동시 분석 (멀티모달)
    content = [
        {
            "type": "text",
            "text": f"""다음 상품 리뷰를 검수 기준에 따라 판정해줘.

검수 기준:
- 광고성 문구 포함 여부
- 부적절한 이미지 여부
- 허위/과장 표현 여부
- 개인정보 노출 여부

리뷰 텍스트: {review_text}

판정 결과 JSON으로 출력:
{{"pass": true/false, "issues": [], "confidence": 0.0~1.0}}"""
        }
    ]

    # 이미지가 있는 경우 추가
    for img_base64 in review_images:
        content.append({
            "type": "image",
            "source": {"type": "base64", "media_type": "image/jpeg", "data": img_base64}
        })

    response = bedrock_client.messages.create(
        model="anthropic.claude-3-sonnet-20240229-v1:0",
        max_tokens=256,
        messages=[{"role": "user", "content": content}]
    )
    return json.loads(response.content[0].text)

# Human-in-the-loop: AI 1차 → 사람 2차
def full_review_pipeline(review):
    ai_result = review_inspection(review.text, review.images)

    if ai_result["confidence"] >= 0.95:
        return ai_result   # 고신뢰 → AI 단독 처리
    else:
        return human_review_queue.enqueue(review, ai_result)  # 저신뢰 → 사람 검토
```

**결과:**
- 정책 변경 시 프롬프트 수정만으로 즉시 대응 (재교육 불필요)
- 검수 처리량 대폭 증가
- 운영팀은 저신뢰 케이스만 집중 검토

**참고:** [카카오스타일 Bedrock 도입 사례 - AWS 기술블로그](https://aws.amazon.com/ko/blogs/tech/kakaostyle-leverage-genai-with-amazon-bedrock/)

---

## 토스

### 1. Software 3.0 — 프롬프트가 프로그램이 되는 시대

토스가 선언한 개발 패러다임 전환.

```
Software 1.0: 규칙 기반 코드 작성 (if/else, 알고리즘)
Software 2.0: 데이터로 학습한 ML 모델
Software 3.0: 자연어(프롬프트)로 원하는 것을 말하면 AI가 처리
              → 프롬프트 = 프로그램
```

**실제 적용:**
- 상담 챗봇 → 규칙 기반 시나리오 → LLM 자연어 처리로 전환
- 이상 거래 탐지 → 규칙 엔진 → LLM 기반 패턴 인식으로 보완
- 에러 분석 → 수동 로그 분석 → LLM이 자동으로 원인 파악 및 해결 방향 제시

**참고:** [토스 기술블로그 - Software 3.0 시대](https://toss.tech/article/software-3-0-era)

---

### 2. 토스페이먼츠 MCP 서버 — 결제 연동 10분 완료

**문제:** 결제 API 연동에 개발자가 평균 수 시간~수 일 소요 (문서 찾기 + 코드 작성)

**해결책:** MCP(Model Context Protocol) 서버로 토스페이먼츠 전체 API 문서를 AI에 제공

```json
// Claude Code settings.json에 추가
{
  "mcpServers": {
    "tosspayments": {
      "command": "npx",
      "args": ["-y", "@tosspayments/mcp-server"],
      "env": {
        "TOSS_SECRET_KEY": "test_sk_..."
      }
    }
  }
}
```

```
Claude Code에서:
"토스페이먼츠로 카드 결제 기능 추가해줘"
    ↓
MCP 서버가 API 문서, 실전 예제, 엣지 케이스까지 Claude에 전달
    ↓
Claude가 프로젝트 맥락에 맞는 정확한 코드 생성
    ↓
결제 연동 완료 (10분)
```

**결과:** 결제 API 연동 시간 수 시간 → **10분 이내**

**참고:** [토스페이먼츠 MCP로 결제 연동 혁신](https://www.tosspayments.com/blog/articles/mcp)

---

### 3. Midjourney로 앱 그래픽 자동 생성

**문제:** 토스 앱 내 이미지/아이콘 요청이 그래픽 디자이너 병목을 만듦

**해결책:** Midjourney + 자체 스타일 가이드 프롬프트로 디자인 자동화

```
프롬프트 구조:
"[객체], toss app style, clean minimal design,
 blue color palette, flat illustration,
 white background, [해상도]"

결과:
- 디자이너 요청 없이 PM/개발자가 직접 생성
- A/B 테스트용 이미지 빠르게 실험
- 실험 속도 대폭 향상
```

**참고:** [달파 블로그 - 빅테크 AI 활용 사례](https://app.dalpha.so/blog/ai-usecase-tech/)

---

## 당근

### 1. 프롬프트 스튜디오 — AI 기능 개발 민주화

**개념:** 개발자가 아닌 PM, 디자이너, 운영팀도 LLM 기반 기능을 직접 만들 수 있는 사내 AI 플랫폼

```
기존:
기획 → 개발팀 요청 → 구현 대기 (수 주)

프롬프트 스튜디오:
기획 → 프롬프트 작성 → 즉시 API 연결 → 서비스 적용 (수 시간)
```

**주요 기능:**
- 다양한 LLM 모델 비교 테스트
- 프롬프트 버전 관리 (변경 이력 추적)
- A/B 테스트 내장
- 성능 평가 데이터셋 관리

---

### 2. AI 매물 정보 요약

**문제:** 부동산 매물 게시글이 장문이라 사용자가 필요한 정보를 찾기 어려움

**해결책:** 사용자의 필터/알림 조건을 기반으로 개인화된 요약 자동 생성

```python
def summarize_listing(listing: dict, user_preferences: dict) -> str:
    prompt = f"""부동산 매물 정보를 사용자 관심사에 맞게 3줄로 요약해줘.

매물 정보:
{listing['description']}

사용자 관심사:
- 예산: {user_preferences['budget']}
- 선호 조건: {user_preferences['conditions']}
- 알림 설정: {user_preferences['alerts']}

사용자가 가장 궁금해할 정보를 중심으로 간결하게 요약해줘."""

    # 실제 구현에서는 당근 내부 LLM 또는 외부 API 사용
    return llm_client.generate(prompt)
```

**결과:** 매물 상세 페이지 체류 시간 증가, 문의 전환율 향상

---

### 3. 비개발자의 AI 도구 개발 (AI Show & Tell)

당근 내부 'AI Show & Tell' 행사에서 발표된 비개발자 직군의 AI 도구 자체 제작 사례:

| 직군 | 만든 도구 | 활용 기술 |
|------|----------|---------|
| 운영팀 | 정규표현식 생성기 | Cursor + 프롬프트 → JS 정규식 자동 생성 |
| 정책팀 | 폴리시 체커 | AI가 내부 정책 문서 분석 → 개선점 제안 |
| 데이터팀 | 슬랙 요약봇 | 핵심 지표 변화를 매일 자동 요약 (30분 → 3분) |
| 디자인팀 | 스타일 가이드 검사기 | 디자인 파일 → AI가 가이드 준수 여부 체크 |

**핵심 인사이트:** "AI 도구 개발은 더 이상 개발자만의 영역이 아니다"

**참고:** [당근 AI Show & Tell #1 - 비개발자 AI 도전기](https://medium.com/daangn/ai-%ED%88%B4-%EA%B0%9C%EB%B0%9C%EC%9D%80-%EC%B2%98%EC%9D%8C%EC%9D%B4%EB%9D%BC-%EB%8B%B9%EA%B7%BC-%EB%B9%84%EA%B0%9C%EB%B0%9C%EC%9E%90-%EA%B5%AC%EC%84%B1%EC%9B%90%EB%93%A4%EC%9D%98-ai-%EB%8F%84%EC%A0%84%EA%B8%B0-fb62d2a6c2f3)

---

## 쿠팡

### 1. 생성형 AI 광고 플랫폼 (AWS Summit 2025 발표)

**문제:** 광고 성과 분석 → 최적화 제안 → 실행의 반복 작업을 사람이 수동으로 처리

**해결책:** LLM 기반 광고 에이전트로 분석-제안-실행 자동화

```
광고 에이전트 아키텍처:

[실시간 광고 지표 데이터]
        ↓
[LLM 에이전트]
  - 지표 분석 (CTR, ROAS, 노출 등)
  - 최적화 방향 추론
  - 구체적 실행 제안 생성
        ↓
[Score 시스템] — 응답 품질 자동 평가
  - 수치 정보 정확도 검증
  - 실제 데이터와 일치율 점수화
        ↓
[승인 또는 자동 실행]
```

**프롬프트 캐싱으로 비용 최적화:**

```python
# 동일/유사 요청에 이전 응답 재활용
class PromptCache:
    def __init__(self):
        self.cache = {}  # 또는 Redis

    def get_or_generate(self, prompt: str) -> str:
        cache_key = self.hash_prompt(prompt)

        if cache_key in self.cache:
            return self.cache[cache_key]   # 캐시 히트

        response = llm_client.generate(prompt)
        self.cache[cache_key] = response
        return response

# 결과: 긴 프롬프트 요청에서
# - 비용 최대 90% 절감
# - 응답 속도 85% 향상
```

**결과:**
- 반복 광고 분석 작업 자동화
- 프롬프트 캐싱으로 **비용 90% 절감**, **응답 속도 85% 향상**

---

### 2. AI 코드 작성 50%

```
2025년 기준:
쿠팡 신규 개발 코드의 약 50%를 AI가 작성

적용 범위:
- 물류 시스템 백엔드 코드
- 광고 플랫폼 기능 개발
- 데이터 파이프라인 스크립트
```

**활용 도구:** GitHub Copilot, Cursor, 사내 AI 코딩 어시스턴트

**참고:** [쿠팡 생성형AI 광고플랫폼 혁신 - AWS Summit 2025](https://iting.co.kr/aws-summit-seoul-2025-techblog-7/)

---

## 배달의민족

### 1. LLMOps 플랫폼 2.0

**문제:** LLM 프로젝트가 늘어나자 MLE/DS만으로는 감당 불가, 관리 체계 부재

**해결책:** 전사 LLMOps 플랫폼 구축

```
LLMOps 플랫폼 2.0 핵심 기능:

[실험 관리]
- 프롬프트 버전 관리 (변경 이력 추적)
- 다양한 LLM 모델 빠른 비교 실험

[평가 시스템]
- 데이터셋 + 평가 기준 관리
- 자동화된 품질 측정

[배포 파이프라인]
- 실험 → 스테이징 → 프로덕션 자동 흐름
- Studio와 긴밀한 연동

[모니터링]
- 실시간 성능 추적
- 이상 감지 알림
```

**성과 지표 (2025년 기준):**
- 사용자 수 전년 대비 **50% 증가**
- LLM 프로젝트 수 **69% 성장**
- MLE/DS 외 직군(PM, 프론트, 백엔드)도 AI 모델 활용 가능

**참고:** [LLMOps로 확장하는 AI플랫폼 2.0 - 우아한형제들 기술블로그](https://techblog.woowahan.com/22839/)

---

### 2. AI 데이터 분석가

**문제:** 데이터 분석 요청이 폭주해 데이터팀 병목 발생

**해결책:** LangChain 기반 자연어 → SQL 자동 변환 분석 시스템

```python
from langchain_anthropic import ChatAnthropic
from langchain.agents import create_sql_agent
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("postgresql://...")
llm = ChatAnthropic(model="claude-sonnet-4-6")

# SQL 에이전트 생성
agent = create_sql_agent(
    llm=llm,
    db=db,
    verbose=True,
    system_message="""당신은 배달의민족 데이터 분석가입니다.
자연어 질문을 SQL로 변환하고 결과를 비즈니스 관점에서 해석하세요.
테이블 구조: orders, restaurants, users, reviews, deliveries"""
)

# 활용 예
result = agent.invoke(
    "지난주 대비 이번주 치킨 카테고리 주문 증감률과
     주요 증감 지역 TOP 5를 알려줘"
)
# → SQL 자동 생성 → 실행 → 자연어 해석 결과 반환
```

**결과:** PM/마케터도 데이터팀 없이 직접 분석 가능

---

### 3. 허위 리뷰 필터링 AI

```
탐지 항목:
- 동일 IP/기기에서 반복 리뷰
- 주문 이력 없는 리뷰
- 비정상 패턴 (시간, 키워드, 평점 분포)
- 광고성 문구

기술:
- ML 분류 모델 + LLM 판정 이유 생성
- 실시간 스트림 처리 (Kafka + Flink)
- 인간 검토 대기열 자동 분류
```

**참고:** [배민 AI 서비스와 MLOps 도입기](https://techblog.woowahan.com/11582/)

---

## 컬리

### 1. Vertex AI Search — 검색 품질 개선

**문제:** 검색 전체 쿼리의 6%가 "결과 없음" → 이탈률 증가

**원인:** 오타, 유의어, 영어/한글 혼용, 띄어쓰기 오류

**해결책:** Google Vertex AI Search 도입

```
기존 검색:
"파프리가" → 결과 없음
"토마토케찹" → 결과 없음
"비빔밥소스" → 결과 없음

Vertex AI Search 적용 후:
"파프리가" → "파프리카" 자동 교정 → 결과 표시
"토마토케찹" → "토마토 케첩" 의미 인식 → 결과 표시
"비빔밥소스" → 관련 상품 의미 검색 → 결과 표시
```

**결과:**
- 검색 무결과율 대폭 감소
- 장바구니 전환율 향상
- 장바구니 금액 증가

---

### 2. Gemini 기반 리뷰 분석 자동화

**문제:** 하루 수천 건 리뷰 수동 분석 → 상품 개선 피드백 반영 지연

```python
# 컬리 리뷰 분석 파이프라인 (개념 재구성)
def analyze_product_reviews(product_id: str, reviews: list[str]) -> dict:
    batch_reviews = "\n".join([f"- {r}" for r in reviews[:50]])

    response = gemini_client.generate_content(f"""
다음 상품 리뷰들을 분석해줘.

리뷰 목록:
{batch_reviews}

분석 결과를 다음 형식으로 출력해줘:
1. 주요 긍정 키워드 (TOP 5)
2. 주요 부정 키워드 (TOP 5)
3. 카테고리별 감정 분류 (맛/신선도/배송/가격/포장)
4. 개선 필요 사항 요약
""")
    return parse_analysis(response.text)
```

**결과:** 리뷰 분석 주기 단축, 상품 소싱팀에 실시간 피드백 제공

---

## 딜라이트룸 (알라미)

> 출처: [AI로 우리 회사 인프라 코드 완벽 관리하기 — 딜라이트룸 기술블로그](https://medium.com/delightroom/ai%EB%A1%9C-%EC%9A%B0%EB%A6%AC-%ED%9A%8C%EC%82%AC-%EC%9D%B8%ED%94%84%EB%9D%BC-%EC%BD%94%EB%93%9C-%EC%99%84%EB%B2%BD-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0-c9f5cb7f2ef6)

### Claude Code로 IaC Drift 해결 (Pulumi)

**배경:** 딜라이트룸은 월 460만 MAU 알라미 앱을 운영. `pulumi preview` 실행 시 수십 개의 변경사항이 터미널을 가득 채웠다. 콘솔에서 급하게 수정한 설정들이 오랜 시간 쌓여 코드와 실제 AWS 리소스가 불일치하는 **Drift** 상태.

#### IaC Drift란

```
코드 (Code)        ←→     상태 (State)     ←→     리소스 (Resource)
  S3 bucket exists         exists               deleted in console
  Lambda exists            exists               config changed manually

→ 이 세 가지가 일치해야 IaC가 "설계도" 역할을 한다.
  하나라도 어긋나면 pulumi up 실행 시 서비스 장애 위험.
```

**문제의 악순환:**
```
Drift 발견
  → "지금 건드리면 더 위험해" 방치
  → 콘솔에서 또 수동 작업
  → Drift 더 커짐
  → 코드 신뢰 불가, 콘솔 의존 심화
```

#### 해결 전략: 실제 리소스를 Source of Truth로

```
두 가지 해결 방법:
1. pulumi up → 코드를 클라우드에 강제 적용 (❌ 운영 리소스 삭제 위험)
2. 코드와 상태를 실제 리소스에 맞게 수정 (✅ 선택)

원칙:
- 실제 리소스는 절대 건드리지 않는다
- 코드·상태를 현재 클라우드에 맞게 조정한다
- pulumi preview 변경사항 0개가 목표
```

#### .claude/ 디렉토리 구조

```
.claude/
├── rules/
│   ├── pulumi-safety.md        # pulumi up 금지 등 안전장치
│   ├── code-conventions.md     # 코드 스타일 일관성
│   └── drift-resolution.md     # Drift 유형별 대응 기준
├── skills/
│   ├── drift-status/SKILL.md   # preview → 현황 리포트
│   ├── drift-import/SKILL.md   # + create Drift 해결
│   ├── drift-remove/SKILL.md   # - delete Drift 해결
│   └── drift-fix/SKILL.md      # ~ update Drift 해결
└── agents/
    └── drift-resolver.md       # 루프 관리 + 서브에이전트 조율
```

#### rules/pulumi-safety.md

```markdown
---
paths:
  - "clusters/*/pulumi/**"
  - "external-services/*/pulumi/**"
  - "modules/**"
---

# Pulumi Safety Rules

## Core Principles

1. **Never run `pulumi up`** — Only `pulumi preview` is allowed
2. **Actual resources are Source of Truth** — Modify code to match resources
3. **Bidirectional stack isolation** — Dev and Prod must never affect each other

| Command        | Allowed  | Notes                              |
|----------------|----------|------------------------------------|
| `pulumi preview` | Yes    | 항상 안전                           |
| `pulumi import`  | Yes    | 클라우드 리소스 변경 없음              |
| `pulumi state delete` | Caution | AWS에서 먼저 삭제 확인 후에만        |
| `pulumi up`      | No     | 절대 금지                           |
| `pulumi destroy` | No     | 절대 금지                           |

## When to Ask User
- 보안 그룹·IAM 변경 감지 시
- 해결 방법이 여러 가지일 때
- AWS 상태가 예상과 다를 때
- 운영 스택에서 예상치 못한 변경 감지 시
```

#### rules/drift-resolution.md

```markdown
# Drift Resolution Rules

## Drift 유형별 대응

| Symbol | 상황 | 대응 |
|--------|------|------|
| `+` create | 코드에 있음, AWS에 없음 | AWS 확인 → 있으면 `pulumi import`, 없으면 코드 삭제 |
| `-` delete | AWS에 있음, 코드에 없음 | AWS 확인 → 삭제됐으면 `pulumi state delete`, 있으면 코드 추가 |
| `~` update | 값이 서로 다름 | AWS 값이 올바름 → 코드를 AWS에 맞게 수정 |

## ignoreChanges 사용 기준
- **허용:** 외부 시스템이 관리하는 필드 (오토스케일러의 desiredCapacity 등)
- **금지:** Drift를 숨기기 위한 목적
- **필수:** 사용 이유 주석 문서화
```

#### skills/drift-status/SKILL.md

```markdown
---
name: drift-status
description: 양쪽 스택 pulumi preview 실행 후 Drift 현황 리포트 생성
---

## Steps
1. dev 스택: `pulumi preview --stack dev 2>&1`
2. prod 스택: `pulumi preview --stack prod 2>&1`
3. 결과 분석 및 리포트 생성

## Output Format

## Drift Status Report

### Dev Stack
| Type | Count | Resources |
|------|-------|-----------|
| + create | N | resource1, resource2 |
| - delete | N | resource3 |
| ~ update | N | resource4 |

### Prod Stack
(동일 형식)

### Recommended Actions
- `+` create → drift-import skill
- `-` delete → drift-remove skill
- `~` update → drift-fix skill
```

#### skills/drift-import/SKILL.md (`+` create 처리)

```markdown
---
name: drift-import
description: 코드에 있지만 Pulumi 상태에 없는 리소스를 import
---

## Workflow
1. AWS CLI로 리소스 실제 존재 확인
2. 존재하면 pulumi import 실행
3. 자동 생성 코드를 기존 코드베이스에 통합
4. 양쪽 스택에서 preview 검증

## Resource ID 형식

| 리소스 타입 | Pulumi Type | ID 형식 |
|------------|-------------|---------|
| S3 Bucket | `aws:s3/bucket:Bucket` | bucket-name |
| EC2 Instance | `aws:ec2/instance:Instance` | i-xxxxxxxxx |
| Security Group | `aws:ec2/securityGroup:SecurityGroup` | sg-xxxxxxxx |
| IAM Role | `aws:iam/role:Role` | role-name |
| Lambda | `aws:lambda/function:Function` | function-name |
| RDS | `aws:rds/instance:Instance` | db-identifier |

## Example
```bash
# 1. AWS에 실제 존재 확인
aws s3api head-bucket --bucket my-bucket

# 2. Pulumi 상태로 가져오기
pulumi import aws:s3/bucket:Bucket my-bucket my-bucket --stack dev
```
```

#### skills/drift-remove/SKILL.md (`-` delete 처리)

```markdown
---
name: drift-remove
description: AWS에서 이미 삭제된 리소스를 Pulumi 상태에서 제거
---

## Workflow
1. AWS CLI로 리소스 삭제 여부 확인 (404/not found 응답 필수)
2. URN 조회: `pulumi stack --show-urns | grep "resource-name"`
3. 상태에서 제거: `pulumi state delete "<URN>"`
4. 코드에서도 제거
5. 양쪽 스택 preview 검증

## URN 형식
`urn:pulumi:<stack>::<project>::<type>::<name>`

## ⚠️ 주의
AWS에서 실제 삭제 확인 전에는 절대 state delete 금지.
리소스가 존재하는데 state에서만 지우면 연결이 끊어져 더 큰 Drift 발생.
```

#### skills/drift-fix/SKILL.md (`~` update 처리)

```markdown
---
name: drift-fix
description: 코드와 AWS 설정값 불일치를 코드 수정으로 해결
---

## Workflow
1. `pulumi preview --diff`로 정확히 어떤 속성이 다른지 확인
2. AWS CLI로 현재 실제 값 조회
3. 코드를 AWS 값에 맞게 수정
4. 양쪽 스택 preview 검증

## 자주 나오는 패턴

| 상황 | 처리 |
|------|------|
| 태그 누락 | AWS 태그 값을 코드에 추가 |
| 타임아웃·메모리 값 다름 | AWS 현재 값으로 코드 업데이트 |
| 외부 시스템 관리 필드 | ignoreChanges + 사유 주석 |
| 어떤 값이 맞는지 불명확 | 사용자에게 확인 요청 |

## Decision Guide
- AWS 값이 의도적으로 변경된 것이면 → 코드 수정
- 외부에서 관리되는 필드면 → ignoreChanges (이유 주석 필수)
- 불명확하면 → 사용자 에스컬레이션
```

#### agents/drift-resolver.md

```markdown
---
name: drift-resolver
description: pulumi preview 변경사항 0개 달성까지 자율 반복
---

## Goal
개발·운영 양쪽 스택 모두 `pulumi preview` 변경사항 = 0

## Workflow
1. drift-status skill → 전체 Drift 목록 수집
2. Todo list 생성 (유형별 분류)
3. 각 Drift 처리:
   - `+` create → drift-import skill
   - `-` delete → drift-remove skill
   - `~` update → drift-fix skill
4. 매 수정 후 **양쪽 스택** preview 검증 (한쪽만 하면 공유 모듈 간섭 발생)
5. 0개 될 때까지 반복
6. 완료 시 최종 리포트 생성

## Subagent 구조
같은 디렉토리 내 여러 Drift는 서브에이전트로 분할:
- Subagent 1: `+` create 유형 담당
- Subagent 2: `~` update 유형 담당
- **순차 실행 필수** (동시 코드 수정 충돌 방지)
- 메인 에이전트가 전체 상태 재검증 담당

## Escalation Triggers
- 보안 그룹 / IAM 변경 감지
- 해결 방법이 여러 가지인 경우
- AWS 상태가 예상과 다른 경우
- 운영 스택 예기치 못한 변경 감지

## Termination
| Status | Condition |
|--------|-----------|
| SUCCESS | 양쪽 스택 모두 변경사항 0개 |
| PARTIAL | 일부 해결, 나머지 사용자 판단 필요 |
| BLOCKED | 사용자 입력 없이 진행 불가 |
```

#### 결과

| 항목 | Before | After |
|------|--------|-------|
| pulumi preview 결과 | 수십 개 변경사항 | **0개** |
| 예상 수작업 시간 | 6~8주 | **며칠** |
| 팀 LLMOps 컨벤션 | 없음 | `.claude/` 팀 공용 자산으로 정착 |
| 인프라 코드 신뢰도 | 낮음 (콘솔 의존) | **높음 (코드가 설계도 역할 회복)** |

**핵심 인사이트:**
> "AI Agent가 모든 것을 대신해 주지는 않는다. 어떤 규칙을 정할지, 예외 상황에서 어떻게 판단할지는 여전히 사람의 몫이다. 중요한 것은 Agent에게 모든 것을 맡기는 것이 아니라 **어떻게 함께 일할 것인지를 설계하는 것**이다." — 딜라이트룸 기술블로그

---

### Terraform 버전 — 동일 패턴 적용

Pulumi 대신 **Terraform**을 쓰는 팀을 위한 동등 구현. 명령어만 다를 뿐 구조와 원칙은 동일하다.

#### Pulumi vs Terraform 명령어 대응표

| 역할 | Pulumi | Terraform |
|------|--------|-----------|
| Drift 확인 | `pulumi preview` | `terraform plan` |
| 변경 적용 (금지) | `pulumi up` | `terraform apply` |
| 리소스 가져오기 | `pulumi import` | `terraform import` |
| 상태에서 제거 | `pulumi state delete` | `terraform state rm` |
| 상태 새로고침 | `pulumi refresh` | `terraform plan -refresh-only` |
| 리소스 목록 | `pulumi stack --show-urns` | `terraform state list` |
| 리소스 상세 | - | `terraform state show <address>` |

#### .claude/ 디렉토리 구조 (Terraform)

```
.claude/
├── rules/
│   ├── terraform-safety.md
│   ├── code-conventions.md
│   └── drift-resolution.md
├── skills/
│   ├── drift-status/SKILL.md
│   ├── drift-import/SKILL.md
│   ├── drift-remove/SKILL.md
│   └── drift-fix/SKILL.md
└── agents/
    └── drift-resolver.md
```

#### rules/terraform-safety.md

```markdown
---
paths:
  - "terraform/**"
  - "infra/**"
  - "modules/**"
  - "**/*.tf"
---

# Terraform Safety Rules

## Core Principles

1. **Never run `terraform apply`** — `terraform plan`으로만 검증
2. **Never run `terraform destroy`** — 절대 금지
3. **Actual AWS resources are Source of Truth** — 코드를 리소스에 맞춤
4. **Always run `terraform plan` on ALL workspaces after any change**

| Command | Allowed | Notes |
|---------|---------|-------|
| `terraform plan` | Yes | 항상 안전 |
| `terraform plan -refresh-only` | Yes | 상태 새로고침 확인 |
| `terraform import` | Yes | 클라우드 리소스 변경 없음 |
| `terraform state list` | Yes | 읽기 전용 |
| `terraform state show` | Yes | 읽기 전용 |
| `terraform state rm` | Caution | AWS 삭제 확인 후에만 |
| `terraform apply` | No | 절대 금지 |
| `terraform destroy` | No | 절대 금지 |
| `terraform apply -target` | No | 절대 금지 |

## Workspace Isolation
- dev, staging, prod 워크스페이스는 서로 영향 금지
- 코드 수정 후 반드시 모든 워크스페이스에서 `terraform plan` 실행
- 공유 모듈 수정 시 모든 워크스페이스 검증 필수

## When to Ask User
- `lifecycle { prevent_destroy = true }` 리소스 변경 시
- security group / IAM policy / KMS key 변경 감지 시
- 여러 해결 방법 중 선택이 필요한 경우
- 의존성 있는 리소스 삭제 순서 판단 필요 시
```

#### rules/drift-resolution.md (Terraform)

```markdown
# Terraform Drift Resolution Rules

## Drift 유형별 대응

| Plan 기호 | 상황 | 대응 전략 |
|-----------|------|----------|
| `+` (will be created) | .tf에 있음, AWS에 없음 | AWS 실제 존재 확인 → 있으면 `terraform import`, 없으면 .tf 코드 삭제 |
| `-` (will be destroyed) | AWS에 있음, .tf에 없음 | AWS 실제 삭제 여부 확인 → 삭제됐으면 `terraform state rm`, 있으면 .tf에 resource 블록 추가 |
| `~` (will be updated) | .tf 값 ≠ AWS 값 | AWS CLI로 현재 값 확인 → .tf 코드를 AWS 값으로 수정 |
| `-/+` (replacement) | 재생성 필요 | **무조건 사용자 확인** — 운영 리소스 다운타임 위험 |

## lifecycle 블록 사용 기준
```hcl
# 허용: 외부에서 관리되는 값
resource "aws_autoscaling_group" "app" {
  # ...
  lifecycle {
    # 오토스케일러가 desiredCapacity를 관리 — Terraform이 덮어쓰면 안 됨
    ignore_changes = [desired_capacity]
  }
}

# 금지: Drift를 숨기기 위한 목적으로 ignore_changes 남용
```

## `terraform refresh`는 언제 쓰나
- `plan`에서 실제 리소스 속성과 상태 파일이 달라 보일 때
- `terraform plan -refresh-only`로 상태 파일만 업데이트 (코드 수정 없음)
```

#### skills/drift-status/SKILL.md (Terraform)

```markdown
---
name: drift-status
description: 모든 워크스페이스 terraform plan 실행 후 Drift 현황 리포트
---

## Steps
1. 각 워크스페이스별 plan 실행:
   ```bash
   terraform workspace select dev && terraform plan -detailed-exitcode 2>&1
   terraform workspace select staging && terraform plan -detailed-exitcode 2>&1
   terraform workspace select prod && terraform plan -detailed-exitcode 2>&1
   ```
   (exit code: 0=변경없음, 1=에러, 2=변경있음)

2. 결과를 리포트로 정리

## Output Format

## Terraform Drift Status Report

### dev workspace
| Type | Resource Address | Detail |
|------|-----------------|--------|
| + create | aws_s3_bucket.logs | 코드에 있지만 AWS에 없음 |
| - destroy | aws_lambda_function.legacy | AWS에 있지만 코드에 없음 |
| ~ update | aws_security_group.app | ingress rules 불일치 |
| -/+ replace | aws_instance.web | AMI 변경 (재생성 필요 ⚠️) |

### Recommended Actions
- `+` create → drift-import skill
- `-` destroy → drift-remove skill
- `~` update → drift-fix skill
- `-/+` replace → **사용자 확인 필수**
```

#### skills/drift-import/SKILL.md (Terraform)

```markdown
---
name: drift-import
description: AWS에 존재하는 리소스를 Terraform state로 가져오기
---

## Workflow
1. AWS CLI로 리소스 실제 존재 확인
2. `terraform state list`로 현재 상태 확인
3. `terraform import <resource_address> <resource_id>` 실행
4. import 후 `terraform plan`으로 잔여 Drift 확인
5. 잔여 Drift 있으면 drift-fix skill로 코드 조정
6. 모든 워크스페이스에서 검증

## Resource ID 형식

| 리소스 타입 | Terraform Address | AWS ID 형식 |
|------------|------------------|------------|
| S3 Bucket | `aws_s3_bucket.name` | bucket-name |
| EC2 Instance | `aws_instance.name` | i-xxxxxxxxxxxxxxxxx |
| Security Group | `aws_security_group.name` | sg-xxxxxxxxxxxxxxxxx |
| IAM Role | `aws_iam_role.name` | role-name |
| Lambda | `aws_lambda_function.name` | function-name |
| RDS Instance | `aws_db_instance.name` | db-identifier |
| ECS Service | `aws_ecs_service.name` | cluster-name/service-name |
| ALB | `aws_lb.name` | arn:aws:elasticloadbalancing:... |

## Example
```bash
# 1. AWS에 버킷 실제 존재 확인
aws s3api head-bucket --bucket my-app-logs-prod

# 2. Terraform state로 가져오기
terraform workspace select prod
terraform import aws_s3_bucket.app_logs my-app-logs-prod

# 3. 잔여 Drift 확인
terraform plan
```

## import 후 코드 자동 생성 (Terraform 1.5+)
```bash
# import 블록으로 코드 자동 생성
cat >> main.tf << 'EOF'
import {
  to = aws_s3_bucket.app_logs
  id = "my-app-logs-prod"
}
EOF

terraform plan -generate-config-out=generated.tf
# generated.tf 검토 후 main.tf에 통합
```
```

#### skills/drift-remove/SKILL.md (Terraform)

```markdown
---
name: drift-remove
description: AWS에서 삭제된 리소스를 Terraform state에서 제거
---

## Workflow
1. AWS CLI로 리소스 실제 삭제 여부 확인
   (404 / ResourceNotFoundException / does not exist 응답)
2. `terraform state list | grep "resource-name"`으로 state 주소 확인
3. `terraform state rm <resource_address>` 실행
4. 코드(.tf 파일)에서 해당 resource 블록 삭제
5. 모든 워크스페이스에서 `terraform plan` 검증

## Example
```bash
# 1. 리소스 삭제 여부 확인
aws lambda get-function --function-name my-legacy-function
# → ResourceNotFoundException: Function not found ✅ 삭제됨

# 2. state 주소 확인
terraform state list | grep "legacy"
# aws_lambda_function.legacy_processor

# 3. state에서 제거
terraform workspace select dev
terraform state rm aws_lambda_function.legacy_processor

terraform workspace select prod
terraform state rm aws_lambda_function.legacy_processor

# 4. .tf 파일에서 resource 블록 삭제
```

## ⚠️ 의존성 확인
state rm 전에 반드시 확인:
- `terraform state show <address>`로 다른 리소스가 참조하는지 확인
- 참조가 있으면 참조하는 리소스도 함께 처리 필요
```

#### skills/drift-fix/SKILL.md (Terraform)

```markdown
---
name: drift-fix
description: .tf 코드와 AWS 실제 값의 불일치를 코드 수정으로 해결
---

## Workflow
1. `terraform plan -detailed-exitcode`로 정확한 diff 확인
2. `terraform state show <resource_address>`로 현재 state 값 확인
3. AWS CLI로 실제 리소스 현재 값 조회
4. .tf 코드를 AWS 값에 맞게 수정
5. 모든 워크스페이스에서 `terraform plan` 검증

## 자주 나오는 패턴

```hcl
# 패턴 1: 태그 불일치
resource "aws_s3_bucket" "app" {
  bucket = "my-app-bucket"

  tags = {
    Environment = "production"      # AWS에 이미 있는 태그를 코드에 추가
    ManagedBy   = "terraform"
    Team        = "platform"        # 누군가 콘솔에서 추가한 태그
  }
}

# 패턴 2: 외부 시스템이 관리하는 값
resource "aws_autoscaling_group" "app" {
  min_size         = 2
  max_size         = 10
  desired_capacity = 2

  lifecycle {
    # EKS Cluster Autoscaler가 이 값을 관리
    # Terraform이 덮어쓰면 스케일링 무력화됨
    ignore_changes = [desired_capacity]
  }
}

# 패턴 3: provider 버전 업 후 새로운 필수 필드
resource "aws_lambda_function" "processor" {
  function_name = "data-processor"
  runtime       = "nodejs18.x"
  handler       = "index.handler"

  # provider 업그레이드 후 새로 필요해진 필드
  architectures = ["x86_64"]   # AWS 기본값이지만 코드에 명시 필요
  package_type  = "Zip"
}
```

## Decision Guide
| 상황 | 처리 방법 |
|------|----------|
| AWS 값이 의도적으로 설정된 것 | 코드를 AWS 값으로 업데이트 |
| 외부 시스템(오토스케일러, K8s 등)이 관리 | `lifecycle { ignore_changes = [...] }` |
| 어느 값이 올바른지 불명확 | 사용자에게 에스컬레이션 |
| `-/+` replace 발생 | **무조건 사용자 확인** |
```

#### agents/drift-resolver.md (Terraform)

```markdown
---
name: drift-resolver
description: 모든 워크스페이스 terraform plan 변경사항 0개 달성까지 자율 반복
---

## Goal
모든 워크스페이스에서 `terraform plan` 종료 코드 = 0
(exit code 0 = "No changes. Infrastructure is up-to-date.")

## Workflow
1. drift-status skill → 전체 워크스페이스 Drift 목록
2. Todo list 생성 (우선순위: 독립 리소스 먼저, 의존성 있는 것 나중)
3. 각 Drift 처리:
   - `+` create → drift-import skill
   - `-` destroy → drift-remove skill
   - `~` update → drift-fix skill
   - `-/+` replace → **즉시 사용자 확인 요청**
4. 매 수정 후 **모든 워크스페이스** plan 검증
5. 0개 될 때까지 반복
6. 완료 시 최종 리포트 생성

## 중요: 공유 모듈 수정 시
modules/ 디렉토리 수정은 모든 워크스페이스에 영향.
한 워크스페이스에서만 검증하고 넘어가지 말 것.

## Subagent 구조 (Drift 수십 개일 때)
- Subagent A: 특정 디렉토리의 `+` create 처리
- Subagent B: 같은 디렉토리의 `~` update 처리
- **순차 실행 필수** — 동일 .tf 파일 동시 수정 충돌 방지
- 메인 에이전트: 각 서브에이전트 완료 후 전체 워크스페이스 재검증

## Escalation Triggers
- `-/+` replace (리소스 재생성) 감지 시
- `prevent_destroy = true` 리소스 변경 시
- security group / IAM / KMS 관련 변경 시
- 한 워크스페이스 수정이 다른 워크스페이스에 예상치 못한 영향을 줄 때
- 의존성 순환 또는 복잡한 ref 구조 발견 시

## Termination
| Status | Condition |
|--------|-----------|
| SUCCESS | 모든 워크스페이스 `terraform plan` exit code 0 |
| PARTIAL | 일부 해결, `-/+` replace 등 사용자 판단 필요 항목 존재 |
| BLOCKED | 사용자 입력 없이 진행 불가 |
```

#### 실제 사용 예시

```bash
# 1. Claude Code 실행
claude

# 2. drift-resolver 에이전트 호출
/drift-resolver

# 에이전트가 자동으로:
# → 모든 워크스페이스 terraform plan 실행
# → Drift 유형 분류 (총 47개: +12, -8, ~27)
# → drift-import/remove/fix skill 순차 적용
# → 각 수정 후 3개 워크스페이스 검증
# → 보안 그룹 변경 발견 → 사용자에게 확인 요청
# → 완료: 모든 워크스페이스 변경사항 0개
```

#### CI/CD 자동화 확장 (딜라이트룸 Next Step)

딜라이트룸이 계획 중인 CI/CD 연동을 Terraform으로 구현:

```yaml
# .github/workflows/terraform-drift-detect.yml
name: Terraform Drift Detection
on:
  push:
    paths: ['terraform/**', 'modules/**']
  schedule:
    - cron: '0 9 * * 1-5'   # 평일 매일 오전 9시

jobs:
  drift-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Terraform Plan (All Workspaces)
        id: plan
        run: |
          for ws in dev staging prod; do
            terraform workspace select $ws
            terraform plan -detailed-exitcode -out=plan-$ws.tfplan 2>&1
            echo "exit_$ws=$?" >> $GITHUB_OUTPUT
          done

      - name: Notify Slack if Drift Detected
        if: |
          steps.plan.outputs.exit_dev == '2' ||
          steps.plan.outputs.exit_staging == '2' ||
          steps.plan.outputs.exit_prod == '2'
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text": "⚠️ Terraform Drift 감지!\n`terraform plan`에서 변경사항이 발견되었습니다.\n확인 필요: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}'

      - name: Auto-fix with Claude Code (Non-breaking only)
        if: steps.plan.outputs.exit_prod == '2'
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # replace(-/+)가 없는 경우에만 자동 수정 시도
          claude --dangerously-skip-permissions \
            -p "terraform plan에서 replace 없이 update/create/delete만 있는 drift를 자동 수정해줘.
                replace가 발견되면 즉시 중단하고 보고해줘.
                수정 후 모든 워크스페이스에서 terraform plan이 0개여야 함."
```

---

## 공통 패턴 & AIOps

### 국내 기업들이 공통으로 사용하는 AI 자동화 패턴

#### 1. Human-in-the-loop

```
AI 1차 처리 (고속, 저비용)
       ↓
신뢰도 임계값 판단
 고신뢰 → 자동 처리 완료
 저신뢰 → 사람 검토 큐 → 최종 결정
       ↓
사람 결정 → AI 학습 데이터로 재활용
```

**채택 기업:** 카카오스타일(리뷰 검수), 쿠팡(광고 실행), 네이버(코드 리뷰)

---

#### 2. LLMOps 파이프라인

```
[실험 단계]                    [운영 단계]
프롬프트 작성                  버전 관리
   ↓                              ↓
모델 선택/비교         →      A/B 테스트
   ↓                              ↓
평가 데이터셋                 성능 모니터링
   ↓                              ↓
품질 기준 설정               이상 감지 & 알림
```

| 기업 | LLMOps 도구 |
|------|------------|
| 배민 | 자체 구축 AI 플랫폼 2.0 |
| 당근 | 프롬프트 스튜디오 (사내) |
| 카카오페이 | AWS Bedrock + LangSmith |
| 네이버 | CLOVA Studio |

---

#### 3. 프롬프트 버전 관리

```python
# 공통 패턴: 프롬프트를 코드처럼 관리
class PromptManager:
    def __init__(self):
        self.prompts = {}

    def register(self, name: str, version: str, template: str, metadata: dict):
        """프롬프트 버전 등록"""
        self.prompts[f"{name}:{version}"] = {
            "template": template,
            "metadata": metadata,
            "created_at": datetime.now(),
            "metrics": {}   # 성능 지표 누적
        }

    def get(self, name: str, version: str = "latest") -> str:
        if version == "latest":
            versions = [k for k in self.prompts if k.startswith(f"{name}:")]
            version = sorted(versions)[-1].split(":")[1]
        return self.prompts[f"{name}:{version}"]["template"]

    def log_performance(self, name: str, version: str, metrics: dict):
        """프롬프트 성능 추적"""
        self.prompts[f"{name}:{version}"]["metrics"].update(metrics)

# 사용 예
pm = PromptManager()
pm.register(
    name="review_classifier",
    version="v2.1",
    template="다음 리뷰를 분류해줘...",
    metadata={"model": "claude-sonnet-4-6", "use_case": "review_moderation"}
)
```

---

#### 4. 비용 최적화 공통 전략

| 전략 | 설명 | 절감 효과 |
|------|------|---------|
| **프롬프트 캐싱** | 동일 요청 응답 재사용 | 최대 90% (쿠팡 사례) |
| **모델 라우팅** | 복잡도에 따라 모델 선택 (Haiku/Sonnet/Opus) | 30~70% |
| **배치 처리** | 실시간 대신 묶음 처리 | 50~80% |
| **프롬프트 압축** | 컨텍스트 최소화 | 20~40% |
| **온디바이스 모델** | 간단한 작업은 소형 모델로 | 60~90% |

---

## 기술 블로그 링크 모음

| 기업 | 기술 블로그 |
|------|------------|
| 네이버 | [CLOVA AI Blog](https://clova.ai/hyperclova) |
| 카카오 | [kakao tech](https://tech.kakao.com) |
| 카카오페이 | [kakaopay tech](https://tech.kakaopay.com) |
| 토스 | [toss.tech](https://toss.tech) |
| 당근 | [Daangn Tech Blog](https://medium.com/daangn) |
| 쿠팡 | [Coupang Engineering](https://medium.com/coupang-engineering) |
| 배달의민족 | [우아한형제들 기술블로그](https://techblog.woowahan.com) |
| 컬리 | [Kurly Tech Blog](https://helloworld.kurly.com) |
| 카카오스타일 | [AWS 파트너 사례](https://aws.amazon.com/ko/blogs/tech/kakaostyle-leverage-genai-with-amazon-bedrock/) |
| 딜라이트룸 | [Delightroom Tech Blog (Medium)](https://medium.com/delightroom) |

---

*마지막 업데이트: 2026-03-02*
*출처: 각 기업 기술 블로그, AWS Summit 2025 발표, 공식 보도자료*
