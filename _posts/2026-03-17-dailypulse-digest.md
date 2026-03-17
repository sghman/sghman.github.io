---
title: "[Daily Bigtech] 2026-03-17 국내 빅테크 오늘의 글"
author: gyuhwan
date: 2026-03-17 09:30:00 +0900
categories: [Daily Bigtech]
tags: [digest, auto, bigtech, daily]
description: "핵심:** Andrej Karpathy가 정의한 Software 3.0 시대에서는 명령형 코드(1.0)와 신경망 가중치(2.0)를 넘어, 자연어 프롬프트 자체가 프로그램이 된다. 하지만 LLM의 원시 능력만으로는 실제 업무를 처리할 수 없다. 코드베이스 읽기, 명령 실행, 데이터베이스 접근 같은 작업을 가능하게 하는 '하네스(harness)'가 필수다."
auto_generated: true
---

## 📋 daily_pulse — 이번 주 핵심 정보

> 수집 기간: 2026-03-17 기준 최근 7일

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Software 3.0 시대: LLM이 프로그래밍의 새로운 패러다임 | ⭐⭐⭐ |
| New | 헬스케어 로봇 공개 데이터셋 Open-H-Embodiment 출시 | ⭐⭐⭐ |
| Trend | AI 에이전트 시대의 실행 중심 인터페이스 전환 | ⭐⭐⭐ |
| Tip | GitHub Actions로 CI/CD 자동화 시작하기 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. Software 3.0: 프롬프트가 곧 프로그램이 되다

**핵심:** Andrej Karpathy가 정의한 Software 3.0 시대에서는 명령형 코드(1.0)와 신경망 가중치(2.0)를 넘어, 자연어 프롬프트 자체가 프로그램이 된다. 하지만 LLM의 원시 능력만으로는 실제 업무를 처리할 수 없다. 코드베이스 읽기, 명령 실행, 데이터베이스 접근 같은 작업을 가능하게 하는 '하네스(harness)'가 필수다.

**공통 의견:** Claude Code, GitHub Copilot SDK 같은 도구들이 LLM을 실제 에이전트로 변환하는 하네스 역할을 한다. 이들은 단순히 텍스트 생성을 넘어 파일 수정, 명령 실행, 에러 복구 같은 다단계 작업을 자동으로 처리한다.

**실무 적용:**

- LLM 기반 자동화 도구 도입 시 단순 텍스트 출력이 아닌 '실행 가능한 에이전트'로 설계하기
- 기존 스크립트 기반 자동화를 LLM 에이전트로 마이그레이션하여 컨텍스트 변화에 대응하는 유연성 확보
- 에이전트가 작동할 수 있는 환경(API 접근, 파일 시스템, 데이터베이스)을 먼저 구축한 후 프롬프트 작성

### 2. AI 에이전트 시대의 실행 중심 인터페이스

**핵심:** "AI as text" 시대가 끝났다. 과거 2년간 AI는 텍스트 입력→텍스트 출력의 단순 교환 방식이었지만, 이제 프로덕션 환경에서는 계획 수립, 도구 호출, 파일 수정, 에러 복구를 자동으로 수행하는 실행 계층이 필수다. GitHub Copilot SDK는 이 실행 엔진을 애플리케이션에 직접 임베드할 수 있게 한다.

**공통 의견:** 다중 벡터 공격 탐지, 접근성 피드백 자동화, 여행지 추천 같은 복잡한 워크플로우들이 에이전트 기반으로 재설계되고 있다. 단순 의도 전달로 다단계 작업이 자동 처리되는 구조로 전환 중.

**실무 적용:**

- 반복적인 멀티스텝 작업(배포, 테스트, 데이터 처리)을 에이전트에 위임하여 스크립트 유지보수 비용 절감
- 에이전트 실행 범위를 명확히 정의하고 제약 조건(권한, 리소스 한도)을 설정하여 안전성 확보
- 기존 자동화 도구의 오케스트레이션 레이어를 Copilot SDK 같은 프로덕션 검증된 엔진으로 교체

### 3. 보안 인프라의 통합화: 다중 벡터 공격 대응

**핵심:** 현대 공격은 단일 벡터가 아니다. API 프로빙, 네트워크 플러딩, 탈취 자격증명 사용이 동시에 일어난다. Cloudflare Log Explorer는 HTTP 요청, DDoS/방화벽 로그, Zero Trust 접근 이벤트를 통합하여 360도 가시성을 제공한다. 또한 RFC 9457 표준 에러 응답으로 AI 에이전트의 토큰 비용을 98% 절감할 수 있다.

**공통 의견:** 보안 팀의 가장 큰 문제는 데이터 부족이 아니라 과잉이다. 수백 개 대시보드를 오가며 "지금 뭘 해야 하나?"를 찾는 것이 현실. Security Action Items 같은 우선순위 기반 필터링이 필수.

**실무 적용:**

- 로그 수집 도구를 단일 인터페이스로 통합하여 MTTD(평균 탐지 시간) 단축
- AI 에이전트 기반 자동화 시 구조화된 에러 응답(JSON/Markdown) 활용으로 토큰 비용 최적화
- 보안 대시보드를 "모든 것 표시"에서 "지금 해야 할 것"으로 재설계

