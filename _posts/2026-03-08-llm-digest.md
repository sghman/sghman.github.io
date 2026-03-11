---
title: "[Tech] 2026-03-08 기술 동향: LLM"
author: gyuhwan
date: 2026-03-08 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** 자율 에이전트 루프에서 비구조화된 상태 관리로 인한 반복적 오류를 해결하기 위해 명시적인 \"블랙보드\" 구조를 도입하는 기법이 주목받고 있습니다."
permalink: /posts/2026-03-08-llm-digest/
auto_generated: true
---

## 🚀 30초 퀵 서머리

- **핵심 키워드**: Structured Blackboarding, 에이전트 보안 Lethal Trifecta, MoE 비용 혁신
- **오늘의 Top Pick**: 에이전트 루프의 "보드 망각" 현상을 구조화된 블랙보드 스키마로 해결 — 일관성 있는 자율 의사결정의 열쇠
- **한줄 평**: AI 에이전트가 똑똑해질수록 상태 관리와 보안이 성능보다 중요한 병목이 된다

---

## 🕑 데일리 큐레이션 보드

| 우선순위 | 도메인 | 핵심 주제 | 추천 대상 | 난이도 |
|:---:|:---|:---|:---:|:---:|
| 🔴Must | Agent Arch | AI 에이전트의 구조화된 상태 관리(Blackboarding) 기법 | BE/Infra | ⭐⭐⭐ |
| 🔴Must | Security | LLM 에이전트 보안: 악의적 도구 응답 + 메모리 포이즈닝 | BE/FE | ⭐⭐⭐ |
| 🟡View | ML Arch | 오픈소스 LLM의 MoE 아키텍처 표준화 및 비용 혁신 | BE/Infra | ⭐⭐⭐⭐ |
| ⚪Skim | Practice | 2026년 LLM 선택 기준: 확장 사고 vs 비용 트레이드오프 | BE/FE | ⭐⭐ |

---

## 🔍 Deep Dive

### [{Dev.to}] AI 에이전트의 상태 관리 혁신: Structured Blackboarding

**Problem:** 자율 에이전트 루프에서 상태 파일을 LLM이 자유롭게 해석하면, 루프마다 다른 결론에 도달하는 "보드 망각(Board Amnesia)" 현상이 발생합니다. 에이전트가 이전 결정을 잊거나 모순된 행동을 반복하게 됩니다.

**Solution:** 명시적인 "블랙보드" 스키마를 정의하여 에이전트 간 일관성 있는 의사결정을 보장합니다. 현재 목표, 마지막 액션, 결정 로그를 구조화된 JSON 객체로 관리하면 LLM의 해석 편차를 제거할 수 있습니다.

**Trade-off:** 스키마가 지나치게 엄격하면 에이전트의 유연한 추론을 제한하고, 지나치게 느슨하면 다시 보드 망각이 발생합니다. 스키마 버전 관리와 마이그레이션 전략도 필요합니다.

```python
# Structured Blackboard 구현
from dataclasses import dataclass, field, asdict
from datetime import datetime
import json

@dataclass
class AgentBlackboard:
    """에이전트 루프 간 공유되는 구조화된 상태"""
    current_goal: str = ""
    last_action: str = ""
    action_result: str = ""    # success / failed / partial
    step_count: int = 0
    decision_log: list[dict] = field(default_factory=list)
    # 루프 간 지속할 정보
    persistent: dict = field(default_factory=dict)
    # 매 루프 초기화할 정보
    ephemeral: dict = field(default_factory=dict)

    def record_decision(self, action: str, reason: str, result: str):
        self.decision_log.append({
            "step": self.step_count,
            "action": action,
            "reason": reason,
            "result": result,
            "timestamp": datetime.now().isoformat()
        })
        self.last_action = action
        self.action_result = result
        self.step_count += 1

    def reset_ephemeral(self):
        """새 루프 시작 시 일시적 상태만 초기화"""
        self.ephemeral = {}

    def to_prompt_context(self) -> str:
        """LLM에 주입할 컨텍스트 문자열 생성"""
        return json.dumps(asdict(self), indent=2, ensure_ascii=False)

# 사용 예시
board = AgentBlackboard(current_goal="PR 리뷰 자동화")
board.record_decision("read_diff", "변경사항 파악 필요", "success")
board.record_decision("analyze_security", "보안 이슈 체크", "partial")
# → 다음 루프에서 board.to_prompt_context()를 시스템 프롬프트에 포함
```

