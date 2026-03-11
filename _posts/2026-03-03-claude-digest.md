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

**핵심:** Claude Code를 단순 채팅 도구로만 쓰면 기능의 20%만 활용하는 것(추정). CLAUDE.md, Hooks, MCP 서버 연결이라는 3가지 설정으로 완전히 다른 경험을 만들 수 있습니다. 2026년 3월 업데이트로 HTTP Hooks가 추가되어 쉘 커맨드 대신 URL로 JSON을 주고받는 웹훅 방식의 자동화가 가능해졌습니다. Claude Code는 3가지 핸들러 타입(command, prompt, agent)을 지원하며, 이는 command-only인 Cursor와 Copilot 대비 차별점입니다(원문 기준).

**공통 의견:** 여러 소스에서 일관되게 강조하는 부분은 "설정 없이는 Claude Code가 매번 처음부터 시작한다"는 점. CLAUDE.md 지침만으로는 컨텍스트 압력에 의해 무시될 수 있지만, Hooks로 강제하면 매번 확실히 실행됩니다.

**실무 적용:**

- **CLAUDE.md 계층 구조 활용**: 전역 설정(`~/.claude/CLAUDE.md`) → 프로젝트 설정(`./CLAUDE.md`) → 로컬 개인 설정(`./.claude/CLAUDE.md`)으로 나누어 관리

```markdown
# ./CLAUDE.md — 프로젝트 레벨 컨텍스트 (팀 전체 공유, git에 커밋)

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

- **Hooks로 자동화 강제**: PreToolUse 훅으로 위험 동작 차단, PostToolUse 훅으로 파일 수정 후 자동 포맷팅. CLAUDE.md 지침은 잊힐 수 있지만, Hook은 매번 실행됩니다.

```json
// .claude/settings.json — Hooks 설정 (3가지 핸들러 타입 지원)
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "bash -c 'echo \"$TOOL_INPUT\" | grep -qE \"rm -rf|git reset --hard\" && exit 1 || exit 0'",
        "description": "위험한 bash 명령어 차단"
      },
      {
        "matcher": "Edit|Write",
        "command": "bash -c '[[ ! \"$FILE_PATH\" =~ \\.env$ ]] || exit 1'",
        "description": ".env 파일 수정 차단"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "command": "bash -c 'if [[ \"$FILE_PATH\" =~ \\.py$ ]]; then black --check \"$FILE_PATH\"; fi'",
        "description": "Python 파일 저장 후 black 포맷 검증"
      },
      {
        "matcher": "Write",
        "command": "bash -c 'if [[ \"$FILE_PATH\" =~ \\.(ts|tsx|js)$ ]]; then npx prettier --check \"$FILE_PATH\"; fi'",
        "description": "JS/TS 파일 저장 후 prettier 검증"
      }
    ]
  }
}
```

- **MCP 서버 연결로 외부 시스템 통합**: GitHub, PostgreSQL, Playwright 같은 도구를 Claude Code에 직접 연결. 단, `enableAllProjectMcpServers`는 반드시 `false`로 유지 — 프로젝트 `.mcp.json`은 git에 포함되므로 손상된 리포가 악성 MCP 서버를 배포할 수 있습니다.

```bash
# MCP 서버 연결 및 보안 확인 워크플로우
# Step 1: 현재 연결된 MCP 서버 확인
claude "/mcp"           # 연결된 MCP 서버 목록

# Step 2: 사용 가능한 도구 확인
claude "/tools"         # MCP 서버가 제공하는 도구 목록

# Step 3: 프로젝트에 MCP 서버 추가 (팀 공유용)
cat > .mcp.json << 'EOF'
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GH_TOKEN}" }
    }
  }
}
EOF
git add .mcp.json && git commit -m "chore: add shared MCP config"

# Step 4: 보안 — 특정 도구만 pre-allow (와일드카드 금지)
# .claude/settings.json에서 allowedTools 설정
```

---

### 2. Anthropic 무료 강의 13개 로드맵

**핵심:** Anthropic이 2026년 3월 2일 Anthropic Academy에서 13개 무료 강의를 공개했습니다. 모든 강의가 무료이며 이메일 등록만으로 수강 가능하고, 수료 시 인증서를 발급합니다(원문 기준). 개발자에게 실질적 가치가 있는 핵심 5개를 순서대로 학습하면 Claude 생태계의 핵심을 10시간 안에 마스터할 수 있습니다(추정).

**공통 의견:** 모든 소스가 "Claude Code in Action" → "Building with Claude API" → "MCP 입문/심화" 순서를 강력 추천. 각 강의가 다음 단계의 기초가 되는 구조로 설계되어 있습니다.

**실무 적용:**

- **1주차: Claude Code 마스터** — "Claude Code in Action"(1시간)으로 CLAUDE.md, Custom Command, Hook, MCP 기초 학습

```shell
# 강의 따라하기 — Claude Code 첫 설정
npm install -g @anthropic-ai/claude-code
claude --version                    # 설치 확인

# 프로젝트에서 바로 시작
cd ~/my-project
claude "이 프로젝트의 구조를 분석하고 CLAUDE.md 초안을 작성해줘"

