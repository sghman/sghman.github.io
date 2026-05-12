---
title: "[Tech] 2026-05-12 기술 동향: LLM"
author: gyuhwan
date: 2026-05-12 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** LangChain + Ollama + PostgreSQL을 조합하여 자연어로 데이터베이스를 쿼리할 수 있는 AI 에이전트를 구축. SQL 생성 및 실행 과정에서 안전성을 확보하기 위해 커스텀 SQL 검증 레이어 추가."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | PostgreSQL AI Agent + LangChain/Ollama로 자연어 SQL 쿼리 실행 | ⭐⭐⭐ |
| Trend | 2026년 LLM 엔지니어링 로드맵: 실전 애플리케이션 구축 중심 | ⭐⭐⭐ |
| Security | OWASP TOP 10 for LLM: 데이터 오염, 민감정보 유출 위험 | ⭐⭐⭐ |
| Tip | Claude Code 아키텍처: 멀티에이전트 오케스트레이션 패턴 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. 로컬 LLM 기반 데이터베이스 에이전트 구축

**핵심:** LangChain + Ollama + PostgreSQL을 조합하여 자연어로 데이터베이스를 쿼리할 수 있는 AI 에이전트를 구축. SQL 생성 및 실행 과정에서 안전성을 확보하기 위해 커스텀 SQL 검증 레이어 추가.

**공통 의견:** 2026년 LLM 애플리케이션의 핵심은 단순 챗봇이 아닌 **구조화된 출력 시스템**과 **실제 데이터 연동**. 로컬 LLM 사용으로 비용 절감 및 데이터 프라이버시 확보 가능.

**실무 적용:**

- `@tool` 데코레이터로 `list_tables()`, `get_table_schema()` 같은 안전한 도구 정의 → 에이전트가 동적으로 스키마 인식
- 환경변수로 DB 자격증명 관리 (하드코딩 금지)
- SQL 실행 전 파괴적 쿼리(DROP, DELETE) 차단하는 검증 레이어 필수

### 2. 2026년 LLM 엔지니어링 실전 로드맵

**핵심:** 단순 프롬프팅을 넘어 **Python API 통합 → NLP 기초 → Transformer 이해 → RAG 파이프라인 → 실제 애플리케이션**의 순차적 학습 경로. 각 단계별 구체적 학습 자료와 도구 제시.

**공통 의견:** 70% 이상의 조직이 AI를 도입했지만, 실제 LLM 애플리케이션 구축 능력은 부족. 2026년은 "프롬프트 엔지니어"에서 "LLM 엔지니어"로의 전환 시점.

**실무 적용:**

- 임베딩 + 벡터DB(FAISS, Pinecone, Chroma)로 자신의 데이터 추가
- RAG 파이프라인: 문서 청킹 → 검색 → 프롬프트 컨텍스트 설계 → 평가
- 작고 실행 가능한 애플리케이션부터 시작 (완벽한 대규모 시스템 목표 금지)

### 3. LLM 보안 취약점: 데이터 오염과 민감정보 유출

**핵심:** OWASP TOP 10 for LLM에서 LLM02(민감정보 유출)와 LLM04(데이터/모델 오염)가 가장 심각한 위협. 학습 데이터 단계부터 배포 후까지 전 수명주기에서 데이터 무결성 보호 필요.

**공통 의견:** LLM 서비스 탑재 시 모델 출력을 통해 개인정보, 독점 알고리즘, 기밀정보가 유출될 수 있음. 특히 신뢰할 수 없는 학습 데이터는 모델 성능 저하 및 보안 위협 야기.

**실무 적용:**

- 입력 데이터 검증 및 필터링 강화
- 출력 마스킹: 민감정보(이메일, 전화번호, 주민번호) 자동 감지 및 제거
- 학습 데이터 출처 검증 및 버전 관리

### 4. 도메인 특화 LLM의 부상

**핵심:** 화학, 의료, 금융 등 특정 산업의 복잡한 기호·수식·도면을 다루기 위해 도메인 특화 LLM 개발 가속화. 텍스트 중심 일반 LLM의 한계 극복.

**공통 의견:** 국내 AI 스타트업(워트인텔리전스, 트릴리온랩스)도 화학 산업 특화 LLM 공동 개발 중. 특허 데이터와 R&D 활용으로 산업별 맞춤형 AI 솔루션 확대.

**실무 적용:**

- 자신의 산업 특화 데이터셋으로 파인튜닝
- 도메인 온톨로지 구축 (기호, 수식, 도면 처리)
- 특허 데이터베이스 연동으로 최신 기술 정보 반영

---

## 🛠️ 지금 당장 해볼 것

- [ ] **PostgreSQL AI Agent 로컬 구축** — GitHub 저장소 클론 후 Ollama 설치 및 LangChain 예제 실행: `git clone https://github.com/gauravk_tweet/postgres-agent && cd postgres-agent && pip install -r requirements.txt`

- [ ] **LLM 엔지니어링 로드맵 첫 단계 시작** — Python 기초 강화: `site:github.com langchain-ai/langchain` 검색 후 공식 튜토리얼 실행

