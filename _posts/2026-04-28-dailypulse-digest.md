---
title: "[Daily Bigtech] 2026-04-28 국내 빅테크 오늘의 글"
author: gyuhwan
date: 2026-04-28 09:30:00 +0900
categories: [Daily Bigtech]
tags: [digest, auto, bigtech, daily]
description: "핵심:** 토스페이먼츠와 NIST가 양자컴퓨터 시대를 대비해 Post-Quantum Cryptography를 실제 서비스에 적용했다. 양자컴퓨터가 아직 위협이 아닌데도 10년 먼저 준비하는 이유는 \"지금 암호화된 데이터가 미래에 해독될 수 있다\"는 harvest now, decrypt later 공격 때문이다."
auto_generated: true
---

## 📋 daily_pulse — 이번 주 핵심 정보

> 수집 기간: 2026-04-28 기준 최근 7일

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | 양자컴퓨터 시대 대비 Post-Quantum Cryptography 실제 도입 사례 | ⭐⭐⭐ |
| Trend | AI 기반 데이터 분석 자동화 (판다, Privacy Filter) | ⭐⭐⭐ |
| Tip | 대규모 시스템 운영: 메트릭 저장소, 멀티테넌트 격리 | ⭐⭐ |
| New | GitHub Copilot 사용량 기반 청구 전환 (2026.6.1) | ⭐⭐ |

---

## 💡 Deep Dive

### 1. 보안 프로토콜 현대화: 양자내성암호의 현실적 도입

**핵심:** 토스페이먼츠와 NIST가 양자컴퓨터 시대를 대비해 Post-Quantum Cryptography를 실제 서비스에 적용했다. 양자컴퓨터가 아직 위협이 아닌데도 10년 먼저 준비하는 이유는 "지금 암호화된 데이터가 미래에 해독될 수 있다"는 harvest now, decrypt later 공격 때문이다.

**공통 의견:** 
- 토스페이먼츠: 수만 개 가맹점과의 연동 때문에 보안 업그레이드가 기술 문제가 아니라 조직 문제. 구형 서버 환경에서 돌아가는 레거시 시스템들을 동시에 업데이트해야 함
- NVIDIA의 NV-Raw2Insights-US: 기존 파이프라인을 버리고 원본 신호(raw data)부터 AI로 학습하는 방식으로 물리 법칙을 더 정확히 반영
- 공통점: 기술 혁신보다 **기존 시스템과의 호환성 유지**가 더 큰 과제

**실무 적용:**

- 보안 정책 변경 시 영향받는 모든 이해관계자(가맹점, 클라이언트)의 기술 수준을 먼저 파악하고, 단계적 마이그레이션 계획 수립
- 기술 문서만으로는 부족 — 비기술 담당자도 이해할 수 있는 비즈니스 언어로 변환 필요
- "지금 괜찮으니까 괜찮다"는 보안 판단은 위험 — 미래 위협(양자컴퓨터, 새로운 공격 벡터)을 현재 시점에서 대비

### 2. AI 기반 데이터 자동화: 단순 추출 작업 70% 자동화

**핵심:** 토스플레이스의 데이터봇 '판다(PANDA)'는 전체 데이터 요청의 70%가 복잡한 분석이 아니라 단순 수치 확인이라는 발견에서 출발했다. 단순히 LLM을 붙이는 것이 아니라 데이터 마트 표준화, 비즈니스 언어 정의, Scoring & Ranking 시스템으로 AI의 일관성을 확보했다.

**공통 의견:**
- 판다: 데이터 정확성을 위해 SSOT(Single Source of Truth) 정비에 가장 많은 시간 투자. 테이블 네이밍 규칙, Description 작성 등 메타데이터 정리가 AI 성능을 좌우
- OpenAI Privacy Filter: 원본 신호(raw data)에서 직접 학습하는 방식으로 128k 토큰 컨텍스트에서 한 번에 처리 가능
- 공통점: **데이터 품질 > 모델 성능**. 깔끔한 데이터 구조가 AI 신뢰도를 결정

**실무 적용:**

- 데이터 자동화 프로젝트 시작 전에 데이터 거버넌스(표준화, 네이밍 규칙, 메타데이터) 먼저 정비
- 비즈니스 팀과 협업해 도메인 용어 사전 구축 — AI가 "설치 매장"과 "활성 매장"을 구분하도록
- AI 모델 선택보다 **데이터 파이프라인 설계**에 더 많은 리소스 할당

### 3. 대규모 시스템 운영: 멀티테넌트 격리와 메트릭 저장소 확장

**핵심:** 토스의 StarRocks와 네이버의 VictoriaMetrics 운영 사례는 같은 클러스터에서 서로 다른 워크로드를 격리하는 방법을 보여준다. StarRocks는 Resource Group으로 CPU 우선순위를 설계했고, VictoriaMetrics는 12.5억 개 시계열을 0.92바이트/데이터포인트로 압축하면서 180대 장비를 무중단으로 교체했다.

**공통 의견:**
- 멀티테넌트 환경에서 "누구의 쿼리를 먼저 보호할 것인가"가 핵심 질문. 서비스 쿼리 > 배치 > 모니터링 순으로 우선순위 설정
- 메트릭 저장소는 단순 저장이 아니라 **카디널리티 관리**가 생명. 쿠버네티스 환경에서 컨테이너 58배 증가 시 메트릭도 기하급수적 증가
- 무중단 장비 교체는 기술보다 **운영 절차와 모니터링**이 중요