### 4. 헬스케어 로봇의 물리적 AI 기초 구축

**핵심:** 헬스케어 AI는 지금까지 인식(perception) 중심이었다. 하지만 실제 의료는 "행동"이다. Open-H-Embodiment는 35개 기관이 협력하여 778시간의 수술 로봇 데이터를 공개했다. 이는 시뮬레이션-현실 페어링, 교차 로봇 벤치마크, 폐쇄 루프 제어를 포함한 첫 대규모 물리적 AI 데이터셋이다.

**공통 의견:** 물리적 AI는 단순 모델 학습이 아니라 표준화된 로봇 바디, 동기화된 비전-힘-운동학 데이터, 실제 환경 적응이 필요하다. 이는 자율주행, 제조 로봇 등 다른 도메인에도 적용되는 패턴.

**실무 적용:**

- 로봇/자동화 프로젝트 시작 시 공개 데이터셋(Open-H-Embodiment 같은) 활용으로 초기 학습 데이터 확보
- 폐쇄 루프 제어와 실시간 피드백을 포함한 데이터 수집 설계
- 시뮬레이션 환경에서 학습한 모델을 실제 환경에 적응시키는 sim-to-real 파이프라인 구축

---

## 🛠️ 지금 당장 해볼 것

- [ ] **GitHub Actions 첫 워크플로우 만들기** — 5분 안에 시작 가능. `.github/workflows/` 디렉토리에 `hello.yml` 파일 생성 후 `on: push` 트리거와 간단한 `run: echo "Hello"` 스텝 추가. [GitHub Actions 공식 가이드](https://github.blog/developer-skills/github/github-for-beginners-getting-started-with-github-actions/)

- [ ] **Cloudflare Log Explorer 접근 권한 확인** — 조직의 보안 팀과 함께 현재 사용 중인 로그 수집 도구 목록 작성 후, Cloudflare 대시보드에서 Log Explorer 활성화 여부 확인. [Cloudflare Log Explorer 문서](https://blog.cloudflare.com/investigating-multi-vector-attacks-in-log-explorer/)

- [ ] **GitHub Copilot SDK 데모 코드 실행** — 로컬 환경에서 `npm install @github/copilot-sdk` 실행 후, 간단한 에이전트 위임 패턴(멀티스텝 작업) 테스트. [GitHub Copilot SDK 저장소](https://github.com/github/copilot-sdk)

- [ ] **RFC 9457 에러 응답 테스트** — curl로 Cloudflare 보호 도메인에 `Accept: application/json` 헤더를 포함한 요청 전송 후, 구조화된 JSON 에러 응답 확인. `curl -H "Accept: application/json" https://your-domain.com/blocked-path`

---

## 🔗 원본 출처 (클릭하여 원문 확인)
- [Embracing the Software 3.0 Era](https://toss.tech/article/harness-for-team-productivity-eng)
- [The First Healthcare Robotics Dataset and Foundational Physical AI Models for Healthcare Robotics](https://huggingface.co/blog/nvidia/physical-ai-for-healthcare-robotics)
- [Standing up for the open Internet: why we appealed Italy’s "Piracy Shield" fine](https://blog.cloudflare.com/standing-up-for-the-open-internet/)
- [GitHub for Beginners: Getting started with GitHub Actions](https://github.blog/developer-skills/github/github-for-beginners-getting-started-with-github-actions/)
- [From legacy architecture to Cloudflare One](https://blog.cloudflare.com/legacy-to-agile-sase/)
- [Recommending Travel Destinations to Help Users Explore](https://medium.com/airbnb-engineering/recommending-travel-destinations-to-help-users-explore-5fa7a81654fb?source=rss----53c7c27702d5---4)
- [Continuous AI for accessibility: How GitHub transforms feedback into inclusion](https://github.blog/ai-and-ml/github-copilot/continuous-ai-for-accessibility-how-github-transforms-feedback-into-inclusion/)
- [Announcing Cloudflare Account Abuse Protection: prevent fraudulent attacks from bots and humans](https://blog.cloudflare.com/account-abuse-protection/)
- [GitHub availability report: February 2026](https://github.blog/news-insights/company-news/github-availability-report-february-2026/)
- [Addressing GitHub’s recent availability issues](https://github.blog/news-insights/company-news/addressing-githubs-recent-availability-issues-2/)
- [Slashing agent token costs by 98% with RFC 9457-compliant error responses](https://blog.cloudflare.com/rfc-9457-agent-error-pages/)
- [AI Security for Apps is now generally available](https://blog.cloudflare.com/ai-security-for-apps-ga/)
- [The era of “AI as text” is over. Execution is the new interface.](https://github.blog/ai-and-ml/github-copilot/the-era-of-ai-as-text-is-over-execution-is-the-new-interface/)
- [Investigating multi-vector attacks in Log Explorer](https://blog.cloudflare.com/investigating-multi-vector-attacks-in-log-explorer/)
- [Building a security overview dashboard for actionable insights](https://blog.cloudflare.com/security-overview-dashboard/)