- [ ] **OWASP LLM 보안 체크리스트 작성** — 자신의 LLM 애플리케이션에서 민감정보 유출 가능 지점 3가지 이상 식별 및 마스킹 로직 추가

- [ ] **벡터DB 실습** — Chroma 설치 후 간단한 RAG 파이프라인 구축: `pip install chroma-db && python -c "import chromadb; client = chromadb.Client()"`

---

## 🔗 참고 자료

- [Build a Secure PostgreSQL AI Agent with LangChain + Ollama](https://dev.to/gauravk_tweet/build-a-secure-postgresql-ai-agent-with-langchain-ollama-16c7)
- [Deconstructing Claude Code Architecture: A Deep Dive into Multi-Agent Orchestration](https://dev.to/khaitrang1995/deconstructing-claude-code-architecture-a-deep-dive-into-multi-agent-orchestration-3d8h)
- [루틴 : 써클 인터내셔널 그룹(CRCL) 26.1Q 실적 &amp; LLM PPT](https://blog.naver.com/parksb_ks/224282813930)
- [OWASP TOP 10 for LLM - LLM04 2025 데이터 및 모델 오염](https://blog.naver.com/jndnghn/224282809393)
- [OWASP TOP 10 for LLM - LLM02 2025 민감 정보 유출](https://blog.naver.com/jndnghn/224282776359)
- [화학 특화 LLM (도메인AI, 특허데이터, R&amp;D활용)](https://blog.naver.com/redfight100/224281216083)
- [How to Actually Learn LLMs in 2026 | Ex-Google, Microsoft Engineer](https://www.youtube.com/watch?v=U07MHi4Suj8)
- [Building LLM-Native Applications in 2026: A Practical Guide - LinkedIn](https://www.linkedin.com/pulse/building-llm-native-applications-2026-practical-guide-chetan-singh-otb6c)
- [The Practical LLM Builder Handbook: How to Build, Train, and ...](https://www.amazon.com/Practical-Builder-Handbook-Step-Step/dp/B0FW52Z1VD)
- [LLM Roadmap 2026: How to Learn Large Language Models from Scratch - Scaler](https://www.scaler.com/blog/llm-roadmap-2026-how-to-learn-large-language-models-from-scratch/)
- [LLM Engineering Roadmap (2026 Practical Guide) 🗺✨

If your goal is to build real LLM apps (not just prompts), follow this order. 🚀

1️⃣ Python + APIs 🐍🔌

You’ll spend most of your time wiring systems.

Learn:
→ functions, classes
→ working with APIs (requests, JSON)
→ async basics
→ environment variables

Resources
→ Python for Everybody
https://lnkd.in/gUqkvnGG
→ Introduction to Python
https://lnkd.in/g7xfYJVZ
→ MLTUT Python Basics Course
https://lnkd.in/gCqfyCGZ

2️⃣ Text Basics (NLP) 📝🧠

You don’t need heavy theory, just the essentials.

Learn:
→ tokenization
→ text cleaning
→ similarity (cosine)
→ basic embeddings idea

Resources
→ Natural Language Processing Specialization
https://lnkd.in/gz_xmqD9
→ NLP in Python
https://lnkd.in/gnpcJxhz

3️⃣ Transformers (What’s happening behind the API) 🤖🔍

Enough to not treat it like a black box.

Learn:
→ tokens, context window
→ attention (high level)
→ why embeddings work
→ limits of LLMs

Resources
→ Generative AI with Large Language Models
https://lnkd.in/gk3PPtyf
→ Hugging Face Transformers Course
https://lnkd.in/ggSR5JNb

4️⃣ Prompting (Make outputs reliable) 💬🎯

Treat prompts like code.

Learn:
→ few-shot examples
→ structured outputs (JSON)
→ system vs user instructions
→ simple evals (does it break?)

Resources
→ Prompt Engineering for ChatGPT
https://lnkd.in/gyg4EiJS
→ Prompt Engineering with LLMs
https://lnkd.in/gn67Mxga

5️⃣ Embeddings + Vector DBs 📊🗄

This is how you add your data.

Learn:
→ embedding generation
→ similarity search
→ indexing
Tools:
→ FAISS
→ Pinecone
→ Chroma

Resources
→ Working with Embeddings
https://lnkd.in/gnngPW4E
→ Vector Databases & Semantic Search
https://lnkd.in/gP2HdMmD

6️⃣ RAG Pipelines 🔗🔄

Most useful apps use this pattern.

Learn:
→ chunking documents
→ retrieval + ranking
→ prompt + context design
→ basic evaluation

Resources
→ Generative AI for Software Development
https://lnkd.in/g3uduecv
→ Build RAG Apps with LangChain
https://lnkd.in/ggXJjgDN

7️⃣ Build Real Applications 🛠💻

Keep them small and usable.

Build: | Coding WithMee](https://www.facebook.com/61580835871666/videos/llm-engineering-roadmap-2026-practical-guide-if-your-goal-is-to-build-real-llm-a/36089132047352023/)

