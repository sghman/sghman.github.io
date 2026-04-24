---
title: "[Daily Bigtech] 2026-04-24 국내 빅테크 오늘의 글"
author: gyuhwan
date: 2026-04-24 09:30:00 +0900
categories: [Daily Bigtech]
tags: [digest, auto, bigtech, daily]
description: "핵심:** 무신사의 AI 네이티브 채용 3부작 최종편에서 드러난 것은 머신 러닝 모델이 아무리 정교해도 사람의 사고 과정 자체를 직접 확인할 수 없다는 근본적 한계입니다. Functional Gate와 Depth 점수는 결과물만 평가하므로, AI가 생성한 코드인지 지원자가 이해하고 작성한 코드인지 구분 불가능합니다."
auto_generated: true
---

## 📋 daily_pulse — 이번 주 핵심 정보

> 수집 기간: 2026-04-24 기준 최근 7일

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | AI 네이티브 채용 시스템에서 점수 너머의 인간 판단의 중요성 | ⭐⭐⭐ |
| Trend | 대규모 메트릭 저장소와 시계열 데이터베이스 운영 사례 증가 | ⭐⭐⭐ |
| Tip | 양자컴퓨터 시대 대비 암호화 기술 선제적 도입 전략 | ⭐⭐ |
| Trend | 에이전트 AI 시스템의 대규모 배포와 인프라 혁신 | ⭐⭐⭐ |

---

## 💡 Deep Dive

### 1. AI 채용 시스템의 한계: 점수가 놓치는 것들

**핵심:** 무신사의 AI 네이티브 채용 3부작 최종편에서 드러난 것은 머신 러닝 모델이 아무리 정교해도 사람의 사고 과정 자체를 직접 확인할 수 없다는 근본적 한계입니다. Functional Gate와 Depth 점수는 결과물만 평가하므로, AI가 생성한 코드인지 지원자가 이해하고 작성한 코드인지 구분 불가능합니다.

**공통 의견:** 토스플레이스의 판다(PANDA) 데이터봇 사례와 맥락을 같이합니다. 데이터 추출 요청의 70%가 단순 조회였듯이, 채용 평가도 자동화 가능한 부분(기능 검증)과 인간 판단이 필수인 부분(의도와 사고 과정)을 명확히 분리해야 합니다.

**실무 적용:**

- 코드 증거 기반 맞춤 면접 질문 생성: 지원자의 제출 코드에서 구체적 구현 선택지를 추출해 "왜 이중 락 전략을 선택했는가"처럼 사고 과정을 직접 묻기
- 재현성 검증 단계 추가: 한 번의 과제 결과로 역량을 판단하지 말고, 면접에서 유사한 문제를 실시간으로 풀게 해 일관성 확인
- 머신 필터 + 휴먼 검증 투 스테이지 구조: 점수는 후보자 풀을 좁히는 도구일 뿐, 최종 판단은 면접관의 직관과 경험에 의존

---

### 2. 대규모 시계열 데이터 운영의 현실: 메모리와 카디널리티 폭발

**핵심:** 네이버 검색의 VictoriaMetrics 운영 사례와 Airbnb의 메트릭 저장소 구축 사례 모두 같은 문제를 마주쳤습니다. 쿠버네티스 전환으로 컨테이너가 58배 증가하면서 메트릭 카디널리티가 기하급수적으로 늘어났고, 단순 수집량 증가가 아닌 레이블 조합 폭발이 핵심 병목이 되었습니다.

**공통 의견:** 토스의 StarRocks Resource Group 격리 전략과 동일한 맥락입니다. 워크로드 분류 → CPU 우선순위 설계 → 배타적 격리라는 3단계 접근이 메트릭 저장소에도 필요합니다. 네이버는 180대 장비 무중단 전환을 성공시켰고, Airbnb는 50M samples/sec를 처리하는 시스템을 구축했습니다.

**실무 적용:**

- 카디널리티 폭발 사전 차단: 레이블 설계 단계에서 불필요한 고유값 조합 제거 (예: 임시 컨테이너 ID를 레이블로 사용하지 않기)
- 테넌트 격리 아키텍처: 읽기/쓰기 경로 분리, 팀별 또는 서비스별 독립 저장소 고려로 한 팀의 쿼리 폭증이 전체 시스템에 영향 주지 않도록
- 무중단 장비 전환 자동화: 쿠버네티스 환경에서는 Pod 재배치 + 데이터 마이그레이션을 병렬화해 서비스 중단 없이 인프라 업그레이드

---

### 3. 양자컴퓨터 시대 대비: 10년 먼저 움직이는 보안 전략

**핵심:** 토스페이먼츠가 양자내성암호(Post-Quantum Cryptography)를 지금 도입하는 이유는 "지금 괜찮으니까 괜찮다"가 통하지 않는 보안 영역의 특수성 때문입니다. 수만 개 가맹점과의 연동 때문에 한 번 변경하면 되돌리기 어려우므로, 위협이 현실화되기 10년 전부터 준비해야 합니다.

**공통 의견:** Cloudflare의 "bots vs. humans" 구분 폐기 논의와 맥락이 다르지만, 둘 다 현재의 관성을 깨고 미래 환경에 맞춰 시스템을 재설계하는 선제적 사고를 강조합니다. GitHub Copilot의 agentic workflow 리소스 폭증도 같은 맥락: 지금의 한계를 인정하고 구조를 바꿔야 합니다.

**실무 적용:**