**Result:** 블랙보드 패턴을 적용한 에이전트는 루프 간 일관성이 크게 향상되며, 디버깅 시 decision_log를 통해 에이전트의 의사결정 과정을 추적할 수 있습니다.

💡 **시니어 관점:** `persistent`와 `ephemeral`의 분리가 핵심입니다. 목표, 컨텍스트, 학습된 패턴은 persistent에, 현재 작업 중간 결과는 ephemeral에 두어야 메모리 누수 없이 장기 실행이 가능합니다.

---

### [{Dev.to}] LLM 에이전트의 보안 취약점: 악의적 도구 응답과 메모리 포이즈닝

**Problem:** 외부 API 응답을 시스템 프롬프트와 동일한 신뢰도로 처리하는 에이전트는 단 하나의 악의적 응답으로 전체 시스템이 붕괴될 수 있습니다. 2026년에는 "메모리 포이즈닝(Memory Poisoning)"까지 위협이 확장되었습니다.

**Solution:** 모든 외부 입력에 대한 스키마 검증 레이어 구현과 함께, "Lethal Trifecta"(프라이빗 데이터 접근 + 외부 비신뢰 토큰 노출 + 외부 요청 가능)를 갖춘 시스템은 아키텍처 수준에서 재설계가 필요합니다. Lakera AI의 2026년 연구에 따르면, 간접 프롬프트 인젝션을 통해 에이전트의 장기 기억에 악의적 정보를 주입하면 세션이 끝나도 공격이 지속됩니다.

**Trade-off:** 강력한 입력 검증은 응답 지연을 유발하고, 메모리 무결성 검증은 에이전트의 학습 능력을 제한할 수 있습니다.

```python
# 에이전트 보안: 도구 응답 검증 + 메모리 무결성 체크
from typing import Any
import hashlib, json

class SecureAgentToolHandler:
    """외부 도구 응답의 보안 검증 레이어"""

    BLOCKED_PATTERNS = [
        "ignore previous instructions",
        "you are now",
        "system:",
        "forget everything",
        "new role:",
    ]

    def __init__(self):
        self.memory_hashes: dict[str, str] = {}  # 메모리 무결성 해시

    def validate_response(self, tool_name: str, response: Any) -> dict:
        """도구 응답 검증 — Lethal Trifecta 방어"""
        raw = json.dumps(response, ensure_ascii=False).lower()

        # 1. 프롬프트 인젝션 패턴 탐지
        for pattern in self.BLOCKED_PATTERNS:
            if pattern in raw:
                raise SecurityError(f"Injection detected in {tool_name}: {pattern}")

        # 2. 응답 크기 제한 (과도한 컨텍스트 주입 방지)
        if len(raw) > 50_000:
            raise SecurityError(f"Response too large from {tool_name}: {len(raw)} chars")

        # 3. 스키마 검증 (예상 필드만 허용)
        return {"tool": tool_name, "data": response, "trusted": False}

    def write_memory(self, key: str, value: str):
        """메모리 쓰기 시 해시 기록 — 포이즈닝 탐지용"""
        self.memory_hashes[key] = hashlib.sha256(value.encode()).hexdigest()

    def verify_memory(self, key: str, value: str) -> bool:
        """메모리 읽기 시 무결성 검증"""
        expected = self.memory_hashes.get(key)
        actual = hashlib.sha256(value.encode()).hexdigest()
        return expected == actual

class SecurityError(Exception):
    pass
```

```bash
# 에이전트 보안 감사 체크리스트 (실행 가능한 검증)
# 1. Lethal Trifecta 점검
grep -rn "requests.get\|httpx\|aiohttp" src/ | grep -v test
# → 외부 요청 가능 코드 식별

# 2. 프롬프트 인젝션 취약점 스캔
grep -rn "user_input\|external_data" src/ | grep -c "validate\|sanitize"
# → 검증 없이 사용되는 외부 입력 식별

# 3. 메모리/상태 파일 쓰기 권한 점검
find . -name "*.json" -path "*/state/*" -exec ls -la {} \;
```

**Result:** 기업의 17%(추정)만이 에이전틱 AI를 도입한 상태이지만, Q4 2025 기준 에이전트 공격 사례가 급증하고 있습니다. "AI Security in 2026" 보고서는 프롬프트 인젝션, 도구 악용, 메모리 포이즈닝을 3대 위협으로 지정했습니다.

💡 **시니어 관점:** 에이전트에 새로운 도구를 추가할 때마다 Lethal Trifecta 체크를 수행하세요. 세 조건이 동시에 충족되는 순간 공격 표면이 기하급수적으로 확대됩니다.

