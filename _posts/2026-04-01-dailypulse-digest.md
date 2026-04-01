---
title: "[Daily Bigtech] 2026-04-01 국내 빅테크 오늘의 글"
author: gyuhwan
date: 2026-04-01 09:30:00 +0900
categories: [Daily Bigtech]
tags: [digest, auto, bigtech, daily]
description: "핵심:** es-toolkit이 lodash를 대체하는 현실이 되었다. 번들 크기 97% 감소, 성능 2배 향상이라는 수치가 단순 마케팅이 아니라 ES Modules와 TypeScript 네이티브 설계의 결과다. Toss가 \"한국에서 글로벌 JavaScript 라이브러리가 나올 수 있을까\"라는 질문에서 시작한 프로젝트가 Microsoft, Yarn, Storybook 같은 기업들에 채택되는 것은 기술 부채 해결의 신호다."
auto_generated: true
---

## 📋 daily_pulse — 이번 주 핵심 정보

> 수집 기간: 2026-04-01 기준 최근 7일

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | es-toolkit 1000만 주간 다운로드 달성, lodash 대체 라이브러리로 부상 | ⭐⭐⭐ |
| Trend | AI 에이전트 기반 개발 패러다임 확산, 코딩 테스트도 변화 중 | ⭐⭐⭐ |
| Tip | GitHub Actions 공급망 보안 강화, 워크플로우 의존성 잠금 필수 | ⭐⭐ |
| Insight | 후기 10만 개 시대, AI 요약으로 정보 과부하 해결 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. 모던 JavaScript 라이브러리의 시대 전환

**핵심:** es-toolkit이 lodash를 대체하는 현실이 되었다. 번들 크기 97% 감소, 성능 2배 향상이라는 수치가 단순 마케팅이 아니라 ES Modules와 TypeScript 네이티브 설계의 결과다. Toss가 "한국에서 글로벌 JavaScript 라이브러리가 나올 수 있을까"라는 질문에서 시작한 프로젝트가 Microsoft, Yarn, Storybook 같은 기업들에 채택되는 것은 기술 부채 해결의 신호다.

**공통 의견:** 10년 전 lodash가 필요했던 이유(기본 유틸리티 부족)는 사라졌다. 현대 JavaScript는 배열 메서드, 객체 조작이 표준화되었고, 번들 크기가 Core Web Vitals에 직결되는 시대다. 레거시 라이브러리를 유지하는 것이 오히려 성능 부채가 되는 상황이 역전되었다.

**실무 적용:**

- lodash 의존성이 있는 프로젝트에서 `es-toolkit/compat`로 점진적 마이그레이션 검토 (drop-in replacement 지원)
- 새 프로젝트는 es-toolkit 우선 고려, 특히 성능이 중요한 웹 애플리케이션
- 번들 분석 도구(webpack-bundle-analyzer)로 현재 lodash 사용량 측정 후 우선순위 결정

### 2. AI 시대의 채용과 개발 패러다임 변화

**핵심:** 무신사 루키즈 채용에서 "AI Agent 코딩 테스트"를 2차 전형으로 도입한 것은 단순 트렌드가 아니라 엔지니어링의 본질 변화를 반영한다. 구현은 AI가 1분이면 하는 시대, 중요한 것은 "무엇을 만들지" 정의하는 능력이다. 의도적으로 모호한 요구사항(수강신청 시스템)을 주고 문제 정의 깊이를 평가하는 방식은 Andrew Ng이 언급한 엔지니어:PM 비율 변화(7:1 → 1:1)를 실제로 구현한 것이다.

**공통 의견:** GitHub Copilot Applied Science 팀의 사례처럼, 개발자들이 "지적 노동의 자동화"를 직접 경험하고 있다. 반복적 코딩은 AI가 담당하고, 엔지니어는 아키텍처 결정, 트레이드오프 분석, 모호함 속에서 올바른 문제 정의에 집중하는 구조로 전환 중이다.

**실무 적용:**

- 팀 내 코드 리뷰 기준을 "구현 정확성"에서 "문제 정의 타당성"으로 점진적 이동
- 신입 온보딩 시 "요구사항 불명확할 때 질문하는 방법" 교육 강화
- 프로젝트 킥오프에서 "이 기능이 정말 필요한가" 검증 단계 추가

### 3. 공급망 보안의 새로운 기준선

**핵심:** GitHub Actions 2026 보안 로드맵의 핵심은 "의존성 잠금(dependencies section in workflow YAML)"이다. 현재 워크플로우는 mutable 태그(v1, latest)로 의존성을 참조하기 때문에, 한 액션이 손상되면 즉시 모든 워크플로우에 전파된다. tj-actions/changed-files, Nx 같은 공격 사례들이 보여주듯이, CI/CD 자체가 공격 대상이 되는 시대다.

**공통 의견:** Go의 go.mod/go.sum 모델을 GitHub Actions에 적용하는 것은 "재현 가능성과 감시 가능성"을 기본값으로 만드는 것이다. 현재 대부분의 팀이 이를 수동으로 관리하고 있어서 실수가 빈번한데, 표준화되면 보안 기본값이 된다.

**실무 적용:**

