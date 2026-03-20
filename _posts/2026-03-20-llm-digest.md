---
title: "[Tech] 2026-03-20 기술 동향: LLM"
author: gyuhwan
date: 2026-03-20 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** 토스 보안팀이 공개한 사례에서 LLM의 취약점 분석 능력은 3개월 만에 급격히 향상되었으며, 단순 패턴 매칭을 넘어 구조적 코드 이해가 가능해졌습니다. 핵심은 AI에게 IDE 수준의 코드 탐색 능력을 부여하는 것입니다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | LLM 기반 보안 취약점 자동 분석 기술 고도화 및 AI 에이전트 프로덕션 인프라 확산 | ⭐⭐⭐ |
| Tip | 대규모 코드베이스 분석 시 MCP + ctags + tree-sitter 조합으로 토큰 효율성 극대화 | ⭐⭐ |
| Trend | "AI를 쓰는 회사"에서 "AI를 만드는 회사"로의 전략 전환 가속화 | ⭐⭐⭐ |

---

## 💡 Deep Dive

### 1. LLM 기반 보안 자동화의 실전 구현 패턴

**핵심:** 토스 보안팀이 공개한 사례에서 LLM의 취약점 분석 능력은 3개월 만에 급격히 향상되었으며, 단순 패턴 매칭을 넘어 구조적 코드 이해가 가능해졌습니다. 핵심은 AI에게 IDE 수준의 코드 탐색 능력을 부여하는 것입니다.

**공통 의견:** 여러 사례에서 LLM의 실제 가치는 "얼마나 똑똑한가"보다 "얼마나 효율적으로 정보를 찾고 활용하는가"에 있다는 점이 일관되게 나타납니다. 외부 AI 서비스의 속도는 빠르지만, 프로덕션 환경에서는 비용과 정확성 문제로 자체 최적화가 필수입니다.

**실무 적용:**

- MCP(Model Context Protocol) 서버로 ctags 인덱싱 + tree-sitter 파싱을 조합하면 AI가 "함수 정의 찾기", "참조 검색" 같은 IDE 기능을 활용 가능
- ripgrep 기반 심볼 검색으로 불필요한 토큰 낭비를 줄이고, 정확한 코드 범위만 LLM에 전달
- 대용량 소스코드는 전체를 보내지 말고, 사전 인덱싱된 메타데이터를 통해 필요한 부분만 동적으로 로드

### 2. 외부 AI 서비스에서 자체 AI 전략으로의 전환 필요성

**핵심:** 무신사의 O4O 팀 사례에서 보듯이, 초기에는 Claude, GPT 같은 멀티모달 LLM으로 빠르게 검증하는 것이 효과적이지만, 서비스가 성숙하면서 "누구나 따라 할 수 있는 기술"이 되어 차별화 불가능해집니다. 이 시점에서 자체 데이터 자산을 기반으로 한 도메인 특화 AI 구축이 필수입니다.

**공통 의견:** 토스의 보안 자동화, 무신사의 체형 분석, Lawmadi의 한국 법률 AI 에이전트 모두 같은 패턴을 보입니다. 초기 빠른 검증 → 한계 발견 → 자체 데이터 + 도메인 튜닝으로의 전환입니다. 이는 단순히 기술 선택의 문제가 아니라 경쟁력 구조의 문제입니다.

**실무 적용:**

- 프로젝트 초기 3~6개월은 외부 LLM(Claude, GPT)으로 빠르게 MVP 검증하되, 이 기간에 수집되는 사용자 데이터와 도메인 지식을 체계적으로 축적
- 서비스 규모가 커지면서 API 비용이 증가하는 시점에 자체 파인튜닝 모델 또는 오픈소스 모델(예: NousCoder-14B) 도입 검토
- 도메인 특화 AI의 경우 실시간 검증 시스템(Lawmadi의 "Stage 4 DRF Verification")을 구축하여 할루시네이션 방지

### 3. AI 에이전트 프로덕션 인프라의 급속한 성숙

**핵심:** 2026년 3월 기준으로 AI 에이전트는 더 이상 연구 단계가 아니라 실제 프로덕션 서비스로 배포되는 단계에 진입했습니다. Railway의 AI-네이티브 클라우드, Anthropic의 Cowork, Salesforce의 Slackbot 에이전트 등이 동시에 출시되는 것은 인프라 레이어가 성숙했다는 신호입니다.

**공통 의견:** 에이전트 구현의 복잡도가 급격히 낮아졌습니다. "60줄의 Python"으로 기본 에이전트를 만들 수 있을 정도로 추상화되었으며, 이는 더 많은 팀이 LLM 기반 자동화에 진입할 수 있음을 의미합니다. 동시에 GDPval 벤치마크에서 최신 모델들이 전문가 수준의 성능(83% 이상)을 보이면서 실무 적용 신뢰도가 높아졌습니다.

**실무 적용:**

- 간단한 도구 호출 에이전트는 JSON Schema 기반 함수 레지스트리로 구현 가능 (OpenAI, Groq 표준)
- 복잡한 멀티스텝 작업은 에이전트가 자신의 오류를 감지하고 수정하는 능력이 있으므로, 명확한 도구 정의와 피드백 루프만 있으면 신뢰성 확보 가능
- 코드 생성 작업(Claude Code 대체)은 이제 오픈소스 모델(NousCoder-14B)로도 충분하므로, 비용 최적화 검토 필요

---

## 🛠️ 지금 당장 해볼 것

- [ ] **MCP 서버 기초 학습** — Anthropic 공식 MCP 문서에서 "Building an MCP Server" 튜토리얼 시작 (검색: `site:github.com/anthropics/mcp-servers`)

- [ ] **ctags + tree-sitter 조합 테스트** — 자신의 프로젝트에서 `ctags -R .` 실행 후 tree-sitter로 함수 범위 파싱해보기 (검색: `site:github.com tree-sitter python bindings`)

- [ ] **NousCoder-14B 로컬 실행** — Ollama 또는 LM Studio에서 `ollama pull nous-coder:14b` 실행하여 Claude Code 대체 가능성 검증

- [ ] **간단한 에이전트 프로토타입 구현** — OpenAI API + 함수 호출로 "계산기 + 웹 검색" 에이전트 만들기 (검색: `site:github.com openai function calling example python`)

---

## 🔗 참고 자료

- [LLM을 이용한 서비스 취약점 분석 자동화 #2](https://toss.tech/article/vulnerability-analysis-automation-2)
- [AI를 쓰는 회사 vs AI를 만드는 회사](https://techblog.musinsa.com/ai%EB%A5%BC-%EC%93%B0%EB%8A%94-%ED%9A%8C%EC%82%AC-vs-ai%EB%A5%BC-%EB%A7%8C%EB%93%9C%EB%8A%94-%ED%9A%8C%EC%82%AC-47932e6565d3?source=rss----f107b03c406e---4)
- [The LLM and AI Agent Releases That Actually Matter This Week, March 2026](https://dev.to/aibughunter/the-llm-and-ai-agent-releases-that-actually-matter-this-week-march-2026-54pp)
- [I'm 온유, Leader 08 of Lawmadi OS — Your AI Lease & Housing Expert for Korean Law](https://dev.to/peter_choe_74edfaa33306ed/im-onyu-leader-08-of-lawmadi-os-your-ai-lease-housing-expert-for-korean-law-ci4)
- [Agents in 60 lines of python : Part 2](https://dev.to/ahd_1337/agents-in-60-lines-of-python-part-2-4g6d)

