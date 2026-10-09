---
layout: post
title: "AI 서비스 핵심 원리 - RAG, Tool Calling, Agent, 보안과 평가"
description: "LLM 추론, RAG, Tool Calling, Structured Output, Agent Workflow, 보안, 평가와 운영 원리를 실행 주체가 명확한 문장으로 정리합니다."
date: 2026-10-09
categories: [career]
tags: [AI, LLM, RAG, Tool Calling, Agent, LangGraph, FastAPI, Security, Evaluation, Study]
---

이 문서는 AI 서비스 개발에 필요한 **입력 → 실행 → 검증 → 출력** 흐름을 정리한다. 각 단계에서는 실행 주체와 책임을 명시한다.

먼저 보기: [AI 핵심 용어 한 줄 암기 노트]({% post_url 2026-10-09-ai-agent-core-terms-memorization %})

## 1. LLM은 어떻게 응답을 생성하는가?

**핵심:** LLM은 입력 토큰과 문맥을 바탕으로 다음 토큰을 예측하면서 응답을 생성한다.

```text
사용자가 질문을 입력한다.
  -> 애플리케이션이 질문·지시·참고 정보를 구성한다.
  -> 토크나이저가 입력을 토큰으로 변환한다.
  -> LLM이 토큰을 순차적으로 생성한다.
  -> 애플리케이션이 생성된 응답을 사용자에게 전달한다.
```

- **Token:** 토큰은 모델이 텍스트를 처리할 때 사용하는 단위다.
- **Context Window:** Context Window는 모델이 한 번의 처리에서 다룰 수 있는 토큰 범위다.
- **Hallucination:** Hallucination은 모델이 사실과 다른 정보를 사실처럼 생성하는 현상이다.
- **정확성 검증:** 애플리케이션은 모델 응답을 신뢰 가능한 문서, DB 결과, 업무 규칙과 대조한다.

## 2. Tool Calling은 어떻게 실행되는가?

**핵심:** LLM은 도구 호출을 요청하고, 애플리케이션 서버는 실행 권한을 확인하여 도구를 호출한다.

```text
1. 애플리케이션이 Tool 이름과 입력 스키마를 LLM에 제공한다.
2. LLM이 사용할 Tool과 호출 인자를 선택한다.
3. 애플리케이션 서버가 인자 형식과 사용자 권한을 검증한다.
4. 애플리케이션 서버가 실제 함수 또는 API를 실행한다.
5. 애플리케이션 서버가 실행 결과를 LLM에 전달한다.
6. LLM이 실행 결과를 바탕으로 응답을 작성한다.
```

| 개념 | 정확한 역할 |
|---|---|
| Tool Calling | LLM은 실행할 도구와 인자를 지정하여 호출을 요청한다. |
| MCP | MCP는 AI 애플리케이션과 MCP Server의 도구·데이터 교환을 표준화한다. |
| MCP Server | MCP Server는 도구 목록과 실행 인터페이스 등을 클라이언트에 제공한다. |
| REST API | REST API는 HTTP 기반으로 애플리케이션 기능과 데이터를 제공한다. |

**적용 기준:** 개발자는 서비스 내부 함수를 Tool로 제공할 수 있다. 개발자는 여러 AI 클라이언트가 같은 도구를 사용하도록 MCP Server를 구현할 수 있다.

## 3. Structured Output은 무엇인가?

**핵심:** Structured Output은 LLM이 JSON Schema처럼 지정된 구조에 맞춰 데이터를 출력하도록 하는 방식이다.

다음 JSON은 상품 분석 응답의 예시다.

```json
{
  "product_id": 10,
  "reason": "판매량 상위"
}
```

- **LLM:** LLM은 지정된 스키마에 맞춰 응답을 생성한다.
- **FastAPI:** FastAPI 서버는 Pydantic 모델로 응답의 자료형과 필수 필드를 검증한다.
- **업무 서비스:** 업무 서비스는 상품 ID의 존재 여부와 판매량 기준을 DB 데이터로 검증한다.

**적용 기준:** 개발자는 LLM 결과를 API 응답이나 DB 저장 데이터로 사용할 때 Structured Output과 업무 검증을 함께 설계한다.

## 4. RAG는 어떻게 동작하는가?

**핵심:** RAG는 외부 자료를 검색하여 LLM의 Context에 근거를 제공하는 방식이다.

