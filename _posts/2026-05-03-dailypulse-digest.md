---
title: "[Daily Bigtech] 2026-05-03 국내 빅테크 오늘의 글"
author: gyuhwan
date: 2026-05-03 09:30:00 +0900
categories: [Daily Bigtech]
tags: [digest, auto, bigtech, daily]
description: "핵심:** Cloudflare의 'Code Orange: Fail Small' 프로젝트가 완료되어 설정 변경 시 점진적 배포와 실시간 헬스 모니터링을 도입했고, 동시에 에이전트가 Cloudflare 계정을 자동으로 생성·구독·배포할 수 있는 기능이 출시되었습니다. 이는 인프라 안정성과 개발자 경험의 동시 개선을 의미합니다."
auto_generated: true
---

## 📋 daily_pulse — 이번 주 핵심 정보

> 수집 기간: 2026-05-03 기준 최근 7일

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Cloudflare 인프라 복원력 강화 완료, 동적 워크플로우 출시 | ⭐⭐⭐ |
| New | GitHub Copilot 사용량 기반 청구 전환, 에이전트 계정 자동 생성 | ⭐⭐⭐ |
| Trend | AI 평가 비용이 새로운 병목, 포스트양자 암호화 확산 | ⭐⭐ |
| Tip | CLI 대화형/비대화형 모드 활용, Markdown 기초 마스터 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. 클라우드 인프라의 신뢰성 전쟁: 설정 변경부터 에이전트 자동화까지

**핵심:** Cloudflare의 'Code Orange: Fail Small' 프로젝트가 완료되어 설정 변경 시 점진적 배포와 실시간 헬스 모니터링을 도입했고, 동시에 에이전트가 Cloudflare 계정을 자동으로 생성·구독·배포할 수 있는 기능이 출시되었습니다. 이는 인프라 안정성과 개발자 경험의 동시 개선을 의미합니다.

**공통 의견:** 클라우드 서비스의 신뢰성은 더 이상 선택이 아닌 필수입니다. Cloudflare는 설정 변경 시 'Snapstone'이라는 내부 컴포넌트로 소프트웨어 배포와 동일한 수준의 헬스 체크를 적용했고, GitHub는 에이전트가 Stripe와의 협력으로 제로 터치 온보딩을 가능하게 했습니다. 이 두 움직임은 자동화 시대에 인프라가 얼마나 "자동화 친화적"이어야 하는지를 보여줍니다.

**실무 적용:**

- 설정 변경 시 카나리 배포 패턴 도입: 전체 네트워크에 즉시 적용하지 말고 일부 트래픽에만 먼저 적용 후 메트릭 모니터링
- 에이전트 워크플로우 설계 시 계정 생성, 결제, 배포를 한 번에 처리할 수 있는 API 체인 구성
- 설정 변경 이력 추적 및 자동 롤백 메커니즘 구축으로 "break glass" 절차 단순화

### 2. AI 평가 비용 폭증과 에이전트 개발의 새로운 병목

**핵심:** AI 모델 평가(evals)가 새로운 컴퓨팅 병목이 되었습니다. 단일 GAIA 벤치마크 실행에 $2,829, HAL 리더보드는 $40,000을 소비했으며, 에이전트 벤치마크는 노이즈가 많아 압축 기법이 제한적입니다.

**공통 의견:** 모델 개발 초기에는 학습 비용이 주요 관심사였지만, 이제는 평가 비용이 더 빠르게 증가하고 있습니다. 특히 에이전트 기반 워크플로우는 스캐폴딩 선택에 따라 동일 작업의 비용이 33배까지 차이 나므로, 평가 설계 자체가 경제성을 좌우합니다.

**실무 적용:**

- 정적 벤치마크 대신 샘플링 기반 평가로 초기 검증 후 필요한 경우만 전체 평가 실행
- 에이전트 스캐폴딩(프롬프트 구조, 도구 선택) 최적화를 평가 비용 절감의 첫 번째 단계로 설정
- 캐싱 활용: 동일 입력에 대한 반복 평가 시 토큰 캐싱으로 비용 50% 이상 절감 가능

### 3. 포스트양자 암호화와 장기 보안 전략의 현실화

**핵심:** Cloudflare가 IPsec에 하이브리드 ML-KEM(FIPS 203) 기반 포스트양자 암호화를 일반 공개했고, Fortinet·Cisco 장비와의 상호운용성을 확보했습니다. TLS는 이미 2/3 이상이 포스트양자 보호를 받지만, WAN 네트워킹은 4년 뒤처져 있었습니다.

**공통 의견:** "Harvest-now-decrypt-later" 공격(지금 수집, 나중에 복호화)이 현실화되면서 기업들은 장기 데이터 보호를 고려해야 합니다. 하지만 IPsec 같은 레거시 프로토콜의 포스트양자 전환은 하드웨어 호환성, 표준화, 성능 오버헤드 때문에 TLS보다 훨씬 느립니다.

**실무 적용:**

