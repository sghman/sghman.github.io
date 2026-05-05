---
title: "[Tech] 2026-05-05 기술 동향: LLM"
author: gyuhwan
date: 2026-05-05 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** LLM은 기본적으로 텍스트 예측 엔진일 뿐이므로, 실제 작업(웹 검색, 캘린더 조회, 예약, 이메일 발송)을 수행하려면 외부 시스템과의 연결이 필수다. 이를 위해 기본 Tool Use → Function Calling → Model Context Protocol(MCP)로 진화했으며, 현재 주요 AI 기업들이 MCP를 채택하고 있다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | LLM을 실제 작업에 연결하는 MCP(Model Context Protocol) 표준화 | ⭐⭐⭐ |
| Tip | LiteLLM으로 100개 이상 LLM을 통일된 인터페이스로 관리 | ⭐⭐ |
| Trend | 각국 정부의 자국 LLM 정책 자금 투자 확대 | ⭐⭐⭐ |
| Insight | RAG와 LLM Wiki의 편의성이 인간의 사고 과정을 대체할 위험성 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. LLM의 실제 활용을 위한 도구 연결 표준화

**핵심:** LLM은 기본적으로 텍스트 예측 엔진일 뿐이므로, 실제 작업(웹 검색, 캘린더 조회, 예약, 이메일 발송)을 수행하려면 외부 시스템과의 연결이 필수다. 이를 위해 기본 Tool Use → Function Calling → Model Context Protocol(MCP)로 진화했으며, 현재 주요 AI 기업들이 MCP를 채택하고 있다.

**공통 의견:** 단순히 LLM의 성능 개선을 넘어, 사용자 관점에서 "요청하면 자동으로 실행된다"는 경험을 만들기 위해서는 모델과 소프트웨어 간의 안전한 상호작용 프로토콜이 필수적이다. 이는 개별 기업의 독자적 구현에서 업계 표준으로 수렴하는 중이다.

**실무 적용:**

- MCP 기반 에이전트 구축 시 도구 정의(tool schema)를 명확히 하고, 모델이 요청한 도구 실행 결과를 정확히 피드백하는 루프 설계
- Sentry의 Seer Agent 사례처럼 기존 시스템의 풍부한 컨텍스트(로그, 트레이스, 프로필)를 LLM에 제공하여 정확도 향상
- 도구 실행 전 안전성 검증 계층 추가(사용자 승인, 권한 확인 등)

### 2. 다중 LLM 통합 관리의 실무 전략

**핵심:** LiteLLM과 같은 통합 라이브러리를 통해 OpenAI, Claude, Gemini, 로컬 모델 등 100개 이상의 LLM을 단일 인터페이스로 호출할 수 있다. 이는 각 모델별 API 호출 방식의 차이를 추상화하여 개발 생산성을 향상시킨다.

**공통 의견:** LLM 기술이 빠르게 진화하면서 최적의 모델도 계속 변한다. 특정 모델에 종속되지 않고 유연하게 모델을 교체할 수 있는 아키텍처가 장기적 경쟁력이 된다.

**실무 적용:**

- 프로젝트 초기 단계에서 LiteLLM 같은 추상화 계층 도입으로 향후 모델 변경 비용 최소화
- 비용 최적화를 위해 작은 작업은 경량 모델, 복잡한 추론은 고성능 모델로 자동 라우팅하는 로직 구현
- 각 모델의 응답 시간, 비용, 정확도를 모니터링하여 주기적으로 최적 모델 조합 재평가

### 3. RAG와 LLM Wiki의 편의성 역설

**핵심:** RAG(Retrieval-Augmented Generation)와 LLM Wiki는 기술적으로 우수하지만, 답변이 너무 빠르고 완전해지면서 사용자가 의도적으로 생각하고 탐색하는 과정을 건너뛰게 된다. 이는 단기적으로는 효율성 향상이지만, 장기적으로는 인간의 사고 능력 약화로 이어질 수 있다.

