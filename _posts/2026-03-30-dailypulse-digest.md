---
title: "[Daily Bigtech] 2026-03-30 국내 빅테크 오늘의 글"
author: gyuhwan
date: 2026-03-30 09:30:00 +0900
categories: [Daily Bigtech]
tags: [digest, auto, bigtech, daily]
description: "핵심:** 무신사의 AI 네이티브 신입 채용 프로세스는 전통적인 알고리즘 문제 풀이 대신 모호한 요구사항에서 스스로 문제를 정의하고 해결하는 능력을 평가했습니다. 이는 AI가 구현을 자동화하는 시대에 엔지니어의 역할이 \"무엇을 만들지\" 결정하는 쪽으로 이동했음을 반영합니다."
auto_generated: true
---

## 📋 daily_pulse — 이번 주 핵심 정보

> 수집 기간: 2026-03-30 기준 최근 7일

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | AI 네이티브 신입 채용 프로세스 혁신 (무신사 루키즈) | ⭐⭐⭐ |
| New | GitHub Actions 2026 보안 로드맵 공개 | ⭐⭐⭐ |
| Trend | 오픈소스 취약점 2025년 분석 리포트 | ⭐⭐ |
| Tip | C++ 타입 캐스팅 실무 가이드 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. AI 시대의 채용 기준 변화: "구현"에서 "문제 정의"로

**핵심:** 무신사의 AI 네이티브 신입 채용 프로세스는 전통적인 알고리즘 문제 풀이 대신 모호한 요구사항에서 스스로 문제를 정의하고 해결하는 능력을 평가했습니다. 이는 AI가 구현을 자동화하는 시대에 엔지니어의 역할이 "무엇을 만들지" 결정하는 쪽으로 이동했음을 반영합니다.

**공통 의견:** 여러 기업의 채용 공고와 기술 블로그에서 일관되게 나타나는 패턴은 AI 에이전트 시대에 필요한 역량이 알고리즘 최적화가 아닌 요구사항 분석, 시스템 설계, 협업 능력으로 재편되고 있다는 점입니다. 네이버 AI Challenge 인턴십도 실무 프로젝트 협업을 강조하고 있습니다.

**실무 적용:**

- 코딩 테스트 준비 시 LeetCode 스타일 문제보다 **설계 문서 작성, API 명세 정의, 엣지 케이스 발굴** 연습에 시간 투자
- 팀 인터뷰 대비 시 "이 요구사항의 문제점은 무엇인가", "어떤 정보가 부족한가" 같은 질문에 답하는 연습
- 프로젝트 포트폴리오에 **단순 구현 결과보다 설계 의사결정 과정과 트레이드오프 분석** 포함

### 2. CI/CD 보안의 새로운 표준: 공급망 공격 방어

**핵심:** GitHub Actions 2026 보안 로드맵은 의존성 결정론화(deterministic dependencies), 정책 기반 접근 제어, 네트워크 격리를 통해 CI/CD 파이프라인을 공급망 공격으로부터 보호하는 데 초점을 맞추고 있습니다. 현재 워크플로우 의존성이 런타임에 해석되면서 mutable 태그 참조로 인한 취약점이 발생하고 있습니다.

**공통 의견:** Cloudflare, GitHub 등 인프라 기업들이 일관되게 강조하는 것은 "보안을 기본값으로 만들기"입니다. 과거 사후 대응식 보안에서 벗어나 정책 기반 강제(policy enforcement)와 실시간 관찰성(observability)으로 전환하는 중입니다.

**실무 적용:**

- GitHub Actions 워크플로우에서 **모든 action 참조를 commit SHA로 고정** (tag 대신 `uses: actions/checkout@abc123def456`)
- 의존성 관리 도구(Dependabot, Renovate) 설정 시 **자동 머지 대신 수동 검토 프로세스** 유지
- 조직 정책으로 **외부 action 사용 제한 및 승인 프로세스** 구축 (GitHub Advanced Security 활용)

### 3. 오픈소스 생태계의 보안 현황: 새로운 취약점 유형의 부상