**실무 적용:**

- 리소스 격리 설정 시 의존 관계를 명확히 — 한 파라미터가 클러스터 전체 성능을 좌우할 수 있음
- 메트릭 수집 시 레이블 조합(pod name, namespace 등)을 제한해 카디널리티 폭증 방지
- 대규모 인프라 변경 시 미리 preview 환경에서 비용 시뮬레이션 (GitHub Copilot의 early May preview bill 사례)

### 4. 장기 컨텍스트 AI 모델의 실제 활용: DeepSeek-V4의 에이전트 설계

**핵심:** DeepSeek-V4는 1M 토큰 컨텍스트를 제공하지만, 단순히 크기가 아니라 **에이전트가 실제로 사용할 수 있는 효율성**을 설계했다. 단일 토큰 추론 시 V3 대비 27% FLOPs, KV 캐시 10% 수준으로 줄여서 장시간 도구 사용 세션을 가능하게 했다.

**공통 의견:**
- 컨텍스트 크기 > 추론 효율성. 1M 토큰을 로드하는 것과 실제로 사용하는 것은 다름
- Compressed Sparse Attention으로 KV 캐시를 2%까지 압축 — 메모리 제약이 있는 환경에서 실용적
- 에이전트 워크로드(SWE-bench, 브라우징, 터미널 세션)는 매 도구 호출마다 컨텍스트가 누적되므로 효율성이 생명

**실무 적용:**

- 장기 컨텍스트 모델 도입 시 단순 토큰 수가 아니라 **추론 비용(FLOPs, 메모리)**을 먼저 계산
- 에이전트 시스템 구축 시 도구 호출 결과를 컨텍스트에 누적하는 방식 설계 — 중간에 컨텍스트 초과로 실패하지 않도록
- 모델 선택 기준: 크기 > 효율성 > 비용

---

## 🛠️ 지금 당장 해볼 것

- [ ] **Post-Quantum Cryptography 준비 상태 점검** — 현재 서비스의 TLS 버전, 암호화 알고리즘 확인. `openssl s_client -connect your-domain.com:443` 실행해 현재 cipher suite 확인
  
- [ ] **데이터 마트 메타데이터 정리 시작** — 팀의 주요 테이블 5개 선정해 Description, 네이밍 규칙 문서화. 스프레드시트나 Notion에 "테이블명 | 용도 | 주요 컬럼 | 업데이트 주기" 작성

- [ ] **GitHub Copilot 사용량 미리 확인** — 2026년 6월 1일 전환 전에 Settings > Billing > Copilot 페이스에서 현재 사용 패턴 분석. 조직 규모별 예상 비용 계산 (site:github.com copilot usage-based billing calculator)

- [ ] **VictoriaMetrics 또는 Prometheus 메트릭 카디널리티 측정** — 현재 운영 중인 모니터링 시스템에서 활성 시계열 수 확인. `promtool query instant 'count(count by (__name__) ({__name__=~".+"}))'` 실행

- [ ] **DeepSeek-V4 또는 유사 장기 컨텍스트 모델 테스트** — Hugging Face에서 `deepseek-ai/DeepSeek-V4` 모델 카드 확인하고, 로컬 환경에서 간단한 에이전트 워크플로우 테스트 (site:huggingface.co deepseek-v4 inference)

---

## 🔗 원본 출처 (클릭하여 원문 확인)
- [Adaptive Ultrasound Imaging with Physics-Informed NV-Raw2Insights-US AI](https://huggingface.co/blog/nvidia/raw2insights-adaptive-ultrasound-imaging)
- [GitHub Copilot is moving to usage-based billing](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)
- [Why We Adopted Post-Quantum Cryptography a Decade Before Quantum Computers Arrive](https://toss.tech/article/post-quantum-cryptography-eng)
- [How to build scalable web apps with OpenAI's Privacy Filter](https://huggingface.co/blog/openai-privacy-filter-web-apps)
- [DeepSeek-V4: a million-token context that agents can actually use](https://huggingface.co/blog/deepseekv4)
- [StarRocks 운영기: Resource Group으로 멀티테넌트 워크로드 격리하기](https://toss.tech/article/operating-starrocks-1)
- [토스플레이스 데이터봇 ‘판다(PANDA)’를 소개합니다 : 모든 팀원이 데이터 전문가처럼 일하는 방법](https://toss.tech/article/da-assistant-panda)
- [양자컴퓨터 시대에 대비한 양자내성암호 적용, 왜 10년 먼저 서비스에 적용했을까?](https://toss.tech/article/post-quantum-cryptography)
- [How to Use Transformers.js in a Chrome Extension](https://huggingface.co/blog/transformersjs-chrome-extension)
- [네이버 검색의 대규모 메트릭 저장소, VictoriaMetrics 운영기](https://d2.naver.com/helloworld/6475419)
- [Making Rust Workers reliable: panic and abort recovery in wasm‑bindgen](https://blog.cloudflare.com/making-rust-workers-reliable/)
- [Building a fault-tolerant metrics storage system at Airbnb](https://medium.com/airbnb-engineering/building-a-fault-tolerant-metrics-storage-system-at-airbnb-26a01a6e7017?source=rss----53c7c27702d5---4)
- [Moving past bots vs. humans](https://blog.cloudflare.com/past-bots-and-humans/)