- 레거시 시스템의 보안 프로토콜 점진적 업그레이드: 한 번에 모든 가맹점을 강제하지 말고, 신규 연동부터 적용 후 기존 고객에게 인센티브 제공
- 기술 문서의 비즈니스 언어화: 보안 용어로 가득한 공지가 아닌, 각 가맹점의 기술 수준에 맞춘 단계별 가이드 제공
- 변경 영향도 사전 시뮬레이션: 수만 개 엔드포인트가 있을 때는 카나리 배포나 A/B 테스트로 리스크 최소화

---

### 4. 에이전트 AI의 인프라 혁신: 클라우드 2.0의 시작

**핵심:** Cloudflare의 Agents Week 발표와 내부 AI 엔지니어링 스택 공개는 에이전트 AI가 단순한 모델 개선이 아닌 인프라 패러다임 전환을 요구한다는 점을 명확히 합니다. 기존 "한 앱이 많은 사용자를 서빙"하는 모델에서 "많은 에이전트가 병렬로 장시간 실행"되는 모델로의 전환입니다.

**공통 의견:** GitHub Copilot의 Pro/Pro+ 플랜 재구조화와 사용량 제한 강화도 같은 현상입니다. agentic workflow가 기존 예상의 5배 이상 리소스를 소비하면서, 단순 모델 업그레이드로는 해결 불가능하고 인프라 자체를 재설계해야 합니다. Cloudflare는 Workers를 에이전트 중심으로 재구성했고, 내부적으로 93% R&D 조직이 AI 코딩 도구를 사용하며 merge request가 2배 증가했습니다.

**실무 적용:**

- MCP(Model Context Protocol) 서버 내부화: 공개 API 대신 조직 내 데이터와 도구에 직접 접근하는 에이전트 구축으로 응답 속도와 정확도 향상
- 에이전트 세션 격리 및 리소스 쿼터: 한 에이전트의 폭주가 다른 요청을 방해하지 않도록 컨테이너/isolate 단위 리소스 제한
- 코드 리뷰 자동화 재정의: 에이전트가 생성한 코드의 품질 검증을 위해 정적 분석 + 자동 테스트 + 휴먼 리뷰의 3단계 파이프라인 구축

---

## 🛠️ 지금 당장 해볼 것

- [ ] **AI 채용 평가 시스템 감사** — 현재 사용 중인 코딩 테스트 플랫폼에서 "점수만 본다"는 함정이 있는지 확인. 무신사 Part 3 글 읽고 면접 질문 템플릿 재작성: `site:techblog.musinsa.com "The Human"`

- [ ] **메트릭 카디널리티 현황 파악** — Prometheus 또는 VictoriaMetrics 사용 중이면 `topk(20, count by (__name__) ({__name__=~".+"}))` 쿼리로 상위 20개 메트릭의 시계열 수 확인. 네이버 사례 참고: `site:d2.naver.com VictoriaMetrics`

- [ ] **양자내성암호 로드맵 수립** — 조직의 암호화 기술 현황 문서화 (TLS 버전, Cipher Suite, 인증서 갱신 주기). 토스페이먼츠 사례 검토: `site:toss.tech "양자내성암호"`

- [ ] **Cloudflare Workers AI 또는 로컬 에이전트 프로토타입** — Gemma 4 VLA 데모 실행해보기 (Jetson Orin Nano 없으면 로컬 CPU로도 가능). GitHub 저장소: `https://github.com/asierarranz/Google_Gemma/tree/main/Gemma4`

---

## 🔗 원본 출처 (클릭하여 원문 확인)
- [The Human: 점수 너머의 판단](https://techblog.musinsa.com/the-human-%EC%A0%90%EC%88%98-%EB%84%88%EB%A8%B8%EC%9D%98-%ED%8C%90%EB%8B%A8-bccc190c9c93?source=rss----f107b03c406e---4)
- [StarRocks 운영기: Resource Group으로 멀티테넌트 워크로드 격리하기](https://toss.tech/article/operating-starrocks-1)
- [토스플레이스 데이터봇 ‘판다(PANDA)’를 소개합니다 : 모든 팀원이 데이터 전문가처럼 일하는 방법](https://toss.tech/article/da-assistant-panda)
- [양자컴퓨터 시대에 대비한 양자내성암호 적용, 왜 10년 먼저 서비스에 적용했을까?](https://toss.tech/article/post-quantum-cryptography)
- [네이버 검색의 대규모 메트릭 저장소, VictoriaMetrics 운영기](https://d2.naver.com/helloworld/6475419)
- [Gemma 4 VLA Demo on Jetson Orin Nano Super](https://huggingface.co/blog/nvidia/gemma4)
- [Making Rust Workers reliable: panic and abort recovery in wasm‑bindgen](https://blog.cloudflare.com/making-rust-workers-reliable/)
- [Building a fault-tolerant metrics storage system at Airbnb](https://medium.com/airbnb-engineering/building-a-fault-tolerant-metrics-storage-system-at-airbnb-26a01a6e7017?source=rss----53c7c27702d5---4)
- [Moving past bots vs. humans](https://blog.cloudflare.com/past-bots-and-humans/)
- [AI and the Future of Cybersecurity: Why Openness Matters](https://huggingface.co/blog/cybersecurity-openness)
- [Changes to GitHub Copilot Individual plans](https://github.blog/news-insights/company-news/changes-to-github-copilot-individual-plans/)
- [Highlights from Git 2.54](https://github.blog/open-source/git/highlights-from-git-2-54/)
- [Building the agentic cloud: everything we launched during Agents Week 2026](https://blog.cloudflare.com/agents-week-in-review/)
- [The AI engineering stack we built internally — on the platform we ship](https://blog.cloudflare.com/internal-ai-engineering-stack/)