---

### [{ByteByteGo}] 오픈소스 LLM 생태계의 아키텍처 수렴: MoE 표준화와 비용 혁신

**Problem:** DeepSeek V3가 $5.576M(원문 기준)이라는 저비용 학습을 달성한 이후, 오픈소스 커뮤니티가 이 아키텍처를 표준으로 빠르게 채택하고 있습니다.

**Solution:** MoE 아키텍처의 핵심은 선택적 활성화입니다. 671B 총 파라미터 중 37B만 활성화(원문 기준)하여 Dense 모델 대비 추론 비용을 대폭 절감합니다. Kimi K2, GLM-5가 DeepSeek의 Multi-Head Latent Attention과 Expert 라우팅 전략을 기반으로 자체 혁신을 추가하고 있습니다.

**Trade-off:** MoE 모델은 총 파라미터만큼의 GPU 메모리가 필요하여 (활성화 파라미터는 적어도) 모델 로딩 시 메모리 요구량이 큽니다. Expert Parallelism으로 분산해야 하며 이는 네트워크 대역폭 병목을 야기합니다.

```bash
# 로컬에서 MoE 기반 모델 성능 비교 (ollama 사용)
# 1. Dense 모델 vs MoE 모델 추론 속도 비교
ollama pull llama3.1:8b          # Dense 8B
ollama pull deepseek-r1:8b       # MoE 기반 distill

# 2. 벤치마크 실행
time ollama run llama3.1:8b "Explain MoE architecture in 3 sentences" --verbose
time ollama run deepseek-r1:8b "Explain MoE architecture in 3 sentences" --verbose

# 3. 메모리 사용량 확인
nvidia-smi --query-gpu=memory.used --format=csv -l 1
```

```python
# MoE vs Dense 비용 비교 계산
def compare_moe_vs_dense(
    total_params_b: float,
    active_params_b: float,
    tokens_per_second: int = 50,
    gpu_cost_per_hour: float = 2.50  # A100 80GB 시간당 (추정)
):
    """MoE와 Dense 모델의 추론 비용 비교"""
    # Dense: 모든 파라미터 연산
    dense_flops = total_params_b * 2 * tokens_per_second * 1e9
    # MoE: 활성화 파라미터만 연산
    moe_flops = active_params_b * 2 * tokens_per_second * 1e9
    savings_pct = (1 - moe_flops / dense_flops) * 100
    return {
        "dense_gflops": dense_flops / 1e9,
        "moe_gflops": moe_flops / 1e9,
        "compute_savings": f"{savings_pct:.1f}%",
        "note": "메모리는 총 파라미터 기준으로 여전히 필요"
    }

result = compare_moe_vs_dense(671, 37)
# {'dense_gflops': 67100.0, 'moe_gflops': 3700.0, 'compute_savings': '94.5%', ...}
```

**Result:** 오픈소스 생태계에서 "공개 → 채택 → 개선 → 재공개" 사이클이 가속화되면서, 2026년 상반기에만 MoE 기반 신규 모델이 5개 이상(추정) 출시되었습니다.

💡 **시니어 관점:** MoE 모델 도입 시 "활성화 파라미터 대비 추론 비용"과 "총 파라미터 대비 메모리 비용"을 분리하여 평가해야 합니다. 연산은 94% 절감되지만 메모리는 절감되지 않는다는 점이 실무에서 가장 큰 함정입니다.

---

### [{AwesomeAgents}] 2026년 LLM 선택 전략: 확장 사고 vs 실용성 트레이드오프

**Problem:** "가장 똑똑한 모델"을 사용하면 비용과 지연시간이 급증합니다. 작업 유형에 따른 체계적인 모델 선택 기준이 필요합니다.

**Solution:** 작업별 난이도 분류 후 적절한 모델을 계층화하여 배치합니다. 단순 작업(분류, 추출)은 경량 모델, 복잡한 추론은 확장 사고 모델을 사용합니다.

**Trade-off:** 멀티모델 아키텍처는 라우팅 로직 유지보수 부담이 있으며, 모델 업데이트 시 각 계층별 재검증이 필요합니다.