### 문서 준비

```text
수집기가 문서를 가져온다.
  -> 분할기가 문서를 Chunk로 나눈다.
  -> 임베딩 모델이 문서 Chunk를 벡터로 변환한다.
  -> 인덱스가 벡터와 문서 정보를 저장한다.
```

### 질문 처리

```text
사용자가 질문을 입력한다.
  -> 검색기가 관련 문서를 찾는다.
  -> 재정렬기가 관련성이 높은 문서를 우선 배치한다(선택 사항).
  -> 애플리케이션이 질문과 검색 결과를 LLM에 전달한다.
  -> LLM이 제공된 근거를 활용해 답변을 생성한다.
  -> 애플리케이션이 답변과 출처를 사용자에게 전달한다.
```

- **Chunking:** 분할기는 문서를 검색에 적합한 크기로 나눈다.
- **Embedding:** 임베딩 모델은 텍스트를 의미적 유사도 비교에 사용할 수 있는 벡터로 변환한다.
- **Vector Search:** 벡터 검색기는 벡터 간 유사도를 이용해 관련 문서를 찾는다.
- **Hybrid Search:** 하이브리드 검색기는 키워드 검색과 벡터 검색을 결합한다.

### RAG와 Fine-tuning

| 기준 | RAG | Fine-tuning |
|---|---|---|
| 작동 | RAG는 외부 문서를 검색해 모델 입력에 추가한다. | Fine-tuning은 학습 데이터로 모델의 가중치를 조정한다. |
| 적용 | RAG는 최신 문서와 사내 지식 조회에 적합하다. | Fine-tuning은 출력 형식과 특정 작업 수행 패턴 개선에 활용된다. |
| 갱신 | 운영자는 문서와 검색 인덱스를 갱신한다. | 개발자는 학습 데이터를 준비하고 모델을 추가 학습시킨다. |

**적용 기준:** 문서 검색이 핵심인 서비스는 RAG를 먼저 검토한다. 개발자는 검색 정확도를 정답 문서 데이터셋으로 측정한다.

## 5. Workflow와 Agent는 어떻게 다른가?

| 구성 요소 | 실행 주체와 역할 |
|---|---|
| Workflow | 개발자는 단계와 조건을 정의하고, 실행 엔진은 정해진 규칙에 따라 작업을 수행한다. |
| Agent | LLM은 다음 행동과 도구 선택을 판단하고, Agent 실행 환경은 그 판단을 수행한다. |
| LangChain | LangChain은 모델, 도구, Agent의 구성 요소를 통합한다. |
| LangGraph | LangGraph는 Node, Edge, State를 사용해 분기·반복·상태 저장을 관리한다. |
| Harness | Agent Harness는 모델 호출, 도구 실행, 권한, 상태와 오류 처리를 통합 관리한다. |

### Text2SQL 검증 워크플로 예시

```text
LLM이 SQL 후보를 생성한다.
  -> SQL 검증기가 구문과 실행 정책을 확인한다.
     -> 검증 성공: DB 서비스가 승인된 SQL을 실행한다.
                  -> 결과 검증기가 조회 결과를 업무 기준과 대조한다.
                  -> API가 사용자에게 결과를 전달한다.
     -> 검증 오류: 워크플로가 오류 정보를 LLM에 제공한다.
                  -> LLM이 SQL을 다시 생성한다.
                  -> 워크플로가 설정된 재시도 횟수를 관리한다.
                  -> 재시도 종료 시 API가 검증 실패 결과를 전달한다.
```

**적용 기준:** 개발자는 단순 모델 호출에 SDK를 사용한다. 개발자는 다단계 상태 관리와 반복 검증에 LangGraph를 사용할 수 있다.

## 6. AI 서비스 보안은 어떻게 설계하는가?

**핵심:** 백엔드 서버는 사용자 권한과 Tool 실행 정책을 강제하고, LLM은 승인된 정보와 기능을 활용한다.

| 보안 대상 | 서버에서 구현할 통제 |
|---|---|
| Prompt Injection | 서버는 외부 문서·도구 결과를 데이터로 분리하고, 승인된 지시와 도구 실행 정책을 유지한다. |
| Tool 권한 | 서버는 사용자별 허용 Tool과 최소 권한을 적용한다. |
| 중요 변경 | 서버는 중요 변경 작업에 사용자 또는 관리자의 승인을 요구한다. |
| 데이터 접근 | 검색 서비스는 사용자 권한에 따라 조회 가능한 문서를 필터링한다. |
| 출력 검증 | 서버는 스키마, 업무 규칙, DB 결과를 기준으로 LLM 출력을 검증한다. |
| 비용 관리 | 서버는 사용자별 호출 한도, Timeout과 재시도 횟수를 적용한다. |