- 기존 워크플로우에서 모든 액션 참조를 commit SHA로 변경 (예: `uses: actions/checkout@a5ac7e51b41094c153fa834630f06be75895f1f6`)
- 2026년 dependencies 섹션 정식 출시 전에 현재 방식으로 의존성 고정 습관화
- Dependabot으로 액션 업데이트 자동화하되, 커밋 SHA 기반으로만 허용

### 4. 정보 과부하 시대의 AI 요약 전략

**핵심:** 무신사의 "AI 후기 요약" 사례는 단순 기술 도입이 아니라 UX 문제 정의의 모범이다. 후기 10만 개는 "정보가 많다"가 아니라 "찾기 어렵다"는 문제였다. 의류는 "사이즈, 핏, 소재" 같은 구조화된 키워드로, 생활용품은 "좋은 점/참고할 점"으로 다르게 요약하는 것은 카테고리별 의사결정 방식을 이해한 결과다.

**공통 의견:** AI 요약의 가치는 "정보량"이 아니라 "의사결정 속도"다. 신상품(후기 부족)에도 요약이 필요하다는 발견은 AI가 단순 집계가 아니라 "부족한 정보를 보완하는 도구"임을 보여준다. Granite 4.0 Vision의 차트 이해 능력도 같은 맥락—숫자와 시각을 함께 해석해야 의미 있는 요약이 된다.

**실무 적용:**

- 요약 기능 도입 전에 "사용자가 정말 원하는 정보가 무엇인가" 카테고리별로 분석
- 한 가지 요약 형식 강요 대신 사용자 선택지 제공 (구조화 vs 자유형)
- 신상품/저후기 상품에 AI 요약이 더 가치 있다는 점 활용해 우선순위 결정

---

## 🛠️ 지금 당장 해볼 것

- [ ] **es-toolkit 마이그레이션 검토** — 프로젝트에서 `npm ls lodash` 실행해 의존성 확인 후, [es-toolkit 공식 문서](https://github.com/toss/es-toolkit)의 마이그레이션 가이드 읽기 (5분)

- [ ] **GitHub Actions 의존성 고정** — 현재 사용 중인 워크플로우 파일(.github/workflows/*.yml)에서 `uses: actions/checkout@v4` 형식을 찾아 [GitHub 액션 커밋 SHA 조회](https://github.com/actions/checkout/releases)로 변경 (10분)

- [ ] **Dependabot 설정 확인** — 리포지토리 Settings → Code security and analysis → Dependabot alerts/updates 활성화 여부 확인 및 활성화 (3분)

- [ ] **AI 요약 기능 분석** — 자신의 서비스에서 "정보 과부하" 문제가 있는 영역 찾기 (후기, 댓글, 검색 결과 등) 및 카테고리별 사용자 의사결정 방식 정리 (15분)

---

## 🔗 원본 출처 (클릭하여 원문 확인)
- [97% Smaller, 2x Faster: How es-toolkit Reached 10 Million Weekly Downloads](https://toss.tech/article/es-toolkit)
- [Agent-driven development in Copilot Applied Science](https://github.blog/ai-and-ml/github-copilot/agent-driven-development-in-copilot-applied-science/)
- [Granite 4.0 3B Vision: Compact Multimodal Intelligence for Enterprise Documents](https://huggingface.co/blog/ibm-granite/granite-4-vision)
- [Introducing Programmable Flow Protection: custom DDoS mitigation logic for Magic Transit customers](https://blog.cloudflare.com/programmable-flow-protection/)
- [TRL v1.0: Post-Training Library Built to Move with the Field](https://huggingface.co/blog/trl-v1)
- [후기 10만 개, 다 읽고 계신가요? - AI 후기 요약 기능 도입기](https://techblog.musinsa.com/%ED%9B%84%EA%B8%B0-10%EB%A7%8C-%EA%B0%9C-%EB%8B%A4-%EC%9D%BD%EA%B3%A0-%EA%B3%84%EC%8B%A0%EA%B0%80%EC%9A%94-ai-%ED%9B%84%EA%B8%B0-%EC%9A%94%EC%95%BD-%EA%B8%B0%EB%8A%A5-%EB%8F%84%EC%9E%85%EA%B8%B0-9efff4b12783?source=rss----f107b03c406e---4)
- [GitHub for Beginners: Getting started with GitHub security](https://github.blog/developer-skills/github/github-for-beginners-getting-started-with-github-security/)
- [To. MUSINSA ROOKIES](https://techblog.musinsa.com/to-musinsa-rookies-51e6e6fa5ee8?source=rss----f107b03c406e---4)
- [How we use Abstract Syntax Trees (ASTs) to turn Workflows code into visual diagrams](https://blog.cloudflare.com/workflow-diagrams/)
- [Liberate your OpenClaw](https://huggingface.co/blog/liberate-your-openclaw)
- [C++ 객체 수명과 암묵적 객체 생성](https://d2.naver.com/helloworld/7997284)
- [[인턴십] 2026 NAVER AI CHALLENGE를 소개합니다.](https://d2.naver.com/news/7477295)
- [What’s coming to our GitHub Actions 2026 security roadmap](https://github.blog/news-insights/product-news/whats-coming-to-our-github-actions-2026-security-roadmap/)
- [A year of open source vulnerability trends: CVEs, advisories, and malware](https://github.blog/security/supply-chain-security/a-year-of-open-source-vulnerability-trends-cves-advisories-and-malware/)
