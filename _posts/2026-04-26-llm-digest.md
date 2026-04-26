---
title: "[Tech] 2026-04-26 기술 동향: LLM"
author: gyuhwan
date: 2026-04-26 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** Anthropic이 Project Deal을 통해 AI 에이전트가 실제 거래를 자율적으로 체결할 수 있음을 증명했다. 69개 에이전트가 $4,000 규모의 186건 거래를 성사시켰고, 46%의 참여자가 유사 서비스 구매 의향을 표현했다. 하지만 폐쇄된 조직 내부에서만 작동했다는 한계가 있다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Anthropic의 Project Deal: AI 에이전트가 실제 거래 186건 체결 | ⭐⭐⭐ |
| Trend | 에이전트 간 상거래 인프라 (ERC-8004, x402 프로토콜) 실제 운영 중 | ⭐⭐⭐ |
| Security | Tool result 기반 프롬프트 인젝션 공격 — 기존 방어 우회 | ⭐⭐⭐ |
| Tip | TikTok 트랜스크립트 도구로 숏폼 콘텐츠 분석 자동화 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. AI 에이전트 상거래의 현실화 — 신뢰 인프라가 핵심

**핵심:** Anthropic이 Project Deal을 통해 AI 에이전트가 실제 거래를 자율적으로 체결할 수 있음을 증명했다. 69개 에이전트가 $4,000 규모의 186건 거래를 성사시켰고, 46%의 참여자가 유사 서비스 구매 의향을 표현했다. 하지만 폐쇄된 조직 내부에서만 작동했다는 한계가 있다.

**공통 의견:** 에이전트 경제의 확장성은 신뢰 인프라에 달려 있다. 개방형 인터넷에서 에이전트 간 거래가 성립하려면 평판 시스템, 에스크로우, 신원 검증, 책임 추적이 필수다. 현재 ERC-8004(온체인 신원), ERC-8183(분쟁 해결 에스크로우), x402 프로토콜(기계 간 결제)이 이미 운영 중이며, Base 네트워크의 에이전트 간 트래픽이 증가하고 있다.

**실무 적용:**

- 에이전트 기반 서비스 구축 시 온체인 신원 검증 레이어 통합 검토 (ERC-8004 표준 참고)
- 다중 에이전트 워크플로우에서 거래 단계별 에스크로우 메커니즘 설계
- 에이전트 평판 점수를 트랜잭션 결과 데이터로부터 자동 계산하는 시스템 구축

### 2. 에이전트 세션 중 프롬프트 인젝션 — 새로운 공격 벡터

**핵심:** 기존 프롬프트 인젝션 방어는 사용자 입력 단계에만 집중했다. 하지만 에이전트가 웹 페이지, 파일, API 응답을 읽을 때 반환되는 `tool_result` 블록에 숨겨진 악성 지시문이 삽입될 수 있다. 이는 사용자 입력 검증을 완전히 우회한다.

**공통 의견:** 멀티턴 에이전트 워크플로우에서 각 도구 호출마다 새로운 공격 표면이 생긴다. 초기 단계의 손상된 결과가 이후 모든 단계를 조용히 리다이렉트할 수 있으며, 로그에는 아무 이상이 나타나지 않는다. 단일 턴 챗봇의 1개 공격 표면에서 N개 공격 표면으로 확대된다.

**실무 적용:**

- 모든 `tool_result` 콘텐츠를 신뢰할 수 없는 입력으로 취급하고 구조화된 파싱 적용
- 에이전트 세션 로그에 각 도구 호출의 원본 응답과 처리 결과를 기록하여 감시
- 에이전트가 접근하는 외부 소스(웹사이트, API)의 신뢰도 점수 시스템 도입

### 3. 숏폼 콘텐츠 분석 자동화 — 창작자의 기계적 작업 제거

**핵심:** TokTranscript는 TikTok URL을 붙여넣으면 타임스탬프가 있는 전체 트랜스크립트를 생성하고, 프로 버전에서는 훅, 페이싱, 콜백 구조를 분석해준다. 창작자가 반복적으로 영상을 재생하고 스크롤하며 라인을 복사하는 작업을 자동화한다.

**공통 의견:** 숏폼 콘텐츠 분석의 병목은 영상 시청이 아니라 구조 파악이다. 트랜스크립트 + AI 분석 조합으로 경쟁 분석, 포맷 역공학, 성공 패턴 추출이 가능해진다. 공개 피드(Plaza)를 통해 니치별 패턴 브라우징도 지원한다.

**실무 적용:**

- 콘텐츠 마케팅 팀: 경쟁사 영상 5~10개를 TokTranscript로 분석하여 구조 비교표 작성
- 창작자: 자신의 고성과 영상 3개를 분석하여 개인 포맷 템플릿 추출
- 에이전시: Script Remix로 검증된 포맷 구조를 유지하면서 클라이언트별 주제로 재작성

---

## 🛠️ 지금 당장 해볼 것

- [ ] **에이전트 신원 검증 표준 학습** — `site:github.com ERC-8004` 검색하여 온체인 에이전트 신원 레지스트리 구현 사례 확인

- [ ] **Tool result 검증 로직 추가** — 현재 운영 중인 에이전트 코드에서 `tool_result` 처리 부분을 찾아 구조화된 파싱(JSON 스키마 검증) 추가

- [ ] **TokTranscript 직접 테스트** — https://toktranscript.com/ 접속하여 자신의 최근 고성과 TikTok 영상 1개 트랜스크립트 생성 후 Viral Breakdown 분석 결과 검토

- [ ] **에이전트 세션 로깅 감시 설정** — 사용 중인 에이전트 프레임워크(LangChain, AutoGen 등)의 콜백 핸들러에서 각 도구 호출 전후 원본 응답 기록 활성화

---

## 🔗 참고 자료

- [Anthropic Just Proved Agents Can Do Commerce. Here's What They Didn't Build.](https://dev.to/aaron_schnieder_4563d5d33/anthropic-just-proved-agents-can-do-commerce-heres-what-they-didnt-build-5209)
- [TokTranscript: turn any TikTok into a transcript, then study why it worked](https://dev.to/shixin_zhang_060518ea1143/toktranscript-turn-any-tiktok-into-a-transcript-then-study-why-it-worked-4hp5)
- [Your AI Agent Is Reading Poisoned Web Pages (And You Don't Know It)](https://dev.to/coridev/your-ai-agent-is-reading-poisoned-web-pages-and-you-dont-know-it-3ea8)
- [오케스트레이션까지 고위 GenAI/LLM 엔지니어링 역할을 해킹...](https://blog.naver.com/artdio1008/224265439035)
- [AI에게 코딩은 "정교한 언어 번역"과 같다](https://blog.naver.com/juvefc/224265451277)

