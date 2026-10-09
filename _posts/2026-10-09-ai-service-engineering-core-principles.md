---
layout: post
title: "AI 서비스 핵심 원리 - RAG, Tool Calling, Agent, 보안과 평가"
description: "Java/Spring과 FastAPI 개발자를 위한 AI 서비스 동작 원리 요약. RAG, Tool Calling, Workflow, Structured Output, Prompt Injection, Evaluation과 운영 기준을 간단히 정리합니다."
date: 2026-10-09
categories: [career]
tags: [AI, LLM, RAG, Tool Calling, Agent, LangGraph, FastAPI, Security, Evaluation, Study]
---

> **암기 원칙:** 정의보다 **입력 → 처리 → 검증 → 출력** 흐름과 **언제 사용할지**를 설명할 수 있어야 한다.
>
> 먼저 보기: [AI 핵심 용어 한 줄 암기 노트]({% post_url 2026-10-09-ai-agent-core-terms-memorization %})

## 1. LLM은 어떻게 동작하나?

**한 줄:** LLM은 입력을 토큰으로 처리하고, 문맥을 참고하여 다음 토큰을 예측하는 과정을 반복해 응답한다.

```text
사용자 입력 + 지시 + 참고 자료(Context)
  -> 토큰화
  -> LLM 추론
  -> 출력 토큰 생성
  -> 응답
```

- **Token:** 모델이 입력·출력을 처리하는 단위. 글자 수와 1:1로 일치하지 않는다.
- **Context Window:** 모델이 한 번에 처리할 수 있는 토큰 범위. 무한한 기억이 아니다.
- **Hallucination:** 근거가 없거나 틀린 내용을 그럴듯하게 생성하는 현상.
- **핵심 대응:** 정확성이 중요한 결과는 출처, 정답 데이터, 도메인 규칙으로 별도 검증한다.

## 2. Tool Calling은 어떻게 실행되나?

**한 줄:** LLM은 도구 호출을 *제안*하고, 실제 실행과 권한 검사는 애플리케이션이 담당한다.

```text
사용자 요청
  -> 애플리케이션이 사용 가능한 Tool과 스키마를 LLM에 전달
  -> LLM이 Tool 이름과 인자 생성
  -> 서버가 인자·권한·허용 기능 검증
  -> 서버가 실제 API/DB/함수 호출
  -> 실행 결과를 LLM에 전달
  -> LLM이 최종 응답 생성
```

- **Tool Calling:** 모델과 애플리케이션 사이에서 함수를 요청·실행하는 메커니즘.
- **MCP:** 여러 AI 애플리케이션에 도구·데이터를 표준 방식으로 노출하는 통신 규약.
- **REST API:** 실제 업무 기능을 제공할 수 있는 인터페이스. MCP Server가 내부적으로 REST API를 사용할 수 있다.
- **언제 사용?** 조회·검색·계산·외부 시스템 작업이 필요할 때. MCP는 복수 클라이언트에서 동일한 도구를 재사용할 때 검토한다.

**기억할 점:** 모델이 반환한 함수 인자는 신뢰할 수 없는 입력이다. 함수의 실행 성공 여부와 비즈니스 권한은 서버에서 결정한다.

## 3. Structured Output이란?

**한 줄:** LLM의 응답을 JSON Schema 같은 **정해진 구조**로 받는 방식이다.

```json
{
  "product_id": 10,
  "reason": "판매량 상위",
  "confidence": 0.85
}
```

- **장점:** 임의 문장 파싱을 줄이고 DTO/Pydantic 모델로 받기 쉽다.
- **한계:** 구조가 맞아도 상품 ID, 이유, 신뢰도가 실제로 맞다는 뜻은 아니다.
- **검증:** 스키마 유효성 + 업무 조건 검증 + DB 조회 결과 대조를 별도로 한다.

## 4. RAG는 어떻게 동작하나?

**한 줄:** 외부 문서를 **검색해서 필요한 내용을 Context로 제공**하고 답변하게 하는 방식이다.

```text
[사전 준비]
문서 수집 -> 문서 분할(Chunking) -> Embedding -> 검색 인덱스 저장

[질문 처리]
질문 -> 관련 문서 검색(Vector/Keyword/Hybrid)
     -> 필요 시 재정렬(Reranking)
     -> 검색 결과 + 질문을 LLM에 전달
     -> 근거 기반 답변 및 출처 표시
```

- **Embedding:** 문장·문서를 의미를 반영하는 수치 벡터로 표현.
- **Vector DB/Index:** 벡터 유사도 기반 검색에 사용하는 저장·검색 수단.
- **Chunking:** 검색하기 적절한 크기로 문서 분할.
- **언제 사용?** 사내 규정, 매뉴얼, 법령 등 외부 문서를 근거로 답해야 할 때.
- **주의:** 검색 정확도가 낮으면 답변도 흔들린다. Vector 검색만이 정답은 아니며 키워드/하이브리드 검색도 검토한다.

### RAG vs Fine-tuning

| 구분 | RAG | Fine-tuning |
|---|---|---|
| 변경 대상 | 모델에 제공하는 근거(Context) | 모델의 가중치 |
| 적합한 문제 | 최신·사내 지식 조회, 근거 표시 | 특정 출력 양식·행동 패턴 개선 |
| 지식 갱신 | 문서·인덱스 갱신 | 학습 데이터 준비 및 재학습 필요 |
| 주의점 | 검색 실패·잘못된 근거 | 학습 비용, 최신 정보 반영의 어려움 |

둘은 함께 쓸 수도 있다. **문서가 최신이어야 한다면 보통 RAG부터 검토한다.**

## 5. Workflow와 Agent는 무엇이 다른가?

