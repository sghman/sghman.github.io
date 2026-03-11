---
title: "[Tech] 2026-03-11 기술 동향: claude"
author: gyuhwan
date: 2026-03-11 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, claude]
description: "핵심:** 대화가 길어질수록 Claude의 초기 지시사항이 압축되거나 손실되어 코드 스타일, 네이밍 컨벤션, 주석 언어가 일관성 없이 변한다. 이는 버그가 아니라 LLM의 컨텍스트 윈도우 특성이다."
permalink: /posts/2026-03-11-claude-digest/
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Claude Code /btw 기능 (사이드 질문 병렬 처리) 실험적 롤아웃 중 | ⭐⭐⭐ |
| Tip | CLAUDE.md로 장기 프로젝트 컨텍스트 일관성 유지 | ⭐⭐⭐ |
| Trend | AI 생성 코드 보안 취약점 — 자동 검증 필수 | ⭐⭐⭐ |
| Tip | 멀티 에이전트 아키텍처로 순차 작업 단축 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. Claude Code 장기 프로젝트에서 컨텍스트 일관성 유지하기

**핵심:** 대화가 길어질수록 Claude의 초기 지시사항이 압축되거나 손실되어 코드 스타일, 네이밍 컨벤션, 주석 언어가 일관성 없이 변한다. 이는 버그가 아니라 LLM의 컨텍스트 윈도우 특성이다.

**공통 의견:** 여러 개발자가 보고한 현상 — 처음엔 camelCase를 따르다가 나중엔 snake_case로 바뀌거나, 영어 주석만 하다가 갑자기 일본어 주석이 섞인다. 이는 오래된 지시사항이 컨텍스트에서 밀려나기 때문이다.

**실무 적용:**

- 프로젝트 루트에 `CLAUDE.md` 파일 생성 — Claude Code는 매 세션 시작과 `/clear` 후에 자동으로 이 파일을 읽고 재주입한다
- "Never"와 "Always"로 절대 규칙 명시 (예: "Never use `any` as TypeScript type", "Always add JSDoc to exported functions") — 소프트한 표현("prefer", "try to")은 컨텍스트 압박 상황에서 무시된다
- CLAUDE.md는 200줄 이하로 유지 — 지시사항 파일 자체가 너무 길면 같은 문제를 야기한다
- `/clear` 명령어를 기능 완성, 테스트 통과, 대규모 리팩토링 후 등 자연스러운 마일스톤에서 실행해 컨텍스트 누적 방지

### 2. Claude Code /btw 기능 — 메인 작업 중단 없이 사이드 질문 처리

**핵심:** 메시지 앞에 "btw"를 붙이면 메인 코딩 작업을 중단하지 않고 별도 스레드에서 사이드 질문을 병렬 처리한다. 현재 실험적 기능으로 일부 사용자에게만 작동 중이다.

**공통 의견:** 코드에 힌트가 있었지만 작동하지 않았고, 이후 실제 작동 보고가 나왔다. 공식 체인지로그나 문서화는 아직 없으며, 점진적 롤아웃 상태로 보인다.

**실무 적용:**

- 메인 작업 중 변수 네이밍 규칙, API 문서, 라이브러리 사용법 등 빠른 질문이 필요할 때 "btw, [질문]" 형식으로 입력
- 기존 방법의 문제점 해결: 지금 물어보면 컨텍스트 스위칭 비용이 크고, 나중에 물어보면 맥락을 잃는다 → /btw는 둘 다 해결
- 작동하지 않으면 새 터미널에서 별도 Claude Code 인스턴스 실행하되, 프로젝트 컨텍스트를 명시적으로 제공해야 한다

### 3. AI 생성 코드의 보안 취약점 — 자동 검증 필수

**핵심:** AI 생성 코드에는 보안 결함이 포함될 수 있다. Tea(여성 안전 앱)는 72,000개 이미지와 13,000개 정부 ID 사진을 노출했는데, 해킹이 아니라 기본 Firebase 설정을 변경하지 않아서였다.

**공통 의견:** Lovable, Cursor, Replit, Claude Code 등 AI 도구는 "작동하는 코드"를 최적화하지 "안전한 코드"를 최적화하지 않는다. 실제 사례로 대규모 API 키 노출과 데이터베이스 접근 제어 부재 사건들이 보고되었다.

**실무 적용:**

- 로그인하지 않은 사용자가 데이터베이스에 접근할 수 있는지 테스트 — 기본 Supabase/Firebase 설정은 공개 상태
- 배포 전 체크리스트 실행: (1) 환경 변수에 API 키 저장 여부, (2) 데이터베이스 접근 제어 설정, (3) 입력 검증 구현, (4) SQL 주입 방지, (5) 민감 정보 로깅 확인
- AI 생성 코드를 완전히 이해하지 못한 상태로 배포하지 않기

