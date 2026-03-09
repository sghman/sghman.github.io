---
title: "[Tech] 2026-03-03 기술 동향: claude"
author: gyuhwan
date: 2026-03-03 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, claude]
description: "핵심:** Claude Code를 단순 채팅 도구로만 쓰면 기능의 20%만 활용하는 것. CLAUDE.md, Hooks, MCP 서버 연결이라는 3가지 설정으로 완전히 다른 경험을 만들 수 있다."
auto_generated: true
permalink: /posts/2026-03-03-claude-digest/
---

## 🚀 30초 퀵 서머리

- **핵심 키워드**: CLAUDE.md 계층 구조, Hooks 자동화, MCP 서버 연결, Anthropic 무료 강의
- **오늘의 Top Pick**: CLAUDE.md + Hooks + MCP 3계층 설정 — 30분 안에 Claude Code를 aiOS로 전환
- **한줄 평**: Claude Code 설치만으로는 20%, 설정 파일 3개만 추가하면 나머지 80%가 열린다

---

## 🕑 데일리 큐레이션 보드

| 우선순위 | 도메인 | 핵심 주제 | 추천 대상 | 난이도 |
|:---:|:---|:---|:---:|:---:|
| 🔴Must | DevTools | CLAUDE.md, Hooks, MCP 설정법 | BE/FE/Infra | ⭐⭐ |
| 🔴Must | Education | Anthropic 무료 강의 5개 로드맵 | BE/FE | ⭐ |
| 🟡View | Architecture | Tool Use + RAG + Agent 패턴 조합 | BE/Infra | ⭐⭐⭐ |
| ⚪Skim | Career | 2026년 개발자 생존 전략 | BE/FE/Infra | ⭐ |

---

## 🔍 Deep Dive

### 1. Claude Code의 진짜 힘은 설정 파일에 있다

**핵심:** Claude Code를 단순 채팅 도구로만 쓰면 기능의 20%만 활용하는 것(추정). CLAUDE.md, Hooks, MCP 서버 연결이라는 3가지 설정으로 완전히 다른 경험을 만들 수 있다. 2026년 3월 업데이트로 HTTP Hooks가 추가되어 쉘 커맨드 대신 URL로 JSON을 주고받는 웹훅 방식의 자동화가 가능해졌다.

**공통 의견:** 여러 소스에서 일관되게 강조하는 부분은 "설정 없이는 Claude Code가 매번 처음부터 시작한다"는 점. 세션이 끝나면 모든 학습 내용이 사라지므로, CLAUDE.md 파일로 프로젝트 규칙을 영구 저장하는 것이 필수다.

**실무 적용:**

- **CLAUDE.md 계층 구조 활용**: 전역 설정(`~/.claude/CLAUDE.md`) → 프로젝트 설정(`./CLAUDE.md`) → 로컬 개인 설정(`./.claude/CLAUDE.md`)으로 나누어 관리

```markdown
# ./CLAUDE.md — 프로젝트 레벨 컨텍스트 (팀 전체 공유)

## 프로젝트 개요
이 프로젝트는 FastAPI + PostgreSQL 기반 REST API 서버입니다.

## 코딩 규칙
- Python 3.12, type hints 필수
- 모든 엔드포인트에 Pydantic v2 스키마 사용
- 테스트: pytest + httpx AsyncClient

## 금지 사항
- ORM 없이 raw SQL 직접 실행 금지
- print() 대신 structlog 사용
- .env 파일 직접 수정 금지 (hooks로 차단)
```

- **Hooks로 자동화 강제**: PostToolUse 훅으로 파일 수정 후 자동 포맷팅, PreToolUse 훅으로 위험 동작 차단

```json
// .claude/settings.json — Hooks 설정 예시
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "write_file",
        "command": "prettier --write $FILE_PATH",
        "description": "파일 저장 후 자동 포맷팅"
      }
    ],
    "PreToolUse": [
      {
        "matcher": "edit_file",
        "command": "bash -c '[[ ! \"$FILE_PATH\" =~ \\.env ]] || exit 1'",
        "description": ".env 파일 수정 차단"
      }
    ]
  }
}
```

- **MCP 서버 연결로 외부 시스템 통합**: GitHub, PostgreSQL, Playwright 같은 도구를 Claude Code에 직접 연결해 터미널 없이 작업 완료. 단, 너무 많은 MCP 서버는 컨텍스트를 소모하므로(200k → ~70k, 원문 기준) lazy loading 활용 권장

```bash
# MCP 서버 연결 확인 커맨드
claude "/mcp"           # 현재 연결된 MCP 서버 목록 확인
claude "/tools"         # 사용 가능한 도구 목록 확인

# 프로젝트에 MCP 서버 추가 (팀 공유용)
# .mcp.json 파일을 프로젝트 루트에 생성하고 git commit
```