### Text2SQL의 안전한 실행 흐름

```text
인증 서비스가 사용자를 인증한다.
  -> 인가 서비스가 사용자에게 허용된 데이터 범위를 결정한다.
  -> 애플리케이션이 허용된 스키마와 업무 메타데이터를 LLM에 제공한다.
  -> LLM이 조회용 SQL 후보를 생성한다.
  -> SQL 검증기가 구문, 허용 테이블, 조회 전용 정책을 검증한다.
  -> 최소 권한 DB 계정이 제한된 시간과 건수로 조회한다.
  -> 결과 검증기가 조회 결과를 업무 규칙과 대조한다.
  -> API가 사용자에게 검증된 결과를 전달한다.
```

## 7. AI 서비스의 품질은 어떻게 평가하는가?

**핵심:** 평가 시스템은 정답 데이터셋과 반복 가능한 테스트로 AI 서비스 품질을 측정한다.

| 평가 대상 | 평가 방법 |
|---|---|
| RAG 검색 | 평가 시스템은 정답 문서의 Recall@K 등을 측정한다. |
| 최종 답변 | 평가 시스템은 사실 정확성, 근거 충실도, 출처 적절성을 평가한다. |
| Tool/Agent | 평가 시스템은 호출 인자 유효성, 도구 실행 성공률과 작업 완료율을 측정한다. |
| Text2SQL | 평가 시스템은 SQL 실행 성공률과 정답 결과의 일치율을 측정한다. |
| 보안 | 보안 테스트는 권한 제한과 승인 절차의 작동을 검증한다. |
| 운영 | 모니터링 시스템은 오류율, 지연시간, 토큰 사용량과 요청별 비용을 측정한다. |

개발자는 모델, Prompt, 도구, 검색 인덱스를 변경할 때 동일한 평가 데이터셋으로 회귀 테스트를 실행한다.

## 8. AI 서비스는 어떻게 운영하는가?

- **Reliability:** API 서버는 Timeout, 제한된 재시도, 지수 백오프와 Fallback 정책을 적용한다.
- **Observability:** 모니터링 시스템은 요청 ID로 API, 검색, 모델 호출, 도구 실행을 추적한다.
- **Cost:** 비용 관리 시스템은 토큰 사용량, 모델별 비용, 사용자별 예산 한도를 집계한다.
- **Security:** 인증·인가 서비스는 데이터 접근 권한과 도구 실행 권한을 확인한다.
- **Release:** 배포 파이프라인은 모델·Prompt·검색 인덱스 버전을 기록하고 평가 결과를 배포 기준에 반영한다.

## 9. 실무 적용 순서

1. **FastAPI + LLM SDK:** 개발자는 모델 호출과 JSON 응답을 구현한다.
2. **Tool Calling:** 개발자는 권한 검사를 포함한 API·DB 조회 도구를 연결한다.
3. **RAG 또는 Text2SQL:** 개발자는 정답 데이터셋으로 검색·조회 결과의 정확도를 평가한다.
4. **LangGraph:** 개발자는 분기, 반복 검증, 사용자 승인과 상태 저장을 구현한다.
5. **운영 품질:** 개발자는 로그, 비용 계측, 회귀 테스트와 보안 검증을 구축한다.

## 10. 암기 요약

> **LLM은 응답을 생성한다 → RAG는 근거를 검색한다 → Tool Calling은 실행을 요청한다 → 서버는 권한과 결과를 검증한다 → Workflow/Agent는 작업을 진행한다 → Evaluation은 품질을 측정한다.**

다음 단계: [AI 서비스 개발 면접 질문과 30초 답변]({% post_url 2026-10-09-ai-service-engineering-interview-questions %})

## 공식 참고 자료

- [OpenAI Function calling](https://developers.openai.com/api/docs/guides/function-calling)
- [OpenAI Structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs)
- [OpenAI Retrieval](https://developers.openai.com/api/docs/guides/retrieval)
- [OpenAI Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP LLM06:2025 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
