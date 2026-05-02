---
title: "[Tech] 2026-05-02 기술 동향: LLM"
author: gyuhwan
date: 2026-05-02 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** 2026년 4월 15일부터 Exa의 `startCrawlDate`/`endCrawlDate` 필터가 요청을 받아도 실제로는 작동하지 않음. HTTP 200 응답을 반환하지만 날짜 범위 제약이 무시되어 최신 데이터가 필요한 에이전트가 2019년 블로그 포스트 같은 오래된 콘텐츠를 반환할 수 있음."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Exa API 대규모 변경: /research 엔드포인트 폐기, 날짜 필터 무음 무시 | ⭐⭐⭐ |
| Tip | 2026년 LLM 학습 로드맵: 채팅봇 대신 구조화된 출력 시스템 구축 | ⭐⭐⭐ |
| Trend | VLA(Vision Language Action) 모델로 AI 시장 이동 — 물리적 에이전트 시대 도래 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. Exa API 무음 파괴(Silent Breaking Change) — RAG 파이프라인 위험

**핵심:** 2026년 4월 15일부터 Exa의 `startCrawlDate`/`endCrawlDate` 필터가 요청을 받아도 실제로는 작동하지 않음. HTTP 200 응답을 반환하지만 날짜 범위 제약이 무시되어 최신 데이터가 필요한 에이전트가 2019년 블로그 포스트 같은 오래된 콘텐츠를 반환할 수 있음.

**공통 의견:** 이는 "조용한 파괴(silent breaking)"의 전형적 사례. 에러가 발생하지 않아 테스트가 통과하고 모니터링 알림도 울리지 않지만, 출력 품질이 점진적으로 악화됨. 개발자는 공식 문서나 변경 로그를 능동적으로 확인하지 않으면 발견 불가능.

**실무 적용:**

- RAG 파이프라인에서 Exa 사용 중이라면 즉시 `/search` 엔드포인트로 마이그레이션하고 `type: "deep-reasoning"` 파라미터 추가
- 날짜 필터 검증 로직 추가: 반환된 결과의 실제 크롤 날짜를 샘플링해서 예상 범위와 일치하는지 확인
- 외부 API 변경사항을 자동으로 감지하는 모니터링 설정 (응답 스키마 변화, 반환 필드 null 여부 등)

---

### 2. 2026년 LLM 학습 경로: 채팅봇 우상향 탈피

**핵심:** 단순 채팅 인터페이스 구축에서 벗어나 구조화된 출력(Structured Output) 시스템으로 전환. LLM을 프로덕션 애플리케이션에 통합하려면 프롬프팅, 임베딩, 벡터 DB, RAG, 파인튜닝이 어떻게 연결되는지 시스템 관점에서 이해해야 함.

**공통 의견:** 2026년 기준 LLM 학습의 핵심은 "도구 사용법"에서 "아키텍처 설계"로 이동. 단순히 API 호출하는 것이 아니라 에이전트, 메모리 관리, 도구 체이닝, 출력 검증을 포함한 전체 시스템을 설계할 수 있어야 함.

**실무 적용:**

- 프로젝트 시작 시 먼저 "이 LLM이 반환해야 할 정확한 데이터 구조"를 정의 (JSON 스키마 작성)
- 벡터 DB 선택 전에 검색 쿼리 패턴 분석 (키워드 검색 vs 의미 검색 vs 하이브리드)
- 파인튜닝 대신 프롬프트 엔지니어링과 Few-shot 예제로 시작해서 필요할 때만 파인튜닝 고려

---

### 3. VLA(Vision Language Action) 모델 — LLM에서 물리적 에이전트로의 진화

**핵심:** LLM(말하는 AI) + Vision + 센서 + 모션 플래닝 + 강화학습 + Edge GPU = 물리적 에이전트. 현대차, 엔비디아, 구글 등이 이 분야에 집중 투자 중. 시장의 자본 흐름이 순수 LLM에서 VLA로 이동 중.

**공통 의견:** 2026년 AI 시장의 분기점. 텍스트 기반 LLM 경쟁은 포화 상태이고, 실제 물리 세계에서 작동하는 에이전트가 새로운 가치 창출 영역. 로봇, 자율주행, 산업 자동화 분야에서 VLA 기술 보유 여부가 경쟁력 결정.

**실무 적용:**

- 현재 LLM 프로젝트가 있다면 "이것이 향후 비전/센서 입력을 받을 수 있도록 확장 가능한가?" 검토
- 엣지 GPU 최적화 기술 학습 (Expert Parallelism, MoE 모델 분산 처리)
- 강화학습 기초 복습: LLM 출력을 실제 행동으로 변환하는 피드백 루프 설계

---

## 🛠️ 지금 당장 해볼 것

- [ ] **Exa API 마이그레이션 체크** — 현재 코드에서 `/research` 또는 `startCrawlDate` 사용 여부 검색: `grep -r "startCrawlDate\|/research" .` 실행 후 `/search` + `type: "deep-reasoning"`으로 변경
- [ ] **LLM 프로젝트 구조화 출력 적용** — OpenAI의 Structured Output 가이드 확인: https://platform.openai.com/docs/guides/structured-outputs 에서 JSON 스키마 예제 다운로드 후 현재 프롬프트에 적용
- [ ] **LLMsPracticalGuide 저장소 클론** — https://github.com/Mooler0410/LLMsPracticalGuide 에서 최신 실무 가이드 다운로드 및 팀 내 공유 (특히 RAG, 파인튜닝 섹션)
- [ ] **벡터 DB 성능 테스트** — 현재 사용 중인 벡터 DB에서 날짜 필터 + 의미 검색 조합 쿼리 성능 측정 (응답 시간, 정확도) 및 로그 기록

---

## 🔗 참고 자료

- [Exa Just Removed /research and Started Silently Ignoring Two Date Filters — Your Agent Is Probably Pulling Stale Pages Right Now](https://dev.to/flarecanary/exa-just-removed-research-and-started-silently-ignoring-two-date-filters-your-agent-is-probably-1p51)
- [260502](https://blog.naver.com/haeun0814/224272271598)
- [26/5/1(금)#스톤이사#현대차로봇#엔비디아구글도러브콜...](https://blog.naver.com/goldbugsuk/224272331236)
- [How to Actually Learn LLMs in 2026 | Ex-Google, Microsoft Engineer](https://www.youtube.com/watch?v=U07MHi4Suj8)
- [Building LLM-Native Applications in 2026: A Practical Guide - LinkedIn](https://www.linkedin.com/pulse/building-llm-native-applications-2026-practical-guide-chetan-singh-otb6c)
- [GitHub - Mooler0410/LLMsPracticalGuide: A curated list of practical ...](https://github.com/Mooler0410/LLMsPracticalGuide)
- [LLM Roadmap 2026: How to Learn Large Language Models from ...](https://www.scaler.com/blog/llm-roadmap-2026-how-to-learn-large-language-models-from-scratch/)
- [Mastering Large Language Models in 2025: A Learning Path for Developers | Turing](https://www.turing.com/blog/mastering-large-language-models-learning-path-for-developers)