- 현재 WAN 인프라의 암호화 알고리즘 감사: RSA, ECDH 기반 IPsec은 2029년까지 포스트양자 대체 계획 수립
- 하이브리드 접근: 기존 알고리즘 + 포스트양자 알고리즘을 동시 사용하여 호환성 유지
- 공급업체 지원 확인: Fortinet, Cisco 외 자사 장비 제조사의 포스트양자 로드맵 확인

### 4. 개발자 도구의 사용량 기반 청구 전환과 에이전트 시대의 가격 책정 재설계

**핵심:** GitHub Copilot이 6월 1일부터 사용량 기반 청구(토큰 소비)로 전환합니다. 기존 정액제는 빠른 질문과 수시간 자동 코딩 세션을 동일 가격으로 처리했지만, 에이전트 워크플로우의 확산으로 이 모델이 지속 불가능해졌습니다.

**공통 의견:** 에이전트 개발이 기본값이 되면서 단순 채팅과 자동화 코딩의 비용 차이가 극대화되었습니다. GitHub는 이를 인정하고 투명한 가격 책정으로 전환하며, 동시에 DeepInfra 같은 대체 추론 제공자를 Hugging Face 허브에 통합하여 선택지를 늘리고 있습니다.

**실무 적용:**

- 5월 초 GitHub 청구 미리보기 활성화하여 팀의 예상 비용 파악
- 에이전트 작업 시 토큰 캐싱 활용: 동일 컨텍스트 재사용 시 캐시된 토큰은 90% 할인
- 대체 추론 제공자 평가: DeepInfra, Together AI 등으로 비용 최적화 검토

---

## 🛠️ 지금 당장 해볼 것

- [ ] **Cloudflare 설정 배포 정책 검토** — 자사 인프라에서 설정 변경 시 카나리 배포 패턴 적용 가능 여부 확인. 참고: https://blog.cloudflare.com/code-orange-fail-small-complete/

- [ ] **GitHub Copilot 청구 미리보기 활성화** — github.com 로그인 후 Billing Overview 페이지에서 "Preview bill experience" 활성화하여 6월 전환 전 예상 비용 확인

- [ ] **CLI 대화형 모드 실습** — 터미널에서 `copilot` 입력 후 대화형 모드로 진입, 비대화형 모드(`copilot -p "prompt"`)와 비교 체험. 참고: https://github.blog/ai-and-ml/github-copilot/github-copilot-cli-for-beginners-interactive-v-non-interactive-mode/

- [ ] **포스트양자 암호화 지원 확인** — 자사 WAN 장비(Fortinet, Cisco 등) 제조사 웹사이트에서 ML-KEM 지원 로드맵 검색 및 문서화

- [ ] **토큰 캐싱 테스트** — Hugging Face 또는 DeepInfra API로 동일 프롬프트 반복 실행 시 캐싱 효과 측정. 참고: https://huggingface.co/blog/inference-providers-deepinfra

---

## 🔗 원본 출처 (클릭하여 원문 확인)
- [Code Orange: Fail Small is complete. The result is a stronger Cloudflare network](https://blog.cloudflare.com/code-orange-fail-small-complete/)
- [Introducing Dynamic Workflows: durable execution that follows the tenant](https://blog.cloudflare.com/dynamic-workflows/)
- [GitHub Copilot CLI for Beginners: Interactive v. non-interactive mode](https://github.blog/ai-and-ml/github-copilot/github-copilot-cli-for-beginners-interactive-v-non-interactive-mode/)
- [Post-quantum encryption for Cloudflare IPsec is generally available](https://blog.cloudflare.com/post-quantum-ipsec/)
- [Agents can now create Cloudflare accounts, buy domains, and deploy](https://blog.cloudflare.com/agents-stripe-projects/)
- [AI evals are becoming the new compute bottleneck](https://huggingface.co/blog/evaleval/eval-costs-bottleneck)
- [비개발자의 AI 협업 도전기 — 생산성 측정하려다 서버까지 띄운 9일](https://d2.naver.com/helloworld/2017402)
- [DeepInfra on Hugging Face Inference Providers 🔥](https://huggingface.co/blog/inference-providers-deepinfra)
- [GitHub for Beginners: Getting started with Markdown](https://github.blog/developer-skills/github/github-for-beginners-getting-started-with-markdown/)
- [Skipper: Building Airbnb’s embedded workflow engine](https://medium.com/airbnb-engineering/skipper-building-airbnbs-embedded-workflow-engine-f6c34552146f?source=rss----53c7c27702d5---4)
- [Introducing NVIDIA Nemotron 3 Nano Omni: Long-Context Multimodal Intelligence for Documents, Audio and Video Agents](https://huggingface.co/blog/nvidia/nemotron-3-nano-omni-multimodal-intelligence)
- [Securing the git push pipeline: Responding to a critical remote code execution vulnerability](https://github.blog/security/securing-the-git-push-pipeline-responding-to-a-critical-remote-code-execution-vulnerability/)
- [Shutdowns, power outages, and conflict: a review of Q1 2026 Internet disruptions](https://blog.cloudflare.com/q1-2026-internet-disruption-summary/)
- [An update on GitHub availability](https://github.blog/news-insights/company-news/an-update-on-github-availability/)
- [GitHub Copilot is moving to usage-based billing](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)
