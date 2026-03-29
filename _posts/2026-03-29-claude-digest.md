---
title: "[Tech] 2026-03-29 기술 동향: claude"
author: gyuhwan
date: 2026-03-29 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, claude]
description: "핵심:** Claude가 사용자의 화면을 보고, 마우스를 클릭하고, 키보드를 입력하는 기능이 출시되었습니다. 이는 단순한 텍스트 생성을 넘어 실제 애플리케이션 조작까지 가능하게 합니다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Claude Computer Use 출시 — AI가 직접 컴퓨터 화면 조작 | ⭐⭐⭐ |
| Tip | Claude Code를 Slack/Telegram과 연결하는 Channels 기능 | ⭐⭐ |
| Trend | LinkedIn이 AI 챗봇의 주요 학습 데이터 소스로 부상 | ⭐ |
| Tool | Bug Ticket Agent — Sentry 에러를 자동으로 Notion에 정리 | ⭐⭐⭐ |

---

## 💡 Deep Dive

### 1. Claude Computer Use: AI가 직접 컴퓨터를 조작하는 시대

**핵심:** Claude가 사용자의 화면을 보고, 마우스를 클릭하고, 키보드를 입력하는 기능이 출시되었습니다. 이는 단순한 텍스트 생성을 넘어 실제 애플리케이션 조작까지 가능하게 합니다.

**공통 의견:** 여러 튜토리얼과 가이드에서 Claude Code와 Computer Use를 함께 언급하며, 이 두 기능이 2026년 AI 개발의 핵심 변화라고 평가합니다. 특히 반복적인 작업 자동화와 크로스 플랫폼 통합이 가능해졌다는 점이 강조됩니다.

**실무 적용:**

- 데이터 입력, 양식 작성, 스크린샷 캡처 등 수동 작업을 Claude에게 위임하여 생산성 향상
- 기존 레거시 시스템과의 통합이 필요한 업무 자동화에 활용 (API 없는 구식 소프트웨어도 조작 가능)
- 테스트 자동화: UI 테스트를 Claude가 직접 수행하도록 구성

### 2. Claude Projects와 맞춤형 작업 공간 구축

**핵심:** Claude의 Projects 기능을 통해 반복되는 지침(프롬프트, 컨텍스트, 파일)을 저장하고 재사용할 수 있습니다. 매번 같은 설정을 입력할 필요가 없어집니다.

**공통 의견:** 여러 사용자 가이드에서 Projects를 "개인 맞춤형 AI 어시스턴트 만들기"의 핵심으로 소개합니다. 직장인, 학생, 크리에이터 모두에게 유용한 기능으로 평가됩니다.

**실무 적용:**

- 직무별 프로젝트 생성: 마케팅, 개발, 기획 등 역할별로 별도 프로젝트 구성
- 회사 가이드라인, 브랜드 톤, 템플릿을 프로젝트에 저장하여 팀 전체가 일관성 있게 사용
- 클라이언트별 프로젝트 분리로 컨텍스트 혼동 방지

### 3. AI 에러 트리아주 자동화: Bug Ticket Agent 패턴

**핵심:** Sentry 에러를 Claude AI가 자동으로 분석하고, 우선순위를 매기고, Notion에 정리된 티켓으로 변환합니다. 2AM 수동 작업이 완전히 자동화됩니다.

**공통 의견:** 이 사례는 Claude의 에이전트 기능(MCP 활용)이 실제 프로덕션 워크플로우에 얼마나 효과적인지 보여줍니다. 중복 제거, 우선순위 자동 할당, 수정 제안까지 한 번에 처리됩니다.

**실무 적용:**

- 모니터링 도구(Sentry, DataDog, New Relic)의 알림을 Claude 에이전트로 자동 분류
- P0/P1 에러는 즉시 PagerDuty/Slack으로 에스컬레이션
- 스택 트레이스 분석 → 영향받은 사용자 수 파악 → 수정 제안까지 자동 생성

