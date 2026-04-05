---
title: "[Tech] 2026-04-05 기술 동향: LLM"
author: gyuhwan
date: 2026-04-05 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** PraisonAI의 `execute_code()` 함수는 3단계 보호층을 갖추고 있었지만, Python의 동적 메서드 오버라이딩을 이용해 모두 무력화되었습니다. `_safe_getattr`이 `startswith()`로 위험한 import(`os`, `subprocess`, `sys`)를 검증할 때, 공격자는 `str`을 상속받은 악의적 클래스를 만들어 검증 단계에서는 `True`, 실행 단계에서는 `False`를 반환하도록 했습니다."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | PraisonAI 프레임워크에서 48시간 내 7개 CVE 동시 발견 — 완전한 시스템 침해 가능 | ⭐⭐⭐ |
| Tip | AI 에이전트 보안 도구 선택 시 데이터 중독, 모델 역추적 공격 대비 필수 | ⭐⭐⭐ |
| Trend | LLM 기반 멀티에이전트 프레임워크의 인증/샌드박스 취약점이 업계 표준 문제로 대두 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. 동적 디스패치를 악용한 샌드박스 우회 — 타입 검증의 함정

**핵심:** PraisonAI의 `execute_code()` 함수는 3단계 보호층을 갖추고 있었지만, Python의 동적 메서드 오버라이딩을 이용해 모두 무력화되었습니다. `_safe_getattr`이 `startswith()`로 위험한 import(`os`, `subprocess`, `sys`)를 검증할 때, 공격자는 `str`을 상속받은 악의적 클래스를 만들어 검증 단계에서는 `True`, 실행 단계에서는 `False`를 반환하도록 했습니다.

**공통 의견:** 문자열 기반 검증은 객체 모델이 풍부한 언어(Python, JavaScript)에서 근본적으로 취약합니다. 타입만 확인하고 행동(behavior)을 검증하지 않으면 동적 디스패치로 우회 가능합니다.

**실무 적용:**

- 샌드박스 검증 로직에서 `isinstance()` 타입 체크 후 **실제 메서드 호출 결과**까지 검증하기
- 위험한 모듈 목록을 블랙리스트 방식이 아닌 **화이트리스트 방식**으로 전환 (허용된 것만 실행)
- 검증 단계와 실행 단계를 **물리적으로 분리된 프로세스**에서 수행하기

### 2. 인증 로직의 역설 — "찾지 못함 = 성공"의 위험성

**핵심:** `OAuthManager.validate_token()`이 토큰을 내부 저장소에서 찾지 못하면 `True`를 반환했습니다. 저장소가 기본값으로 비어있으므로 **모든 문자열이 유효한 토큰으로 인정**되어 MCP 도구와 에이전트 기능에 무제한 접근이 가능했습니다. 또한 `/info` 엔드포인트는 인증 없이 전체 에이전트 토폴로지(이름, 기능)를 노출했습니다.

**공통 의견:** 인증 실패는 명시적으로 거부해야 합니다. "찾지 못함"을 "허용"으로 해석하는 것은 1글자 버그지만 심각한 결과를 초래합니다.

**실무 적용:**

- 모든 인증 함수의 기본 반환값을 `False`로 설정하고, **명시적 검증 성공 시에만** `True` 반환
- 토큰 저장소 초기화 시 **더미 토큰이나 기본값 없이** 빈 상태로 시작
- 민감한 정보 반환 엔드포인트(`/info`, `/status`)에 **인증 미들웨어 필수 적용**

### 3. AI 에이전트의 확대된 공격 표면 — RAG와 MCP의 이중 위험

**핵심:** 검색 증강 생성(RAG) 파이프라인과 다중 당사자 계산(MCP)을 사용하는 에이전트는 공격 표면이 기하급수적으로 증가합니다. 단순한 모델 입력 조작을 넘어 데이터 중독(data poisoning), 모델 역추적(model inversion), 모델 추출(extraction) 공격이 가능합니다. 악의적 입력 하나가 민감한 사용자 데이터를 유출할 수 있습니다.

**공통 의견:** 성능과 정확도 중심의 AI 모델 설계는 보안을 간과합니다. 복잡한 시스템일수록 취약점 식별이 어려워지므로 **포괄적인 AI 보안 플랫폼**이 필수입니다.

**실무 적용:**

- 외부 데이터 소스(RAG 문서, 사용자 입력)에 대한 **입력 새니타이제이션 및 검증 레이어** 추가
- 에이전트가 접근할 수 있는 도구와 데이터를 **최소 권한 원칙(Principle of Least Privilege)**으로 제한
- 프로덕션 배포 전 **적대적 입력 테스트(adversarial testing)** 및 모델 추출 공격 시뮬레이션 수행

---

## 🛠️ 지금 당장 해볼 것

- [ ] PraisonAI 프로젝트 GitHub에서 7개 CVE 패치 확인 및 의존성 업데이트 — `pip install --upgrade praisonai` 실행 후 버전 확인 (https://github.com/MervinPraison/PraisonAI)

- [ ] 자신의 에이전트 코드에서 인증 로직 감사 — `grep -r "validate_token\|authenticate" .` 명령어로 모든 인증 함수 찾아 기본 반환값이 `False`인지 확인

- [ ] 샌드박스 검증 테스트 작성 — 다음 악의적 클래스를 실제 코드에 입력해 우회 가능 여부 테스트:
  ```python
  class EvilStr(str):
      def startswith(self, prefix, *args):
          return True  # 항상 True 반환
  
  # 자신의 검증 함수에 EvilStr("os") 전달해 실제로 차단되는지 확인
  ```

- [ ] 프로덕션 에이전트의 민감한 엔드포인트(`/info`, `/status`, `/tools`) 목록 작성 후 각각에 인증 미들웨어 추가 — FastAPI 예시: `@app.get("/info", dependencies=[Depends(verify_token)])`

---

## 🔗 참고 자료

- [7 CVEs in 48 Hours: How PraisonAI Got Completely Owned — And What Every Agent Framework Should Learn](https://dev.to/claude-go/7-cves-in-48-hours-how-praisonai-got-completely-owned-and-what-every-agent-framework-should-learn-434n)
- [How to Choose an AI Security Tool for Your Production Agent](https://dev.to/botguard/how-to-choose-an-ai-security-tool-for-your-production-agent-3ilh)
- [LLM·API·토큰… 헷갈리는 AI 용어 한 번에 정리](https://blog.naver.com/must_computer/224238992610)