```python
# LLM 선택 의사결정 트리 자동화
import time
from dataclasses import dataclass

@dataclass
class LLMBenchResult:
    model: str
    accuracy: float
    avg_latency_ms: float
    cost_per_1k_tokens: float

def benchmark_models(prompt: str, expected: str, models: list[str]) -> list[LLMBenchResult]:
    """동일 프롬프트로 여러 모델 벤치마크"""
    results = []
    for model in models:
        start = time.time()
        # response = call_llm(model, prompt)  # 실제 API 호출
        latency = (time.time() - start) * 1000
        # accuracy = evaluate(response, expected)
        results.append(LLMBenchResult(
            model=model,
            accuracy=0.0,  # 실제 측정값으로 대체
            avg_latency_ms=latency,
            cost_per_1k_tokens=0.0  # 실제 비용으로 대체
        ))
    # 비용 대비 정확도로 정렬
    return sorted(results, key=lambda r: r.accuracy / max(r.cost_per_1k_tokens, 0.001), reverse=True)
```

**Result:** 2026년의 LLM 선택은 단순 벤치마크 점수가 아니라 작업 유형, 예산, 컨텍스트 윈도우, 응답 시간을 종합적으로 고려하는 "비용 효율성 매트릭스"로 전환되고 있습니다.

💡 **시니어 관점:** 프로덕션 환경에서는 A/B 테스트로 모델별 실제 사용자 만족도를 측정한 후 라우팅 비율을 조정하는 것이 가장 신뢰할 수 있는 방법입니다. 벤치마크 점수와 실제 업무 정확도는 상관관계가 낮을 수 있습니다.

---

## 🛠️ 지금 당장 해볼 것

- [ ] 에이전트 프로젝트에 Blackboard 패턴 적용: 위의 `AgentBlackboard` 클래스를 복사해서 기존 에이전트 루프에 `to_prompt_context()` 결과를 시스템 프롬프트에 주입 — 검색: `"structured blackboarding agent pattern python"`
- [ ] Lethal Trifecta 자가 점검 실행: `grep -rn "requests.get\|httpx" src/ | grep -v test` 로 외부 요청 코드 식별 후, 검증 레이어 유무 확인
- [ ] MoE vs Dense 추론 비용 직접 비교: `ollama pull deepseek-r1:8b && ollama run deepseek-r1:8b --verbose` 로 로컬 추론 속도 측정 — [Ollama Models](https://ollama.com/library)

---

## 🤔 생각해볼 질문

> 현재 운영 중인 AI 에이전트에서 "보드 망각" 현상이 발생하는 구간은 어디이며, persistent/ephemeral 상태를 어떤 기준으로 분리하면 루프 간 일관성을 최대화할 수 있을까?

> 자사 에이전트 시스템이 Lethal Trifecta(프라이빗 데이터 접근 + 외부 비신뢰 입력 + 외부 요청 가능)의 세 조건 중 몇 개를 충족하고 있으며, 충족 조건을 하나라도 제거하려면 어떤 아키텍처 변경이 필요한가?

---

## 🔗 참고 자료

- [Why I replaced "think freely" with structured blackboarding in my agent loops](https://dev.to/askpatrick/why-i-replaced-think-freely-with-structured-blackboarding-in-my-agent-loops-am3)
- [What Happens When an AI Agent Gets a Malicious Tool Response](https://dev.to/botguard/what-happens-when-an-ai-agent-gets-a-malicious-tool-response-10jg)
- [The Architecture Behind Open-Source LLMs](https://blog.bytebytego.com/p/the-architecture-behind-open-source)
- [The Architecture Behind Modern AI Companion Platforms](https://dev.to/ai_angels/the-architecture-behind-modern-ai-companion-platforms-163f)
- [Training LLMs with JAX: A Practitioner's Guide to High-Performance Model Training](https://shahzadasghar.medium.com/training-llms-with-jax-a-practitioners-guide-to-high-performance-model-training-d4ecd66092ca?source=rss------artificial_intelligence-5)
- [How to Choose the Right LLM in 2026: A Practical Guide](https://awesomeagents.ai/guides/how-to-choose-an-llm-2026/)
- [My LLM coding workflow going into 2026 - by Addy Osmani - Elevate](https://addyo.substack.com/p/my-llm-coding-workflow-going-into)
- [How to Actually Learn LLMs in 2026 | by Hareem Fatima - Medium](https://medium.com/data-and-beyond/how-to-actually-learn-llms-in-2026-95f40aff2f49)
- [AI Security in 2026: Prompt Injection, the Lethal Trifecta, and How to Defend](https://airia.com/ai-security-in-2026-prompt-injection-the-lethal-trifecta-and-how-to-defend/)
- [AI Agent Security: The Complete Enterprise Guide for 2026](https://www.mintmcp.com/blog/ai-agent-security)