**공통 의견:** 기술의 성능만 평가할 것이 아니라, 그것이 인간의 인지 과정에 미치는 구조적 영향을 함께 고려해야 한다. 편의성과 자율적 사고 능력 사이의 균형이 중요하다.

**실무 적용:**

- RAG 시스템 설계 시 단순히 "최고의 답변"을 제공하기보다, 사용자가 추가 탐색을 유도하는 관련 질문이나 대안 제시 포함
- 조직 내 LLM Wiki 도입 시 직원들이 여전히 기초 개념을 학습하고 비판적 사고를 유지할 수 있도록 가이드라인 수립
- 완전 자동화보다는 "인간-AI 협업" 모드를 기본값으로 설정하여 의사결정 과정에 인간의 개입 지점 확보

### 4. 정부 정책 자금과 국가 AI 경쟁의 재정의

**핵심:** 한국을 포함한 각국 정부가 자국 LLM에 정책 자금을 투자하는 것은 단순한 기술 경쟁이 아니라, 칩(반도체) + 모델(LLM) + 데이터를 국내 기업으로 통합하려는 전략적 선택이다. 이는 AI 기술 주권과 경제 생태계 구축의 문제다.

**공통 의견:** AI 경쟁에서 "이기는 것"보다 중요한 것은 "지지 않는 것"이다. 즉, 특정 국가나 기업에 기술 종속되지 않으면서 자체 역량을 갖추는 것이 국가 차원의 전략이다.

**실무 적용:**

- 기업 입장에서는 정부 지원 프로그램(R&D 자금, 데이터 제공 등)을 적극 활용하되, 글로벌 표준(MCP 등)과의 호환성 유지
- 장기 로드맵에서 특정 해외 LLM에 대한 의존도를 단계적으로 낮추고, 국내 모델 또는 오픈소스 모델 활용 비중 확대
- 데이터 수집 및 학습 과정에서 국내 규제(개인정보보호법 등)를 선제적으로 충족하는 체계 구축

---

## 🛠️ 지금 당장 해볼 것

- [ ] **LiteLLM 설치 및 기본 테스트** — `pip install litellm` 후 공식 문서(https://docs.litellm.ai/)의 Quick Start 예제로 OpenAI와 Claude 모델을 동일한 코드로 호출해보기

- [ ] **MCP 프로토콜 이해하기** — Anthropic의 공식 MCP 저장소(https://github.com/anthropics/mcp) 방문하여 README와 예제 코드 검토, 간단한 tool 정의 작성해보기

- [ ] **RAG 시스템에 사용자 피드백 루프 추가** — 현재 운영 중인 RAG 시스템에서 "이 답변이 도움이 되었나요?" 같은 간단한 평가 버튼 추가하고, 부정 피드백 시 대체 답변이나 관련 질문 제시하는 로직 구현

- [ ] **로컬 LLM 테스트 환경 구축** — Ollama(https://ollama.ai/) 설치 후 Mistral 또는 Llama 2 같은 경량 모델 다운로드하여 로컬에서 실행, 클라우드 모델과의 응답 시간 및 비용 비교

---

## 🔗 참고 자료

- [Connecting LLMs to the Real World: Tool Use, Function Calling, and MCP](https://blog.bytebytego.com/p/connecting-llms-to-the-real-world)
- [RAG, LLM Wiki, and the Displacement of Human Thinking:
 A Structural Critique Beyond Capability](https://medium.com/@bulanramai2558/rag-llm-wiki-and-the-displacement-of-human-thinking-a-structural-critique-beyond-capability-6a1be03e95c9?source=rss------artificial_intelligence-5)
- [각국 정부가 자국 LLM에 정책 자금을 붓는 이유](https://blog.naver.com/littleorigins/224275282625)
- [LLM 비교 분석, 어떤 모델이 진짜 똑똑할까?](https://blog.naver.com/theloisbattery/224273358158)
- [인공지능교육, 머신러닝부터 LLM까지 한 번에 배우는 법](https://blog.naver.com/sbsacaart/224274681876)
- [LiteLLM: 100개 이상 LLM 통합, AI 개발의 새로운 표준!](https://blog.naver.com/info_honey/224275295026)