### 4. Claude.md 포맷 규칙: 기계가 읽기 쉬운 문서 작성

**핵심:** Claude와 협업할 때 사용할 CLAUDE.md 파일의 7가지 포맷 규칙이 정리되었습니다. 구조화된 문서가 AI의 이해도와 응답 품질을 크게 향상시킵니다.

**공통 의견:** 단순히 프롬프트를 잘 쓰는 것을 넘어, 문서 자체를 "AI-friendly"하게 구성하는 것이 중요하다는 인식이 확산되고 있습니다.

**실무 적용:**

- 프로젝트 루트에 CLAUDE.md 작성: 아키텍처, 코딩 컨벤션, 금지 사항 명시
- 마크다운 헤딩, 코드 블록, 리스트를 일관되게 사용하여 파싱 오류 최소화
- 예제 코드와 반례를 함께 제시하여 Claude의 이해도 향상

---

## 🛠️ 지금 당장 해볼 것

- [ ] Claude Computer Use 공식 문서 확인 — https://www.anthropic.com/news/computer-use 에서 베타 신청 및 데모 영상 시청 (5분)

- [ ] Claude Code 설치 및 첫 프로젝트 생성 — https://code.claude.com 접속 후 "New Project" 클릭, 간단한 Python 스크립트 작성해보기 (10분)

- [ ] 자신의 업무에 맞는 Claude Project 템플릿 작성 — 현재 반복하는 작업 3가지를 선택하고, 각각에 대한 프롬프트 + 파일 구조를 CLAUDE.md 형식으로 정리 (15분)

- [ ] Bug Ticket Agent 데모 실행 — https://github.com/tunabearfish/bug-ticket-agent 클론 후 `npm install && npm run demo -- --scenario p0` 실행하여 Notion 연동 동작 확인 (10분)

- [ ] Claude Channels로 Slack 연결 테스트 — https://code.claude.com/docs/en/channels 문서 참고하여 Slack 워크스페이스에 Claude 봇 추가 및 첫 메시지 전송 (5분)

---

## 🔗 참고 자료

- [# Bug Ticket Agent — AI-Powered Sentry Triage into Notion](https://dev.to/tunabearfish/-bug-ticket-agent-ai-powered-sentry-triage-into-notion-1moh)
- [LinkedIn Emerges as a Top Source for AI Chatbot Knowledge](https://medium.com/the-linkedin-creators-playbook/linkedin-emerges-as-a-top-source-for-ai-chatbot-knowledge-fd045df83eb1?source=rss------artificial_intelligence-5)
- [CLAUDE.md 모범 사례: 기계용 7가지 서식 규칙](https://blog.naver.com/artdio1008/224233165641)
- [Claude Code와 슬랙 연결하기](https://blog.naver.com/majinman/224233151774)
- [Claude Computer Use 출시 클로드 만만세](https://blog.naver.com/after_work_lab/224233160767)
- [AI 툴인 클로드(Claude), 특히 나만의 맞춤형 작업 공간을...](https://blog.naver.com/bkyang2011/224233192925)
- [클로드(Claude) AI 완벽 가이드 — Anthropic의 AI, 무엇이...](https://blog.naver.com/isan2264/224233036561)
- [FULL Claude Tutorial for Beginners in 2026! (Become a PRO!)](https://www.youtube.com/watch?v=rRrBbyv3ChM)
- [Claude Code Tutorial 2026: Beginner to Pro Guide](https://www.nocode.mba/articles/claude-code-tutorial)
- [Claude Code Complete Guide 2026: From Zero to Hero](https://claude-world.com/articles/claude-code-complete-guide-2026/)
- [Claude Code Learning Path: a practical guide to getting started](https://medium.com/@dan.avila7/claude-code-learning-path-a-practical-guide-to-getting-started-fcc601550476)
- [Claude Code Tutorial for Beginners - Complete 2026 Guide to AI ...](https://codewithmukesh.com/blog/claude-code-for-beginners/)

