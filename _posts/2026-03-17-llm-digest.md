---
title: "[Tech] 2026-03-17 기술 동향: LLM"
author: gyuhwan
date: 2026-03-17 09:00:00 +0900
categories: [Tech Newsletter]
tags: [digest, auto, LLM]
description: "핵심:** Agent와의 대화를 종료해도 기억하도록 세션 메타데이터와 메시지를 디스크에 저장하고, LLM 토큰 한계에 대비해 이중 방어 전략(요약 + 절단)으로 컨텍스트를 압축하는 기법."
auto_generated: true
---

## 🕑 Quick Glance

| 분류 | 주요 내용 | 중요도 |
|:---:|:---|:---:|
| New | Agent 대화 이력 관리 및 컨텍스트 압축 기법 | ⭐⭐⭐ |
| Tip | Multimodal RAG로 이미지 기반 문서 검색 구현 | ⭐⭐⭐ |
| Trend | 엔비디아 LPU 삼성 생산, SKT 자체 LLM 'A.X K1' 고도화 | ⭐⭐ |

---

## 💡 Deep Dive

### 1. Agent 시스템의 대화 이력 관리: 영속성과 컨텍스트 최적화

**핵심:** Agent와의 대화를 종료해도 기억하도록 세션 메타데이터와 메시지를 디스크에 저장하고, LLM 토큰 한계에 대비해 이중 방어 전략(요약 + 절단)으로 컨텍스트를 압축하는 기법.

**공통 의견:** OpenClaw 시리즈에서 강조하는 것처럼, Agent의 실용성은 단순한 루프 구조가 아니라 **대화 지속성**과 **토큰 효율성**에 달려 있다. 현재 LLM의 토큰 윈도우도 장시간 대화에서는 부족할 수 있으므로, 적극적인 메모리 관리가 필수다.

**실무 적용:**

- `.history/` 디렉토리 구조로 세션별 메시지를 JSONL 형식으로 저장 — 각 세션마다 별도 파일로 관리하면 확장성과 조회 성능 모두 확보
- `HistoryStore` 클래스로 `create_session()`, `save_message()`, `get_messages()` 메서드 구현 — 대화 시작 시 이전 세션 로드, 매 메시지마다 저장하는 패턴
- 두 단계 컨텍스트 압축: 먼저 도구 출력 결과 중 과도하게 큰 부분 절단(저비용), 그 후 LLM을 호출해 오래된 메시지 요약(고비용) — 토큰 임계값 기준으로 단계적 적용

### 2. Multimodal RAG: 이미지 기반 문서 검색의 근본적 해결

**핵심:** 표준 RAG는 텍스트만 추출하므로 다이어그램, 사진, 스키마 등 시각 정보를 완전히 무시한다. 비전 모델을 번역 계층으로 활용해 이미지를 의미 있는 텍스트 설명으로 변환하면, 기존 임베딩·벡터 검색 파이프라인을 수정 없이 확장할 수 있다.

**공통 의견:** 기술 매뉴얼, 의료 영상 보고서, 건축 도면 같은 시각 정보 중심 문서에서 표준 RAG는 **근본적 한계**를 보인다. 이를 해결하려면 추출(Extract) → 설명(Describe) → 검색·렌더링(Retrieve & Render) 세 단계를 재설계해야 한다.

**실무 적용:**

- PDF/DOCX 업로드 시 텍스트와 이미지를 동시에 추출 — 기존 텍스트 청킹 파이프라인과 병렬로 이미지 처리 채널 추가
- 각 이미지를 비전 모델(GPT-4V, Claude Vision 등)에 전달해 "이 이미지가 무엇을 보여주는가"라는 의미 있는 설명 생성 — alt-text 대신 실제 의미 캡처
- 벡터 스토어에 이미지 설명을 텍스트로 임베딩하고, 검색 결과 반환 시 설명과 원본 이미지 모두 제공 — 사용자가 시각 정보에 직접 접근 가능

### 3. LLM 인프라 경쟁: 엔비디아 LPU와 국내 자체 LLM 개발 가속화

**핵심:** 엔비디아가 LLM 추론 특화 LPU를 삼성에서 생산하기로 발표(2026년 하반기 출하 예정)하는 한편, SKT는 자체 초거대 언어모델 'A.X K1'을 고도화하며 전직원 AI 개발 체계로 전환 중이다.

**공통 의견:** 글로벌 칩 제조 경쟁과 국내 기업의 자체 LLM 개발이 동시에 가속화되고 있다. 삼성의 LPU 생산 수주는 반도체 입지 강화이고, SKT의 'A.X K1' 고도화는 AI 인프라 자립화 전략이다. 이는 LLM 기술이 더 이상 OpenAI·Google 독점이 아니라 **국가·기업 차원의 전략 자산**임을 의미한다.

**실무 적용:**

- 자체 LLM 개발 계획이 있다면 추론 최적화(LPU 같은 전문 칩)와 파인튜닝 인프라 동시 구축 필요
- 기존 OpenAI/Claude API 의존도를 낮추기 위해 오픈소스 LLM(Llama, Mistral 등) 평가 및 파일럿 시작 — 비용과 지연시간 개선 가능성 검토

---

## 🛠️ 지금 당장 해볼 것

- [ ] OpenClaw 공식 저장소에서 Agent 루프 + 이력 관리 코드 확인 — [dev.to 시리즈 첫 번째 글](https://dev.to/zeling_chen_73840b4951f53/understand-openclaw-by-building-one-1-every-agent-starts-as-a-loop-36ej)의 repo 링크 참고

- [ ] LangChain으로 간단한 Agent 만들고 대화 이력을 JSON 파일로 저장하는 코드 작성 — `pip install langchain openai` 후 `HistoryStore` 클래스 구현 시작

- [ ] 자신의 RAG 시스템에서 이미지 처리 현황 점검 — 현재 PDF 업로드 시 이미지가 버려지는지 확인하고, Claude Vision API 또는 GPT-4V로 이미지 설명 생성 파이프라인 프로토타입 구축

- [ ] SKT A.X K1 관련 공식 자료 검색 — 자체 LLM 스펙과 API 가용성 확인

---

## 🔗 참고 자료

- [Understand OpenClaw by Building One - 3: Pack the Conversation And Carry On](https://dev.to/zeling_chen_73840b4951f53/understand-openclaw-by-building-one-3-pack-the-conversation-and-carry-on-432a)
- [Understand OpenClaw by Building One - 2: Gear up Your Agent](https://dev.to/zeling_chen_73840b4951f53/understand-openclaw-by-building-one-2-gear-up-your-agent-1gcp)
- [Understand OpenClaw by Building One - 1: Every Agent Starts as a Loop](https://dev.to/zeling_chen_73840b4951f53/understand-openclaw-by-building-one-1-every-agent-starts-as-a-loop-36ej)
- [Standard RAG Is Blind — Building Multimodal RAG in .NET to Fix It](https://dev.to/argha_dev/standard-rag-is-blind-building-multimodal-rag-in-net-to-fix-it-4998)
- [LangChain 시작하기](https://blog.naver.com/hyobin_jang/224219368206)
- [엔비디아 차세대 LPU 삼성 생산에 삼성전자 강세](https://blog.naver.com/prepared_optimist/224219386750)
- [SKT 전직원이 AI를 직접 개발? 파격적인 'AX 전환' 선언의...](https://blog.naver.com/mykosoo9/224219487776)