**핵심:** 2025년 GitHub Advisory Database 분석 결과, 새로 보고된 취약점은 전년 대비 19% 증가했으며, Go 생태계에서 특히 높은 비율의 취약점이 발견되었습니다. 전통적인 XSS(CWE-79)는 여전히 가장 많지만, 인프라 코드와 AI 에이전트 관련 새로운 취약점 유형이 증가하는 추세입니다.

**공통 의견:** 오픈소스 보안 위협이 단순 코드 결함에서 **공급망 자체의 신뢰성 문제**로 진화하고 있습니다. 개별 패키지 보안보다 의존성 체인 전체의 투명성과 추적 가능성이 중요해졌습니다.

**실무 적용:**

- 프로젝트 의존성 정기 감사 시 **Dependabot alerts 외에 SBOM(Software Bill of Materials) 생성 및 검토** 추가
- Go 프로젝트의 경우 특히 **go.mod 파일의 indirect 의존성 검토** 강화
- 내부 보안 정책에 **transitive dependency 추적 및 제한** 규칙 포함

---

## 🛠️ 지금 당장 해볼 것

- [ ] GitHub 저장소의 모든 action 참조를 commit SHA로 변환 — `grep -r "uses:" .github/workflows/ | grep -v "@"` 실행 후 [GitHub Actions 공식 문서](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsuses)의 "Using commit SHAs" 섹션 참고

- [ ] 현재 프로젝트의 의존성 트리 시각화 — Node.js는 `npm ls`, Python은 `pip show <package>`, Go는 `go mod graph` 실행하여 transitive dependency 확인

- [ ] Dependabot 설정 활성화 (미설정 시) — GitHub 저장소 Settings → Code security and analysis → Dependabot alerts 활성화 후 [Dependabot 설정 가이드](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts) 확인

- [ ] 팀의 코딩 테스트 평가 기준 검토 — 알고리즘 문제 비중을 줄이고 "요구사항이 불명확할 때 어떻게 대응하는가"를 평가하는 설계 문제 추가 (무신사 루키즈 사례 참고)

---

## 🔗 원본 출처 (클릭하여 원문 확인)
- [To. MUSINSA ROOKIES](https://techblog.musinsa.com/to-musinsa-rookies-51e6e6fa5ee8?source=rss----f107b03c406e---4)
- [How we use Abstract Syntax Trees (ASTs) to turn Workflows code into visual diagrams](https://blog.cloudflare.com/workflow-diagrams/)
- [Liberate your OpenClaw](https://huggingface.co/blog/liberate-your-openclaw)
- [C++ 객체 수명과 암묵적 객체 생성](https://d2.naver.com/helloworld/7997284)
- [[인턴십] 2026 NAVER AI CHALLENGE를 소개합니다.](https://d2.naver.com/news/7477295)
- [What’s coming to our GitHub Actions 2026 security roadmap](https://github.blog/news-insights/product-news/whats-coming-to-our-github-actions-2026-security-roadmap/)
- [A year of open source vulnerability trends: CVEs, advisories, and malware](https://github.blog/security/supply-chain-security/a-year-of-open-source-vulnerability-trends-cves-advisories-and-malware/)
- [A one-line Kubernetes fix that saved 600 hours a year](https://blog.cloudflare.com/one-line-kubernetes-fix-saved-600-hours-a-year/)
- [Updates to GitHub Copilot interaction data usage policy](https://github.blog/news-insights/company-news/updates-to-github-copilot-interaction-data-usage-policy/)
- [C++ std::bit_cast와 reinterpret_cast — 언제 어떤 것을 써야 하는가](https://d2.naver.com/helloworld/1155434)
- [Metric Review, 실행을 이끌다](https://toss.tech/article/place-metric-review)
- [What COVID did to our forecasting models (and what we built to handle the next shock)](https://medium.com/airbnb-engineering/what-covid-did-to-our-forecasting-models-and-what-we-built-to-handle-the-next-shock-ccbf0e1f7fa9?source=rss----53c7c27702d5---4)
- [Building AI-powered GitHub issue triage with the Copilot SDK](https://github.blog/ai-and-ml/github-copilot/building-ai-powered-github-issue-triage-with-the-copilot-sdk/)
- [Sandboxing AI agents, 100x faster](https://blog.cloudflare.com/dynamic-workers/)
