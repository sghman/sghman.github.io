---
title: "[Tech] 2026-03-18 기술 동향: claude"
author: gyuhwan
date: 2026-03-18 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, claude]
description: "핵심:** Claude Code는 세션이 리셋될 때마다 이전 작업 내용을 잃어버린다. 2시간 이상의 긴 작업에서는 매번 같은 내용을 다시 설명해야 하는 비효율이 발생한다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Claude Code의 컨텍스트 손실 문제 해결 방법 | ⭐⭐⭐ |
| Tip | 권한 확인 프롬프트 비활성화로 작업 속도 향상 | ⭐⭐⭐ |
| Tip | MCP 서버를 통한 외부 서비스 연동 | ⭐⭐ |
| Trend | Claude를 활용한 빠른 프로토타이핑 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. Claude Code 컨텍스트 손실 문제와 해결책

**핵심:** Claude Code는 세션이 리셋될 때마다 이전 작업 내용을 잃어버린다. 2시간 이상의 긴 작업에서는 매번 같은 내용을 다시 설명해야 하는 비효율이 발생한다.

**공통 의견:** 파일 기반의 체크포인트 시스템이 가장 효과적이다. 대화 히스토리는 컨텍스트 압축 과정에서 사라지지만, 디스크의 파일은 매 세션마다 새로 읽혀진다.

**실무 적용:**

- 프로젝트 루트에 `tasks/current-task.md` 파일 생성하여 현재 목표, 완료된 단계, 주요 결정사항, 마지막 체크포인트 기록
- CLAUDE.md에 "세션 시작 시 tasks/current-task.md 읽기" 지시사항 추가
- 매 주요 작업 후 `last_checkpoint` 필드를 "다음 할 일"까지 명확히 업데이트
- 체크포인트 작성 시 "리팩토링 중" 같은 모호한 표현 대신 "auth 로직을 lib/auth.ts로 추출 완료. 다음: service.ts의 3개 호출 지점 업데이트" 같이 구체적으로 기록

### 2. Claude Code 권한 확인 프롬프트 최적화

**핵심:** 기본 설정에서 Claude Code는 모든 명령 실행과 파일 편집 전에 확인을 요청한다. 신뢰할 수 있는 작업에서는 이 과정이 상당한 시간 낭비가 된다.

**공통 의견:** 프로젝트의 성격과 위험도에 따라 설정을 구분해야 한다. 프로덕션 환경이나 민감한 코드베이스에서는 기본값 유지, 개인 프로젝트나 프로토타입에서는 자동 수락 모드 사용이 권장된다.

**실무 적용:**

- `.claude/settings.json`에 `"defaultMode": "acceptEdits"` 설정으로 자동 수락 활성화 (프로젝트별 적용)
- 전역 설정은 `~/.claude/settings.json`에 동일 설정 추가
- CLAUDE.md에 "명령 실행 전 확인 요청하지 말 것. 실행 후 결과 보고" 지시사항 추가 (더 유연한 방식)
- 프로덕션 환경에서는 `default` 모드 유지, 개발 환경에서만 `acceptEdits` 사용

### 3. MCP 서버를 통한 Claude Code 기능 확장

**핵심:** MCP(Model Context Protocol)는 Claude가 데이터베이스, API, 파일 시스템 등 외부 서비스에 접근할 수 있게 해주는 표준 인터페이스다. Claude Code는 기본 MCP 서버(파일시스템, HTTP 요청, 메모리)를 포함하고 있으며, 커스텀 서버 추가가 가능하다.

**공통 의견:** MCP 서버는 도메인별로 분리하고, 도구 이름과 설명을 명확하게 작성해야 Claude가 올바른 시점에 호출할 수 있다. 응답 속도가 세션 전체 속도에 영향을 미치므로 성능 최적화가 중요하다.

**실무 적용:**

- `.claude/settings.json`의 `mcpServers` 섹션에 커스텀 서버 등록 (Node.js 또는 Python 프로세스)
- 도구 이름을 `query` 대신 `run_read_only_sql_query` 같이 구체적으로 작성
- 각 MCP 서버는 하나의 도메인만 담당하도록 설계 (데이터베이스용, API용 등 분리)
- 에러 메시지에 원인과 해결 방법을 포함하여 Claude가 적응할 수 있도록 작성

### 4. Claude를 활용한 빠른 프로토타이핑

**핵심:** Claude Pro 구독과 명확한 기획안만으로 웹 앱을 빠르게 만들 수 있다. 운세 앱 같은 간단한 프로젝트는 기획 문서 하나로 완성 가능하다.

**공통 의견:** 성공의 핵심은 기획안의 품질이다. "앱 만들어줘"라는 모호한 요청보다 요구사항을 명확히 정리한 문서가 결과물의 질을 크게 좌우한다.

**실무 적용:**

- 프로토타입 개발 전에 기획안 문서 작성 (목표, 주요 기능, UI 흐름 등)
- Netlify 같은 무료 호스팅 서비스와 연동하여 즉시 배포 가능한 환경 구성
- Claude에 기획안을 먼저 제시한 후 단계별로 개발 진행

---

## 🛠️ 지금 당장 해볼 것

- [ ] 현재 진행 중인 Claude Code 프로젝트에서 `tasks/current-task.md` 파일 생성하고 현재 상태를 구체적으로 기록 — 다음 세션에서 컨텍스트 손실 방지 확인
- [ ] `.claude/settings.json`에 `"defaultMode": "acceptEdits"` 추가하여 권한 확인 프롬프트 비활성화 — 개인 프로젝트에서 작업 속도 측정
- [ ] 공식 MCP 서버 목록 확인: https://modelcontextprotocol.io — 현재 프로젝트에 필요한 서버 검토
- [ ] Claude Pro 구독 상태 확인 후 간단한 웹 앱 기획안 작성 (예: 투두 리스트, 날씨 앱) — 기획안 품질이 결과물을 결정함

---

## 🔗 참고 자료

- [Claude Code context loss: the before and after of fixing it](https://dev.to/builtbyzac/claude-code-context-loss-the-before-and-after-of-fixing-it-3h2l)
- [How to stop Claude Code asking for permission on every command](https://dev.to/builtbyzac/how-to-stop-claude-code-asking-for-permission-on-every-command-2pfa)
- [Claude Code MCP servers: what they are and how to add them](https://dev.to/builtbyzac/claude-code-mcp-servers-what-they-are-and-how-to-add-them-4037)
- [【IT 자동화 삽질기 1편】 Claude로 바이브코딩 — 운세 앱...](https://blog.naver.com/forkpari/224220673094)

