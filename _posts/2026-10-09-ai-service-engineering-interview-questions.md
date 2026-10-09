---
layout: post
title: "AI 서비스 개발 면접 질문 10개 - 핵심 키워드와 30초 답변"
description: "RAG, Tool Calling, MCP, Agent, LangGraph, 평가, 보안, 운영을 개발자 면접에서 30초 내 설명할 수 있도록 질문·키워드·짧은 답변으로 정리합니다."
date: 2026-10-09
categories: [career]
tags: [AI, LLM, Interview, RAG, MCP, LangGraph, Tool Calling, FastAPI, Security, Study]
---

**목표:** 정의만 외우지 말고 각 질문에 **핵심 1문장 + 처리 흐름 1문장 + 주의점 1문장**으로 답한다.

함께 보기: [한 줄 용어 암기 노트]({% post_url 2026-10-09-ai-agent-core-terms-memorization %}) · [AI 서비스 핵심 원리]({% post_url 2026-10-09-ai-service-engineering-core-principles %})

## 1. RAG는 무엇이고 언제 사용하나요?

**암기 키워드:** 문서 검색 → Context 보강 → 근거 기반 답변

> RAG는 외부 자료를 검색해 관련 내용을 LLM의 입력에 추가하고 답변을 생성하는 방식입니다. 최신 문서나 사내 지식을 근거로 응답해야 할 때 사용합니다. 검색된 자료가 정확하지 않으면 답변도 틀릴 수 있어 검색 품질과 답변의 근거를 별도로 평가해야 합니다.

## 2. LLM이 Tool을 호출하는 과정은 어떻게 되나요?

**암기 키워드:** 도구 정의 → 호출 요청 → 서버 검증·실행 → 결과 전달

> 애플리케이션이 도구의 이름과 인자 스키마를 제공하면 LLM이 사용할 도구와 인자를 반환할 수 있습니다. 실제 실행은 서버가 담당하며, 인자 유효성·접근 권한을 검사한 뒤 결과를 모델에 전달합니다. 모델이 도구를 선택했다고 무조건 실행하면 안 됩니다.

## 3. MCP와 REST API는 무엇이 다른가요?

**암기 키워드:** 표준 도구 연결 vs 업무 API

> REST API는 일반적으로 HTTP 자원을 통해 서비스 기능을 제공하는 인터페이스입니다. MCP는 AI 애플리케이션이 외부 도구와 데이터를 발견하고 사용할 수 있도록 정의한 표준 프로토콜입니다. MCP Server가 기존 REST API를 내부적으로 호출할 수 있으며, MCP를 반드시 써야 Tool Calling이 가능한 것은 아닙니다.

## 4. LangChain과 LangGraph는 각각 언제 사용하나요?

**암기 키워드:** 구성 요소 통합 vs 흐름·상태 제어

> LangChain은 모델과 도구, Agent를 조합해 LLM 애플리케이션을 개발할 때 사용합니다. LangGraph는 작업 단계, 조건 분기, 반복 검증, 상태 보존과 재개를 명시적으로 관리할 때 적합합니다. 단순한 모델 호출은 SDK만으로 구현할 수 있으므로 복잡도에 따라 선택합니다.

## 5. Workflow와 Agent의 차이는 무엇인가요?

**암기 키워드:** 정해진 순서 vs 모델의 동적 판단

> Workflow는 개발자가 작업 순서와 분기 조건을 정의하고, Agent는 LLM이 다음 작업이나 도구 사용을 동적으로 판단하도록 구성합니다. 처리 절차가 명확하면 일반 Workflow가 제어와 테스트에 유리합니다. Agent는 유연성이 필요할 때 적용하되 도구 권한과 실행 횟수를 제한해야 합니다.

## 6. LLM의 잘못된 응답은 어떻게 검증하나요?

**암기 키워드:** 형식 → 사실 → 업무 규칙

> 먼저 Structured Output과 스키마 검증으로 응답 형식을 확인합니다. 그다음 DB 결과, 검색 근거 또는 정답 데이터와 비교해 사실 정확성과 업무 규칙 충족 여부를 확인합니다. JSON 형식이 맞는 것과 응답 내용이 맞는 것은 별개이므로 회귀 평가 데이터셋을 운영합니다.