---

### 2. Anthropic 무료 강의 5개 로드맵

**핵심:** Anthropic이 공개한 13개 강의 중 개발자에게 실질적 가치가 있는 것은 5개. 이들을 순서대로 학습하면 Claude 생태계의 핵심을 10시간 안에 마스터할 수 있다(추정).

**공통 의견:** 모든 소스가 "Claude Code in Action" → "Building with Claude API" → "MCP 입문/심화" 순서를 강력 추천. 각 강의가 다음 단계의 기초가 되는 구조로 설계되어 있다.

**실무 적용:**

- **1주차: Claude Code 마스터** - "Claude Code in Action"(1시간)으로 CLAUDE.md, Custom Command, Hook, MCP 기초 학습

```shell
# 강의 따라하기 — Claude Code 첫 설정
npm install -g @anthropic-ai/claude-code
claude --version                    # 설치 확인

# 프로젝트에서 바로 시작
cd ~/my-project
claude "이 프로젝트의 구조를 분석하고 CLAUDE.md 초안을 작성해줘"

# 생성된 CLAUDE.md 확인 후 커밋
git add CLAUDE.md && git commit -m "docs: add CLAUDE.md for AI context"
```

- **2주차: API 전체 스펙트럼** - "Building with Claude API"(16 레슨, 2~3일)로 Tool Use, RAG 파이프라인, Extended Thinking, Prompt Caching, Agent/Workflow 패턴 습득

```python
# Tool Use 기초 예시 — Claude API에 함수 등록
import anthropic

client = anthropic.Anthropic()

tools = [{
    "name": "get_stock_price",
    "description": "주어진 종목의 현재 주가를 조회합니다",
    "input_schema": {
        "type": "object",
        "properties": {
            "ticker": {"type": "string", "description": "종목 코드 (예: AAPL)"}
        },
        "required": ["ticker"]
    }
}]

response = client.messages.create(
    model="claude-haiku-4-5-20251001",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "애플 주가 알려줘"}]
)
```

- **3주차: MCP 심화** - "Introduction to MCP"(8 레슨)로 Tools/Resources/Prompts 프리미티브 이해 후, "Advanced MCP"(8 레슨)로 Sampling, Notifications, Transport 메커니즘 학습

---

### 3. 실제 앱 구축을 위한 3가지 패턴

**핵심:** Claude API 호출만으로는 서비스가 아니다. Tool Use(외부 함수 호출), RAG(데이터 검색 주입), Agent/Workflow 패턴(작업 흐름 설계)이 결합되어야 실제 제품이 된다.

**공통 의견:** 모든 사례에서 "Workflow부터 시작하고 안정화 후 Agent 추가"라는 원칙을 강조. 예측 가능한 구조로 먼저 검증한 후 유연성을 더하는 방식이 프로덕션 환경에서 안전하다.

**실무 적용:**

- **Tool Use**: 결정론적 함수를 Claude에 노출. Claude가 필요할 때 자동으로 호출하고 결과를 받아 최종 응답 생성

```python
# RAG 파이프라인 기초 — 청킹 + 임베딩 + 검색
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 1. 문서 청킹
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " "]
)
chunks = splitter.split_text(document_text)

# 2. 임베딩 + 벡터 저장 (Supabase pgvector 예시)
from supabase import create_client
supabase = create_client(SUPABASE_URL, SUPABASE_KEY)

for chunk in chunks:
    embedding = get_embedding(chunk)  # OpenAI or Voyage AI
    supabase.table("documents").insert({
        "content": chunk,
        "embedding": embedding
    }).execute()

# 3. 검색 후 Claude 컨텍스트 주입
results = supabase.rpc("match_documents", {
    "query_embedding": query_embedding,
    "match_count": 5
}).execute()
```

- **RAG 파이프라인**: 청킹 → 임베딩 → BM25+벡터 검색 → 리랭킹 → 컨텍스트 주입 순서로 구현. "아는 척" 하는 대신 실제 데이터 기반 답변 보장
- **Workflow 설계**: 가격 조회(Tool Use) + 정책 검색(RAG) + 병렬 다축 분석(Workflow)을 조합. 각 단계가 명확하고 테스트 가능한 구조 유지

```yaml
# Workflow 설계 예시 — 부동산 분석 파이프라인
steps:
  - name: fetch_price
    type: tool_use
    tool: get_apartment_price
    input: { address: "{{ user_input.address }}" }

  - name: search_policy
    type: rag
    query: "{{ user_input.address }} 관련 부동산 정책 규제"
    top_k: 5

  - name: parallel_analysis
    type: parallel
    tasks:
      - { name: "시세 분석", prompt: "{{ fetch_price.result }}를 기반으로..." }
      - { name: "정책 영향", prompt: "{{ search_policy.result }}를 기반으로..." }
      - { name: "투자 전망", prompt: "종합 분석..." }
```