### 4. 멀티 에이전트 아키텍처로 병렬 처리 성능 극대화

**핵심:** 단일 Claude 인스턴스로 대규모 코드베이스 분석, 테스트 작성, 문서화를 동시에 하면 컨텍스트 고갈과 순차 병목이 발생한다. 멀티 에이전트 아키텍처는 오케스트레이터가 작업을 분할하고 워커가 병렬 실행한다.

**공통 의견:** Claude Code의 Agent 도구로 프로그래밍 방식 서브에이전트 생성 가능. 각 에이전트는 독립적 컨텍스트 윈도우, 도구, 모델을 가진다. 비용 효율을 위해 오케스트레이터는 Opus, 워커는 Haiku 사용.

**실무 적용:**

- 여러 파일 이해가 필요한 리팩토링: Opus 오케스트레이터 1개 + Haiku 워커 병렬 실행 (각 파일 요약)
- `run_in_background` 파라미터로 느린 작업을 논블로킹 방식 실행 — 오케스트레이터가 결과를 나중에 확인
- 모델 선택 기준: 단순 읽기/요약은 Haiku, 복잡한 분석/생성은 Sonnet, 최종 종합은 Opus

---

## 🛠️ 지금 당장 해볼 것

- [ ] 현재 프로젝트 루트에 `CLAUDE.md` 파일 생성 후 프로젝트 규칙 작성 — 템플릿: `# Project Rules\n## Language\n- All code comments: English only\n- All variable names: camelCase\n## Never Do\n- Never add console.log to production code` (참고: https://dev.to/myougatheaxo/claude-code-context-management-keep-ai-output-consistent-on-long-projects-4d5h)

- [ ] Claude Code에서 "btw, [빠른 질문]" 형식으로 메시지 입력해 /btw 기능 작동 여부 테스트 — 작동하지 않으면 아직 롤아웃 대기 중 (공식 지원 예정)

- [ ] 배포된 앱의 데이터베이스 접근 제어 확인 — Supabase 콘솔에서 RLS(Row Level Security) 정책 활성화 여부 검증

- [ ] `.claude/skills/code-review/SKILL.md` 파일 생성 후 5축 코드 리뷰 스킬 정의 — 이후 `/code-review src/` 명령어로 자동 실행 (참고: https://dev.to/myougatheaxo/how-to-build-claude-code-custom-skills-for-automated-code-review-85p)

---

## 🔗 참고 자료

- [Claude Code Context Management: Keep AI Output Consistent on Long Projects](https://dev.to/myougatheaxo/claude-code-context-management-keep-ai-output-consistent-on-long-projects-4d5h)
- [Claude Code /btw 기능: 메인 작업 중단 없이 사이드 질문하기](https://dev.to/_46ea277e677b888e0cd13/claude-code-btw-gineung-mein-jageob-jungdan-eobsi-saideu-jilmunhagi-1bpj)
- [20 NixOS modules, 11 systemd services: the infrastructure running an autonomous AI company](https://dev.to/0coceo/20-nixos-modules-11-systemd-services-the-infrastructure-running-an-autonomous-ai-company-117l)
- [Your Vibe-Coded App Might Be Wide Open (Here's How to Check)](https://dev.to/noa-agent/your-vibe-coded-app-might-be-wide-open-heres-how-to-check-4ihe)
- [Building Parallel AI Pipelines with Claude Code Multi-Agent Architecture](https://dev.to/myougatheaxo/building-parallel-ai-pipelines-with-claude-code-multi-agent-architecture-di7)
- [How to Build Claude Code Custom Skills for Automated Code Review](https://dev.to/myougatheaxo/how-to-build-claude-code-custom-skills-for-automated-code-review-85p)
- [클로드 코드 공부](https://blog.naver.com/sone_han/224212312613)
- [The ONLY Claude Code Tutorial You'll Ever Need in 2026 - YouTube](https://www.youtube.com/watch?v=LlFgLsffbBs)
- [1000+ hours of Learning Claude Code in 10 Minutes (2026 Tutorial)](https://www.youtube.com/watch?v=3aKVArutiIU)
- [Claude Code Tutorial for Beginners (2026) - YouTube](https://www.youtube.com/playlist?list=PL4HikwTaYE0ETMaJqnNvm_2I3NEbexMDZ)
- [Claude Code Learning Path: a practical guide to getting started](https://medium.com/@dan.avila7/claude-code-learning-path-a-practical-guide-to-getting-started-fcc601550476)
- [CLAUDE CODE FULL COURSE 4 HOURS: Build & Sell (2026)](https://www.youtube.com/watch?v=QoQBzR1NIqI)

