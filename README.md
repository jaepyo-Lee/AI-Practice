# AI Practice Repository

AI 도구, 기법, 실제 적용 사례를 정리한 학습 레포지토리.

## 구조

```
AI-Practice/
├── claude-code/          # Anthropic Claude Code (CLI)
├── techniques/           # AI 핵심 기법들
├── korean-companies/     # 국내 기업 AI 자동화 실제 사례
└── README.md
```

## 폴더 설명

| 폴더 | 설명 | 주요 내용 |
|------|------|---------|
| [claude-code/](./claude-code/) | Claude Code CLI 완전 가이드 | 스킬 시스템, 멀티에이전트 패턴, 헤드리스 모드 |
| [techniques/](./techniques/) | AI 핵심 기법 정리 | RAG, MoE, Inference-Time Scaling 등 |
| [korean-companies/](./korean-companies/) | 국내 기업 실제 AI 사례 | 네이버, 카카오, 토스, 당근, 쿠팡, 배민 등 |

---

## korean-companies/ 주요 내용

| 기업 | 핵심 사례 | 기술 스택 |
|------|----------|---------|
| **네이버** | AI 코드리뷰, 쇼핑라이브 대본 자동화 | Llama3, HyperCLOVA X |
| **카카오** | 스팸 분류, 업무도우미봇, 리뷰 검수 | Kanana, Claude, Bedrock |
| **토스** | MCP 서버, Software 3.0, 그래픽 생성 | Claude MCP, Midjourney |
| **당근** | 프롬프트 스튜디오, 매물 요약, 비개발자 AI | 사내 플랫폼 |
| **쿠팡** | 광고 에이전트, 프롬프트 캐싱 90% 절감 | LLM + Score 시스템 |
| **배민** | LLMOps 2.0, AI 데이터 분석가 | LangChain, 자체 플랫폼 |
| **컬리** | Vertex AI Search, Gemini 리뷰 분석 | Google Cloud AI |

---

## claude-code/ 주요 내용

| 섹션 | 내용 |
|------|------|
| 스킬 시스템 | SKILL.md, 프론트매터, 플러그인 (v2.1.3+) |
| 헤드리스 & 배치 모드 | `-p` 플래그, 병렬 에이전트, CI/CD 연동 |
| Ralph Wiggum Loop | Stop 훅으로 자율 반복, 테스트 통과까지 루프 |
| Agent Teams | Coder + Reviewer + Tester 역할 분담 협력 |
| Adversarial Review | 두 AI가 서로 반박하며 품질 향상 |
| MCP | 300+ 외부 서비스 연동 |
| 훅 시스템 | PreToolUse, PostToolUse, Stop, SubagentStop |

---

## techniques/ 주요 내용

| 기법 | 카테고리 | 비고 |
|------|----------|------|
| RAG / Agentic RAG / GraphRAG | 지식 확장 | 단순 → 에이전트 → 그래프 진화 |
| Inference-Time Scaling | 추론 방법 | 2025 핵심 패러다임 전환 ⭐ |
| MoE (Mixture of Experts) | 아키텍처 | 프론티어 모델 60%+ 채택 ⭐ |
| Speculative Decoding | 추론 최적화 | 2-3배 속도 향상 ⭐ |
| Context Engineering | 에이전트 | 동적 컨텍스트 조립 ⭐ |
| ReAct / ToT / CoT | 추론 방법 | 에이전트 추론 기반 기법 |
| RLHF / DPO / CAI | 학습 방법 | 모델 정렬 기법 |
| Fine-tuning / LoRA | 학습 방법 | 도메인 특화 학습 |
| Embeddings & Vector DB | 데이터 | 의미 기반 검색 인프라 |

> ⭐ 2025~2026 신규/주요 업데이트

---

## 업데이트 방법

```bash
# 전체 리뷰
/ai-update

# 특정 문서만
/ai-update claude-code
/ai-update techniques

# 새 섹션 추가
/ai-update new [주제명]
```

---

*마지막 업데이트: 2026-03-02*