# 생성된 CLAUDE.md 확인 후 커밋
git add CLAUDE.md && git commit -m "docs: add CLAUDE.md for AI context"

# /insights 커맨드 활용 — Claude가 반복 패턴 분석 후 개선 제안
claude "/insights"
# 제안된 내용을 CLAUDE.md에 규칙으로 추가하거나 Hook으로 강제
```

- **2주차: API 전체 스펙트럼** — "Building with Claude API"(16 레슨, 2~3일)로 Tool Use, RAG 파이프라인, Extended Thinking, Prompt Caching, Agent/Workflow 패턴 습득

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
# Claude가 tool_use 블록 반환 → 함수 실행 → 결과를 다시 Claude에 전달
```

- **3주차: MCP 심화** — "Introduction to MCP"(8 레슨)로 Tools/Resources/Prompts 프리미티브 이해 후, "Advanced MCP"(8 레슨)로 Sampling, Notifications, Transport 메커니즘 학습

```python
# MCP 서버 기본 구조 — FastMCP로 5분 만에 서버 구축
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-tools")

@mcp.tool()
def search_jira(query: str, max_results: int = 10) -> str:
    """JIRA 이슈를 검색합니다"""
    # JIRA REST API 호출
    import requests
    resp = requests.get(
        f"{JIRA_URL}/rest/api/3/search",
        params={"jql": query, "maxResults": max_results},
        auth=(JIRA_EMAIL, JIRA_TOKEN)
    )
    issues = resp.json().get("issues", [])
    return "\n".join(f"[{i['key']}] {i['fields']['summary']}" for i in issues)

# 실행: mcp.run()
# .mcp.json에 등록 후 Claude Code에서 자동 사용 가능
```

---

### 3. 실제 앱 구축을 위한 3가지 패턴

**핵심:** Claude API 호출만으로는 서비스가 아닙니다. Tool Use(외부 함수 호출), RAG(데이터 검색 주입), Agent/Workflow 패턴(작업 흐름 설계)이 결합되어야 실제 제품이 됩니다.

**공통 의견:** 모든 사례에서 "Workflow부터 시작하고 안정화 후 Agent 추가"라는 원칙을 강조. 예측 가능한 구조로 먼저 검증한 후 유연성을 더하는 방식이 프로덕션 환경에서 안전합니다.

**실무 적용:**

- **Tool Use**: 결정론적 함수를 Claude에 노출. Claude가 필요할 때 자동으로 호출

```python
# RAG 파이프라인 기초 — 청킹 + 임베딩 + 검색
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 1. 문서 청킹 — chunk_size와 overlap 조정이 품질 결정
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,       # 토큰 수가 아닌 문자 수 기준
    chunk_overlap=50,     # 청크 간 문맥 연결 보장
    separators=["\n\n", "\n", ". ", " "]  # 의미 단위 분리
)
chunks = splitter.split_text(document_text)

# 2. 임베딩 + 벡터 저장 (Supabase pgvector 예시)
from supabase import create_client
supabase = create_client(SUPABASE_URL, SUPABASE_KEY)

for chunk in chunks:
    embedding = get_embedding(chunk)  # Voyage AI 또는 OpenAI
    supabase.table("documents").insert({
        "content": chunk,
        "embedding": embedding
    }).execute()

# 3. 검색 후 Claude 컨텍스트 주입
results = supabase.rpc("match_documents", {
    "query_embedding": query_embedding,
    "match_count": 5
}).execute()

# 4. Claude에 검색 결과를 컨텍스트로 전달
context = "\n---\n".join(r["content"] for r in results.data)
response = client.messages.create(
    model="claude-haiku-4-5-20251001",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": f"다음 문서를 기반으로 답변해줘:\n{context}\n\n질문: {question}"
    }]
)
```

- **Workflow 설계**: Workflow부터 시작 → 안정화 후 Agent 추가가 프로덕션 안전 원칙

```yaml
# Workflow 설계 예시 — 기술 블로그 큐레이션 파이프라인
steps:
  - name: collect_articles
    type: tool_use
    tool: rss_collector
    input: { feeds: "{{ config.rss_feeds }}", days_back: 2 }

  - name: evaluate_quality
    type: parallel
    tasks:
      - { name: "relevance", model: "haiku", prompt: "relevance 0-10 평가..." }
      - { name: "freshness", model: "haiku", prompt: "freshness 0-10 평가..." }

  - name: generate_digest
    type: tool_use
    tool: refiner
    input:
      articles: "{{ evaluate_quality.passed }}"
      model: "haiku"
      max_tokens: 3000

  - name: publish
    type: conditional
    condition: "{{ generate_digest.quality_gate == 'pass' }}"
    on_true: { tool: publisher, action: "batch_commit" }
    on_false: { tool: notifier, action: "send_hold_alert" }
```

---

### 4. 2026년 개발자 생존 전략

**핵심:** Agentic AI와 Claude Code의 등장으로 코딩 작업의 성격이 변하고 있습니다. Claude Code가 VS Code 에이전트 마켓플레이스 1위를 차지하고(원문 기준), Anthropic 연 매출이 20억 달러에 근접하면서(2026년 1월 기준, 추정) AI 도구가 일상화되는 속도가 가속되고 있습니다.

