---
title: "[Tech] 2026-03-15 기술 동향: LLM"
author: gyuhwan
date: 2026-03-15 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** 전통적인 Python 조건문 대신 Markdown 프롬프트 파일 13개로 에이전트의 행동을 정의하는 방식이 등장했다. 9B 로컬 모델(qwen3.5:9b)만으로도 소셜 네트워크 자동 포스팅, 댓글, 답글 기능을 구현 가능하다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Amazon Rufus 38% 채택률 달성 — AI 쇼핑 에이전트의 실질적 영향 시작 | ⭐⭐⭐ |
| Tip | 자연언어 프롬프트를 코드처럼 사용하는 에이전트 아키텍처 설계 | ⭐⭐⭐ |
| Security | LLM 애플리케이션 대상 악성 봇 공격 탐지 및 차단 기법 | ⭐⭐ |
| Trend | Answer Engine Optimization(AEO) — SEO에서 AI 추천 최적화로 전환 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. AI 에이전트 아키텍처의 새로운 패러다임: 자연언어 기반 설계

**핵심:** 전통적인 Python 조건문 대신 Markdown 프롬프트 파일 13개로 에이전트의 행동을 정의하는 방식이 등장했다. 9B 로컬 모델(qwen3.5:9b)만으로도 소셜 네트워크 자동 포스팅, 댓글, 답글 기능을 구현 가능하다.

**공통 의견:** 코드는 "무엇을 결정할 것인가"의 프레임워크만 제공하고, 프롬프트가 "어떻게 결정할 것인가"를 담당한다. 이는 새로운 주제나 상황에 대응할 때 코드 수정 없이 프롬프트만 업데이트하면 되므로 유지보수성이 크게 향상된다.

**실무 적용:**

- 에이전트 행동 규칙을 `config/prompts/` 디렉토리의 Markdown 파일로 관리 (relevance.md, comment.md, reply.md 등)
- 의도적인 모호성을 프롬프트에 남겨두어 LLM이 문맥에 따라 판단하도록 설계
- 4가지 핵심 원칙(axioms)을 `contemplative-axioms.md`에 정의하여 에이전트의 철학적 기반 구축

### 2. AI 쇼핑 에이전트의 실질적 영향: Answer Engine Optimization 시대 도래

**핵심:** Amazon Rufus가 블랙프라이데이 2025에서 38% 채택률을 기록했으며, Rufus 사용자는 구매 완료율이 60% 높다. 이는 AI 에이전트가 더 이상 실험 단계가 아닌 매출 창출 도구임을 의미한다.

**공통 의견:** 전통 SEO의 "파란 링크 최적화"에서 "AI 추천 최적화"로 패러다임이 전환되고 있다. Rufus는 링크를 반환하지 않고 제품 추천을 반환하므로, 제품 데이터의 구조화와 의미론적 풍부함이 가시성을 결정한다.

**실무 적용:**

- 제품 설명을 "누가, 무엇을, 언제, 어디서, 왜" 관점에서 재작성 (키워드 스터핑 제거)
- 리뷰 데이터를 체계적으로 수집 및 구조화 (Rufus가 패턴 분석에 활용)
- Reddit, YouTube, 에디토리얼 등 크로스플랫폼 신뢰도 신호 구축

### 3. LLM 애플리케이션 보안: 악성 봇 공격의 현실적 위협

**핵심:** 단일 악성 봇이 초당 100,000개 이상의 요청을 생성할 수 있으며, 기존 IP 기반 레이트 리미터는 분산 공격에 무력하다. LLM 응답 파밍, 민감 데이터 스크래핑, 자격증명 스터핑이 주요 공격 벡터다.

**공통 의견:** Flask-Limiter 같은 단순 레이트 리미터만으로는 부족하며, 포괄적인 AI 보안 플랫폼과 LLM 방화벽이 필요하다. 에이전트 보안이 특히 취약한 영역으로 지적된다.

**실무 적용:**

- 단순 IP 기반 제한 대신 요청 패턴 분석 및 행동 기반 탐지 도입
- 추론 엔드포인트에 대한 DDoS 방어 및 속도 제한 강화
- LLM 방화벽을 통한 입출력 검증 및 악의적 프롬프트 차단

---

## 🛠️ 지금 당장 해볼 것

- [ ] **자연언어 에이전트 아키텍처 학습** — `site:github.com autonomous agent prompt markdown` 검색하여 오픈소스 에이전트 구현 사례 분석 및 로컬 9B 모델(Ollama에서 qwen3.5:9b 다운로드) 설치 후 간단한 프롬프트 기반 에이전트 프로토타입 작성

- [ ] **제품 데이터 구조화 감사** — 현재 운영 중인 전자상거래 플랫폼의 상위 10개 제품 설명을 "누가, 무엇을, 언제, 어디서, 왜" 프레임워크로 재작성하고, 리뷰 요약 프롬프트 작성 후 LLM으로 테스트 (`site:github.com product description optimization LLM`)

- [ ] **LLM 애플리케이션 보안 테스트** — Flask 앱에 Flask-Limiter 설정 후, `ab -n 10000 -c 100 http://localhost:5000/predict` 명령으로 분산 공격 시뮬레이션 실행 및 로그 분석 (Apache Bench 설치 필수)

---

## 🔗 참고 자료

- [Natural Language as Architecture — Controlling an Autonomous Agent with Prompts, Memory, and Fail-Safe Design](https://dev.to/shimo4228/natural-language-as-architecture-controlling-an-autonomous-agent-with-prompts-memory-and-m74)
- [How to Detect and Block Malicious Bots Targeting Your AI Application](https://dev.to/botguard/how-to-detect-and-block-malicious-bots-targeting-your-ai-application-19g1)
- [Amazon Rufus Hit 38% Adoption on Black Friday — What Every Developer and Brand Should Know](https://dev.to/abraf_kadar_87317f62d70ff/amazon-rufus-hit-38-adoption-on-black-friday-what-every-developer-and-brand-should-know-1hnk)
- [요새는 직접 자료 만들어서 수업중.](https://blog.naver.com/rims_shelter/224217015131)
- [[AI] 구글 제미나이 모델 완전 정리](https://blog.naver.com/kosehun/224216989271)