## 7. AI Agent가 허용되지 않은 작업을 못 하게 하려면?

**암기 키워드:** 최소 권한 → 서버 인가 → 승인 → 감사 로그

> Tool을 필요한 기능만 노출하고 호출자의 권한을 서버에서 확인합니다. 중요한 변경이나 외부 전송은 별도 승인 절차를 적용하며, 호출 내역을 감사 로그에 남깁니다. Prompt Injection을 고려해 검색 문서나 Tool 결과에 포함된 지시를 신뢰하지 않습니다.

## 8. 일반 백엔드와 AI 서비스 운영은 무엇이 다른가요?

**암기 키워드:** 비결정성 → 품질 평가 → 비용·지연 → 보안

> 일반적인 API 검증에 더해 AI 응답의 비결정성과 품질을 관리해야 합니다. 모델 오류, 지연, 토큰 비용, Tool 실행 실패를 측정하고 Timeout, 제한된 재시도와 Fallback 정책을 설계합니다. Prompt나 모델을 변경할 때도 평가 데이터로 회귀 테스트를 해야 합니다.

## 9. RAG와 Fine-tuning의 차이는 무엇인가요?

**암기 키워드:** 근거 추가 vs 가중치 조정

> RAG는 검색한 자료를 모델 입력에 추가하는 방식이고, Fine-tuning은 학습을 통해 모델의 가중치를 조정하는 방식입니다. 최신 문서 조회나 출처가 중요한 서비스라면 보통 RAG를 먼저 검토합니다. 특정 출력 패턴을 지속적으로 개선해야 한다면 Fine-tuning을 검토할 수 있습니다.

## 10. Text2SQL에서 가장 중요한 안전 장치는 무엇인가요?

**암기 키워드:** 권한 → SQL 검증 → 읽기 전용 → 결과 대조

> LLM이 생성한 SQL은 실행 전에 구문 구조, 허용 테이블, 조회 권한을 검사해야 합니다. DB에는 최소 권한 계정을 사용하고 시간·조회 건수를 제한합니다. SQL 실행 성공만으로 정답이라고 판단하지 않고 예상 결과 및 업무 조건과 비교합니다.

---

## 실무 기술 선택 5초 암기표

| 요구사항 | 먼저 검토할 기술·방법 |
|---|---|
| 단순 LLM 호출 | 모델 SDK |
| 문서 기반 답변 | RAG |
| API·DB 기능 호출 | Tool Calling (+ 필요하면 MCP) |
| 분기·반복·상태 복구 | LangGraph |
| 응답 형식 고정 | Structured Output |
| 모델/Prompt 품질 비교 | Evaluation 데이터셋 |
| 자체 오픈 모델 서빙 | vLLM |
| Agent 실행·승인·도구 통제 | Harness와 백엔드 보안 통제 |

## 실제 면접 답변 시 주의점

- **개념 설명과 실무 경험을 분리한다.** 학습·실습 내용은 실무 적용 경험인 것처럼 말하지 않는다.
- **수치와 성과는 검증한 값만 말한다.** 임의의 정확도나 성능 개선율을 만들지 않는다.
- **기술명보다 선택 이유가 중요하다.** LangChain/Graph를 왜 썼는지, SDK만으로 충분한지 판단 기준을 설명한다.
- **보안은 프롬프트가 아닌 서버가 강제한다.** 접근 권한과 변경 승인 기능은 실제 코드에서 검증한다.

## 복습 방법

1. **1차:** 질문만 보고 키워드 3개를 말한다.
2. **2차:** 각 질문을 20~30초 내에 설명한다.
3. **3차:** 실제 구현한 코드·테스트·장애 사례가 있는 질문만 추가 경험 답변으로 확장한다.

## 공식 참고 자료

- [OpenAI Function calling](https://developers.openai.com/api/docs/guides/function-calling)
- [OpenAI Structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs)
- [OpenAI Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
- [LangGraph Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)
- [MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP LLM06:2025 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