**공통 의견:** 업계 전문가들이 일관되게 경고하는 부분은 기초 코딩 작업은 자동화되고, 아키텍처 설계와 문제 정의 능력이 핵심 경쟁력이 된다는 점입니다.

**실무 적용:**

- **Claude Code 숙련도 높이기**: 단순 채팅이 아닌 CLAUDE.md, Hooks, MCP를 활용한 고급 활용법 습득

```shell
# 생존 전략 실천 — 오늘부터 시작하는 3단계

# Step 1: 현재 프로젝트에 CLAUDE.md 추가
claude "이 프로젝트를 분석하고 CLAUDE.md를 작성해줘.
  기술 스택, 아키텍처 패턴, 코딩 규칙, 금지 사항을 포함해줘"

# Step 2: Hook으로 안전장치 설정
# .claude/settings.json에 PreToolUse 추가
python3 -c "
import json
settings = {
    'hooks': {
        'PreToolUse': [{
            'matcher': 'Bash',
            'command': 'bash -c \'echo \"\$TOOL_INPUT\" | grep -qE \"rm -rf\" && exit 1 || exit 0\'',
            'description': 'rm -rf 차단'
        }]
    }
}
with open('.claude/settings.json', 'w') as f:
    json.dump(settings, f, indent=2)
"

# Step 3: 일주일간 Claude Code로 모든 PR 작성 후 효율 측정
git log --since="7 days ago" --oneline | wc -l
```

- **AI 시스템 설계 능력 강화**: Tool Use, RAG, Workflow 패턴을 이해하고 실제 프로젝트에 적용
- **지속적 학습 체계 구축**: Anthropic Academy 강의를 정기적으로 학습하고, 새로운 MCP 서버나 기능을 빠르게 프로젝트에 통합하는 민첩성 확보

```python
# 개발자 역량 자가 진단 (AI 시대 기준)
SKILL_CHECKLIST = {
    "CLAUDE.md 작성 및 계층 구조 설정": False,
    "PreToolUse/PostToolUse Hook 설정": False,
    "MCP 서버 연결 및 .mcp.json 관리": False,
    "Tool Use + RAG 파이프라인 구축": False,
    "Workflow → Agent 전환 판단": False,
    "모델별 비용 최적화 (Haiku/Sonnet/Opus)": False,
    "AI 생성 코드의 보안 취약점 식별": False,
}

# 3개 이상 False → Anthropic Academy 수강 권장
# https://anthropic.skilljar.com/
```

---

## 🛠️ 지금 당장 해볼 것

- [ ] `https://anthropic.skilljar.com/` 접속하여 "Claude Code in Action"(1시간) 수강 시작 — 무료, 수료증 발급
- [ ] 현재 프로젝트에서 `claude "이 프로젝트를 분석하고 CLAUDE.md 초안을 작성해줘"` 실행 후 `git add CLAUDE.md && git commit -m "docs: add CLAUDE.md"`
- [ ] `.claude/settings.json`에 PreToolUse 훅 추가: `bash -c '[[ ! "$FILE_PATH" =~ \.env$ ]] || exit 1'` 로 `.env` 수정 차단

---

## 🤔 생각해볼 질문

> 현재 팀의 온보딩 과정에서 신규 입사자가 가장 오래 헤매는 부분은 무엇이며, 그 내용을 CLAUDE.md에 어떤 형식(개요/규칙/금지사항/FAQ)으로 담으면 Claude Code가 즉시 활용할 수 있겠는가?

> Tool Use + RAG + Workflow 3가지 패턴 중, 지금 담당하고 있는 서비스에 가장 먼저 적용한다면 어떤 패턴이고, 그 이유는 무엇인가? Workflow로 먼저 검증한 후 Agent로 전환하는 시점을 어떤 기준으로 판단하겠는가?

---

## 🔗 참고 자료

- [Claude Code, 깔아놓고 채팅만 하고 있다면 — CLAUDE.md, Hooks, MCP 설정법](https://dev.to/ji_ai/claude-code-ggalanohgo-caetingman-hago-issdamyeon-claudemd-hooks-mcp-seoljeongbeob-3o18)
- [Anthropic Academy — 13 Free Courses](https://anthropic.skilljar.com/)
- [Hooks reference — Claude Code Docs](https://code.claude.com/docs/en/hooks)
- [awesome-claude-code — Skills, Hooks, Commands curated list](https://github.com/hesreallyhim/awesome-claude-code)
- [Building Real Apps With the Claude API — Tool Use, RAG, and Agent Patterns](https://dev.to/ji_ai/building-real-apps-with-the-claude-api-tool-use-rag-and-agent-patterns-explained-kcb)
- [My Claude Code Setup: MCP, Hooks, Skills — Real Usage 2026](https://okhlopkov.com/claude-code-setup-mcp-hooks-skills-2026/)
- [Understanding Claude Code's Full Stack: MCP, Skills, Subagents, and Hooks](https://alexop.dev/posts/understanding-claude-code-full-stack/)
