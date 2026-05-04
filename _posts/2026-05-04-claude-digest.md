---
title: "[Tech] 2026-05-04 기술 동향: claude"
author: gyuhwan
date: 2026-05-04 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, claude]
description: "핵심:** 현대 LLM은 자기지도학습(Self-Supervised Learning) + 지도학습(Supervised Learning) + 강화학습(Reinforcement Learning)의 세 단계로 구성되어 있다. Yann LeCun의 비유를 빌리면 \"케이크는 자기지도학습, 아이싱은 지도학습, 체리는 강화학습\"이다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Claude Code를 활용한 자동화 워크플로우 확산 | ⭐⭐⭐ |
| Tip | AI 대화 내용 체계적으로 관리하는 습관 | ⭐⭐⭐ |
| Trend | AI 에이전트를 위한 구조화된 데이터 레이어 필요성 대두 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. Claude의 학습 구조: 세 가지 레이어의 조합

**핵심:** 현대 LLM은 자기지도학습(Self-Supervised Learning) + 지도학습(Supervised Learning) + 강화학습(Reinforcement Learning)의 세 단계로 구성되어 있다. Yann LeCun의 비유를 빌리면 "케이크는 자기지도학습, 아이싱은 지도학습, 체리는 강화학습"이다.

**공통 의견:** 기존 ML 교과서의 "지도 vs 비지도" 이분법으로는 Claude 같은 현대 LLM을 설명할 수 없다. 자기지도학습이 전체 학습량의 대부분을 차지하며, 이것이 Claude의 강력한 기초를 만든다.

**실무 적용:**

- Claude와 상호작용할 때 초기 응답(자기지도학습 기반)과 미세 조정된 응답(강화학습 기반)의 차이를 인식하고 활용
- 복잡한 작업은 여러 턴의 대화로 나누어 강화학습 단계의 이점을 최대화

### 2. Claude Code를 활용한 실무 자동화의 실제 사례

**핵심:** Claude Code는 단순 코드 생성을 넘어 복잡한 상태 관리, 설계 검토, 반복적 개선을 자동으로 수행할 수 있다. FiveM 프로젝트의 의료 시스템 개발 사례에서 "우아한 상태 머신"을 3번 삭제하고 재설계한 과정은 AI와의 협업이 단순 구현을 넘어 설계 사고까지 영향을 미친다는 것을 보여준다.

**공통 의견:** Claude Code의 가치는 코드 작성 속도가 아니라 설계 결정 과정에서의 질문 제기와 대안 제시에 있다. 강의 스크립트 생성 사례에서도 개요만으로 완벽한 스크립트를 생성하는 능력이 확인되었다.

**실무 적용:**

- 프로젝트 초기 단계에서 Claude Code에 요구사항을 제시하고 설계 대안을 먼저 검토한 후 구현 시작
- 반복적인 리팩토링 작업을 Claude Code에 위임하여 개발 속도 가속화

### 3. AI 대화 관리의 체계화: 검색 불가능한 지식의 손실 방지

**핵심:** AI 플랫폼의 검색 기능은 본질적으로 제한적이다. 중요한 대화는 자동으로 저장되지 않으며, 시간이 지날수록 찾기 어려워진다. 체계적인 내보내기 습관이 없으면 고가치 대화 내용이 영구적으로 손실된다.

**공통 의견:** 모든 주요 AI 플랫폼(ChatGPT, Claude, Gemini)이 동일한 문제를 가지고 있다. 자동 생성된 제목은 검색 불가능하고, 즐겨찾기 기능도 부족하다.

**실무 적용:**

- 설계 결정, 아키텍처 검토, 코드 리뷰 등 실질적 가치가 있는 대화는 즉시 내보내기 (Chrome 확장 프로그램 활용)
- 내보낸 파일에 날짜와 프로젝트명을 명시하여 로컬 저장소에 체계적으로 관리

### 4. AI 에이전트 시대의 새로운 인프라: 구조화된 데이터 레이어

**핵심:** 현재 AI 에이전트(ChatGPT, Claude, Perplexity)는 제품 검색 같은 실무 작업을 수행할 수 없다. 이유는 제품 데이터가 AI 파싱에 최적화되지 않았기 때문이다. 50M+ 제품이 있어도 기계 가독성이 없으면 AI는 활용할 수 없다.

**공통 의견:** 향후 AI 에이전트 경제에서는 "Stripe for product data" 같은 구조화된 API 레이어가 필수 인프라가 될 것이다. 이는 단순 데이터 제공이 아니라 AI 발견 가능성(AI discoverability)을 보장하는 새로운 비즈니스 기회다.

**실무 적용:**

- 자신의 제품/서비스를 AI 에이전트가 쿼리할 수 있는 구조화된 형식으로 제공 준비
- 3.5M+ AI 에이전트 개발자들이 필요로 하는 데이터 표준화 방식 학습

---

## 🛠️ 지금 당장 해볼 것

- [ ] Claude 대화 내보내기 시작 — Chrome 확장 프로그램 설치: `XWX AI Chat Exporter` 검색 후 설치, 중요한 대화마다 Export 버튼 클릭

- [ ] Claude Code 활용 테스트 — Claude.ai에서 새 프로젝트 생성 후 "이 코드를 리팩토링하고 개선 방안을 제시해줘"라는 프롬프트로 설계 검토 기능 체험

- [ ] 자신의 제품 데이터 구조 검토 — 현재 제공하는 제품 정보가 CSV/JSON 형식으로 표준화되어 있는지 확인, 업계 표준 확인

---

## 🔗 참고 자료

- [How GPT and Claude Are Trained: The Three-Stage Recipe Behind Modern AI](https://irtezaasadrizvi.medium.com/how-gpt-and-claude-are-trained-the-three-stage-recipe-behind-modern-ai-9ae82233330c?source=rss------artificial_intelligence-5)
- [The Most Elegant State Machine I Ever Deleted](https://dev.to/tackk3/the-most-elegant-state-machine-i-ever-deleted-59ig)
- [The AI Habit I Wish I'd Started Sooner](https://dev.to/doremi/the-ai-habit-i-wish-id-started-sooner-2c7n)
- [The Missing API Layer for AI Agent Commerce](https://dev.to/buywhere/the-missing-api-layer-for-ai-agent-commerce-1k0d)
- [강의 개요만으로 완벽한 스크립트를 작성하는 Claude Code...](https://blog.naver.com/rotater/224259872029)

