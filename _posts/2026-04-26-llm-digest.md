---
title: "[Tech] 2026-04-26 기술 동향: LLM"
author: gyuhwan
date: 2026-04-26 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** Anthropic의 Project Deal 실험에서 AI 에이전트가 자율적으로 186건의 거래를 체결했으며(총 거래액 $4,000 이상), 참여자의 46%가 유사 서비스에 대한 구매 의향을 보였다. 이는 단순한 데모가 아닌 실제 제품-시장 적합성(PMF) 신호다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Anthropic의 Project Deal: AI 에이전트가 실제 거래 186건 체결 (2026년 4월 26일) | ⭐⭐⭐ |
| Trend | 오픈 인터넷에서 에이전트 간 거래를 위한 온체인 인프라 구축 중 | ⭐⭐⭐ |
| Security | 에이전트 세션 중 tool_result 블록을 통한 프롬프트 인젝션 공격 | ⭐⭐⭐ |
| Tool | TokTranscript: TikTok 영상을 스크립트로 변환하는 LLM 활용 도구 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. AI 에이전트의 상용화 시대 도래 — 신뢰 인프라가 핵심

**핵심:** Anthropic의 Project Deal 실험에서 AI 에이전트가 자율적으로 186건의 거래를 체결했으며(총 거래액 $4,000 이상), 참여자의 46%가 유사 서비스에 대한 구매 의향을 보였다. 이는 단순한 데모가 아닌 실제 제품-시장 적합성(PMF) 신호다.

**공통 의견:** 폐쇄된 신뢰 환경(Anthropic 사무실)에서는 에이전트 간 거래가 완벽하게 작동했지만, 오픈 인터넷으로 확장되려면 평판 시스템, 에스크로우, 신원 검증, 책임 추적성이 필수다. 동시에 ERC-8004(온체인 신원, 129K+ 에이전트 등록), ERC-8183(분쟁 해결), x402 프로토콜(기계 간 결제, 165M+ 거래, $50M+ 거래량) 등 필요한 인프라가 이미 구축되고 있다.

**실무 적용:**

- 에이전트 기반 서비스 개발 시 신뢰 메커니즘(평판 점수, 거래 이력)을 초기 설계에 포함
- Base 네트워크 같은 에이전트 친화적 블록체인 스택 모니터링 (현재 트래픽의 20%가 에이전트 간 거래)
- AgentLux 같은 검증된 온체인 신원 시스템 활용으로 초기 신뢰도 확보

### 2. 에이전트 세션 중 프롬프트 인젝션 — 새로운 공격 벡터

**핵심:** 기존 프롬프트 인젝션 방어는 사용자 입력 단계에만 집중했지만, 에이전트가 웹 페이지, API, 파일을 읽을 때 반환되는 tool_result 블록 내 숨겨진 지시문이 LLM을 직접 제어할 수 있다. 단일 턴 챗봇은 1개의 공격 표면을 가지지만, 에이전트는 세션당 N개의 공격 표면을 노출한다.

**공통 의견:** 다단계 에이전트 워크플로우에서 2단계의 손상된 tool_result가 이후 모든 단계를 조용히 리다이렉트할 수 있으며, 로그에는 아무 이상이 나타나지 않는다. 이는 기존 보안 감시 도구로 탐지 불가능한 블라인드 스팟이다.

**실무 적용:**

- tool_result 콘텐츠에 대한 별도의 샌드박싱 및 검증 레이어 구현 (사용자 입력과 동일 수준)
- 에이전트 로그에 각 tool_result의 출처와 내용 해시 기록하여 사후 감시 가능하게 설계
- 웹 스크래핑 에이전트의 경우 신뢰할 수 있는 도메인 화이트리스트 우선 적용

### 3. LLM 기반 개발자 도구의 실용화 — 반복 작업 자동화

**핵심:** TokTranscript는 TikTok 영상을 자동 전사하고 구조화된 분석(훅, 페이싱, 콜백)을 제공하는 도구로, 창작자가 수십 개 클립을 비교 분석할 때의 기계적 마찰을 제거한다. 이는 LLM이 단순 텍스트 생성을 넘어 실제 워크플로우 최적화에 기여하는 사례다.

**공통 의견:** AI 에이전트와 LLM 도구의 가치는 새로운 기능 추가보다 기존 작업의 반복 비용 제거에 있다. 전사, 구조화, 리믹싱 같은 기계적 작업을 자동화하면 인간은 창의적 판단(맛, 전략)에 집중할 수 있다.

**실무 적용:**

- 자신의 업무에서 "반복적으로 복사-붙여넣기하는 부분" 찾아 LLM 자동화 기회 발굴
- 단순 텍스트 생성보다 "구조화된 출력 + 선택지 제시" 형태의 도구 설계 (사용자 판단 비용 최소화)
- 공개 API(TikTok, YouTube 등)를 활용한 소규모 LLM 도구 개발로 틈새 시장 공략

---

## 🛠️ 지금 당장 해볼 것

- [ ] **Anthropic Project Deal 기술 스택 분석** — `site:github.com anthropic agents commerce` 검색하여 오픈소스 에이전트 프레임워크 확인 및 로컬 테스트 (5분)

- [ ] **tool_result 프롬프트 인젝션 테스트** — Claude API 또는 로컬 LLM에서 간단한 에이전트 스크립트 작성 후 의도적으로 악성 tool_result 주입해보기: https://github.com/anthropics/anthropic-sdk-python (10분)

- [ ] **TokTranscript 또는 유사 도구 직접 사용** — https://toktranscript.com/ 접속하여 자신이 자주 분석하는 TikTok 영상 1개 전사 및 구조화 분석 기능 테스트 (5분)

- [ ] **에이전트 신뢰 인프라 모니터링** — `ERC-8004 agent identity` 또는 `AgentLux on-chain reputation` 검색하여 현재 구축 상황 파악 및 테스트넷 접근성 확인 (10분)

---

## 🔗 참고 자료

- [Anthropic Just Proved Agents Can Do Commerce. Here's What They Didn't Build.](https://dev.to/aaron_schnieder_4563d5d33/anthropic-just-proved-agents-can-do-commerce-heres-what-they-didnt-build-5209)
- [TokTranscript: turn any TikTok into a transcript, then study why it worked](https://dev.to/shixin_zhang_060518ea1143/toktranscript-turn-any-tiktok-into-a-transcript-then-study-why-it-worked-4hp5)
- [Your AI Agent Is Reading Poisoned Web Pages (And You Don't Know It)](https://dev.to/coridev/your-ai-agent-is-reading-poisoned-web-pages-and-you-dont-know-it-3ea8)
- [오케스트레이션까지 고위 GenAI/LLM 엔지니어링 역할을 해킹...](https://blog.naver.com/artdio1008/224265439035)
- [AI에게 코딩은 "정교한 언어 번역"과 같다](https://blog.naver.com/juvefc/224265451277)