---

### 4. 2026년 개발자 생존 전략

**핵심:** Agentic AI와 Claude Code의 등장으로 코딩 작업의 성격이 변하고 있다. 단순 코드 작성 능력만으로는 경쟁력이 떨어지며, AI 도구를 능숙하게 다루고 시스템 설계 능력을 갖춘 개발자만 살아남는다.

**공통 의견:** 업계 전문가들이 일관되게 경고하는 부분은 "2026년이 분기점"이라는 점. Claude Code 같은 도구가 일상화되면서 기초 코딩 작업은 자동화되고, 아키텍처 설계와 문제 정의 능력이 핵심 경쟁력이 된다.

**실무 적용:**

- **Claude Code 숙련도 높이기**: 단순 채팅이 아닌 CLAUDE.md, Hooks, MCP를 활용한 고급 활용법 습득

```shell
# 생존 전략 실천 — 오늘부터 시작하는 3단계
# Step 1: 현재 프로젝트에 CLAUDE.md 추가
echo "# Project Context\n\n## Tech Stack\n- ...\n\n## Rules\n- ..." > CLAUDE.md

# Step 2: Hook으로 안전장치 설정
claude "/hooks"   # 인터랙티브 메뉴에서 PostToolUse 설정

# Step 3: 일주일간 Claude Code로 모든 PR 작성 후 효율 측정
# git log로 커밋 빈도 비교
git log --since="7 days ago" --oneline | wc -l
```

- **AI 시스템 설계 능력 강화**: Tool Use, RAG, Workflow 패턴을 이해하고 실제 프로젝트에 적용
- **지속적 학습 체계 구축**: Anthropic 무료 강의 같은 공식 자료를 정기적으로 학습하고, 새로운 MCP 서버나 기능을 빠르게 프로젝트에 통합하는 민첩성 확보

---

## 🛠️ 지금 당장 해볼 것

- [ ] Anthropic 무료 강의 첫 번째 "Claude Code in Action"(1시간) 수강 시작 — `site:anthropic.com courses` 검색하여 접속
- [ ] 현재 진행 중인 프로젝트에 CLAUDE.md 파일 생성 — 기술 스택, 코딩 규칙, 금지 사항 3가지만 적어서 커밋
- [ ] `.claude/settings.json`에 PostToolUse 훅 하나 추가 — 파일 저장 후 `black --check` 또는 `prettier --check` 자동 실행 설정

---

## 🤔 생각해볼 질문

> 현재 팀의 온보딩 과정에서 신규 입사자가 가장 오래 헤매는 부분은 무엇이며, 그 내용을 CLAUDE.md에 어떤 형식으로 담으면 AI가 바로 활용할 수 있겠는가?

> Tool Use + RAG + Workflow 3가지 패턴 중, 지금 담당하고 있는 서비스에 가장 먼저 적용한다면 어떤 패턴이고, 그 이유는 무엇인가?

---

## 🔗 참고 자료

- [Claude Code, 깔아놓고 채팅만 하고 있다면 — CLAUDE.md, Hooks, MCP 설정법](https://dev.to/ji_ai/claude-code-ggalanohgo-caetingman-hago-issdamyeon-claudemd-hooks-mcp-seoljeongbeob-3o18)
- [If You Installed Claude Code and Only Chat With It — You're Missing the Point](https://dev.to/ji_ai/if-you-installed-claude-code-and-only-chat-with-it-youre-missing-the-point-4elg)
- [Anthropic이 공짜로 풀어놓은 13개 강의, 내가 다 뜯어봤다](https://dev.to/ji_ai/anthropici-gongjjaro-puleonoheun-13gae-gangyi-naega-da-ddeudeobwassda-3h0d)
- [Anthropic Dropped 13 Free Courses — I Broke Down Every Single One](https://dev.to/ji_ai/anthropic-dropped-13-free-courses-i-broke-down-every-single-one-p87)
- [Building Real Apps With the Claude API — Tool Use, RAG, and Agent Patterns Explained](https://dev.to/ji_ai/building-real-apps-with-the-claude-api-tool-use-rag-and-agent-patterns-explained-kcb)
- [My Claude Code Setup: MCP, Hooks, Skills — Real Usage 2026](https://okhlopkov.com/claude-code-setup-mcp-hooks-skills-2026/)
- [Understanding Claude Code's Full Stack: MCP, Skills, Subagents, and Hooks Explained](https://alexop.dev/posts/understanding-claude-code-full-stack/)