- **Workflow:** 개발자가 순서·조건을 명시한다.
- **Agent:** LLM이 도구 선택 등 일부 다음 행동을 동적으로 결정한다.
- **LangChain:** 모델·도구·Agent를 조합하는 프레임워크.
- **LangGraph:** 상태(State), 실행 단계(Node), 전이(Edge), 중단·재개 등을 설계하는 프레임워크.
- **Harness:** Agent의 실행 루프, 도구 권한, 상태, 관측과 오류 처리를 둘러싼 실행 환경.

```text
질문 -> SQL 생성 -> SQL 검증
                     | 성공 -> 승인된 조회 실행 -> 결과 검증 -> 응답
                     | 실패 -> 오류 전달 -> 재생성(재시도 횟수 제한)
                                         -> 실패 시 안전하게 종료
```

**선택 기준:** 단순 호출은 SDK만으로 충분하다. 여러 단계의 분기·상태 복구가 필요할 때 LangGraph를 고려한다. 도구가 있다고 무조건 자율 Agent로 구현하지 않는다.

## 6. AI 서비스 보안은 무엇이 다른가?

LLM의 입력은 사용자 질문뿐 아니라 검색 문서, 웹 페이지, 도구 결과까지 포함할 수 있다. **외부 자료에 적힌 명령을 시스템의 지시로 취급하면 Prompt Injection 위험이 생긴다.**

| 위험 | 적용할 통제 |
|---|---|
| Prompt Injection | 외부 내용을 비신뢰 데이터로 취급, 권한 검사 별도 유지 |
| Excessive Agency | Tool 최소화, 최소 권한, 중요 변경 전 승인 |
| 부정확한 출력 | JSON Schema + 업무 검증 + 근거 확인 |
| 정보 유출 | 검색 결과 권한 필터, Secret 비노출, 로그 민감정보 제거 |
| 과도한 호출·비용 | 사용자별 한도, Timeout, 재시도 상한, 사용량 계측 |

**핵심:** Prompt만으로 접근통제를 보장하지 않는다. 인증·인가와 실행 차단은 백엔드에서 적용한다.

### Text2SQL 안전한 구현 예시

```text
사용자 인증/인가
  -> 허용된 DB 스키마와 업무 메타데이터 제공
  -> LLM SQL 후보 생성
  -> SQL 구조/허용 테이블/읽기 전용 여부 검증
  -> 최소 권한 DB 계정으로 조회(시간·건수 제한)
  -> 조회 결과와 업무 규칙 검증
  -> 응답
```

SQL이 문법상 올바른 것과 업무적으로 맞는 것은 다르다. **정확한 SQL인지 판단할 정답 데이터셋**이 필요하다.

## 7. AI 품질은 어떻게 평가하나?

**한 줄:** 느낌이나 데모 한 번이 아니라 **정답 데이터셋과 재현 가능한 테스트**로 측정한다.

| 평가 대상 | 확인할 지표/방법 |
|---|---|
| 검색(RAG) | 정답 문서가 검색되는 비율(Recall@K 등) |
| 최종 답변 | 사실 정확성, 근거 충실도, 출처 적절성 |
| Tool/Agent | 호출 인자 유효성, 실행 성공률, 완료율 |
| Text2SQL | 실행 성공률 + 정답 결과와의 일치율 |
| 보안 | 금지된 기능·데이터에 접근하려는 테스트 차단 여부 |
| 운영 | 오류율, P95 지연시간, 토큰 및 요청당 비용 |

평가 데이터에는 정상·경계·오류·권한 없는 요청을 포함하고, **모델·Prompt·도구 변경 전후에 같은 테스트**를 실행한다.

## 8. 운영에서 확인할 것

- **Reliability:** Timeout, 제한된 재시도, 지수 백오프, 필요 시 Fallback.
- **Observability:** 요청 ID로 API → 검색 → 모델 → Tool 실행을 추적. 민감정보는 로그에서 제외.
- **Cost:** 입력/출력 토큰, 모델 선택, Context 크기, 중복 호출, 사용자별 예산 한도 확인.
- **Security:** 인증/인가, 도구 최소 권한, 감사 로그, 승인 절차.
- **Release:** 모델/Prompt/검색 인덱스 버전 관리, 평가 통과 후 배포, 문제 발생 시 되돌리기.

성능·비용을 측정하지 않았다면 개선 효과를 숫자로 주장하지 않는다.

## 9. 실무 적용 우선순위

1. **FastAPI + LLM SDK:** 호출, JSON 응답, Timeout 및 기본 테스트.
2. **Tool Calling:** 권한 검사 후 읽기 전용 API/DB 조회.
3. **RAG 또는 Text2SQL:** 검색/SQL 결과를 정답 데이터로 평가.
4. **LangGraph:** 오류 재시도·승인·상태 복구가 실제로 필요할 때 적용.
5. **운영 품질:** 로그, 요청별 비용, 회귀 평가, 보안 검증을 추가.

## 10. 암기 요약

> **LLM은 생성한다 → RAG는 근거를 가져온다 → Tool Calling은 기능 실행을 요청한다 → 서버는 권한과 결과를 검증한다 → Agent/Workflow는 순서를 제어한다 → Evaluation은 품질을 증명한다.**

다음 단계: [AI 서비스 개발 면접 질문과 30초 답변]({% post_url 2026-10-09-ai-service-engineering-interview-questions %})

## 공식 참고 자료

- [OpenAI Function calling](https://developers.openai.com/api/docs/guides/function-calling)
- [OpenAI Structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs)
- [OpenAI Retrieval](https://developers.openai.com/api/docs/guides/retrieval)
- [OpenAI Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
- [LangGraph Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)
- [MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP LLM06:2025 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
