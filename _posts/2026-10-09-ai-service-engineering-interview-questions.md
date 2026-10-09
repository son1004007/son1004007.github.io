---
layout: post
title: "AI 서비스 개발 면접 질문 10개 - 핵심 키워드와 30초 답변"
description: "RAG, Tool Calling, MCP, Agent, LangGraph, 보안과 운영을 주어가 분명한 짧은 면접 답변으로 암기합니다."
date: 2026-10-09
categories: [career]
tags: [AI, LLM, Interview, RAG, MCP, LangGraph, Tool Calling, FastAPI, Security, Study]
---

**답변 공식:** 질문마다 **핵심 정의 1문장 + 동작 원리 1문장 + 적용 기준 1문장**을 말한다. 각 문장은 실행 주체를 명확하게 표현한다.

함께 보기: [한 줄 용어 암기 노트]({% post_url 2026-10-09-ai-agent-core-terms-memorization %}) · [AI 서비스 핵심 원리]({% post_url 2026-10-09-ai-service-engineering-core-principles %})

## 1. RAG는 무엇이고 언제 사용하나요?

**암기 키워드:** 검색 → Context 제공 → 근거 기반 응답

> RAG는 외부 자료를 검색해 LLM에 근거를 제공하는 방식입니다. 검색기는 질문과 관련된 문서를 찾고, 애플리케이션은 검색 결과를 LLM의 Context로 전달합니다. 개발자는 최신 문서나 사내 지식 기반 서비스에 RAG를 적용하고 검색 정확도와 답변 근거를 평가합니다.

## 2. LLM이 Tool을 호출하는 과정은 어떻게 되나요?

**암기 키워드:** Tool 정의 → LLM 요청 → 서버 검증·실행 → 결과 전달

> 애플리케이션은 Tool의 기능과 인자 스키마를 LLM에 제공합니다. LLM은 사용할 Tool과 인자를 지정하고, 애플리케이션 서버는 사용자 권한과 인자를 검증한 뒤 Tool을 실행합니다. 서버는 실행 결과를 LLM에 전달하고, LLM은 그 결과로 최종 응답을 생성합니다.

## 3. MCP와 REST API는 무엇이 다른가요?

**암기 키워드:** 도구 연결 표준 / 업무 기능 API

> REST API는 HTTP 기반으로 서비스의 기능과 데이터를 제공합니다. MCP는 AI 애플리케이션이 외부 도구와 데이터를 발견하고 사용하는 방식을 표준화합니다. MCP Server는 기존 REST API를 호출하는 Tool을 제공할 수 있습니다.

## 4. LangChain과 LangGraph는 각각 언제 사용하나요?

**암기 키워드:** 모델·도구 통합 / 상태·흐름 제어

> LangChain은 모델과 Tool을 연결해 LLM 애플리케이션과 Agent를 구성하는 프레임워크입니다. LangGraph는 Node, Edge, State로 작업의 분기와 반복, 상태 저장과 재개를 관리합니다. 개발자는 모델·도구 연결에는 LangChain을, 상태 기반 다단계 실행에는 LangGraph를 검토합니다.

## 5. Workflow와 Agent의 차이는 무엇인가요?

**암기 키워드:** 개발자 정의 흐름 / LLM의 동적 선택

> Workflow에서는 개발자가 작업 순서와 분기 조건을 정의합니다. Agent에서는 LLM이 다음 행동이나 Tool 사용을 동적으로 판단하고, 실행 환경이 해당 작업을 수행합니다. 개발자는 처리 절차가 명확한 기능에는 Workflow를, 유연한 도구 선택이 필요한 기능에는 Agent를 적용합니다.

## 6. LLM 응답의 정확성을 어떻게 검증하나요?

**암기 키워드:** 구조 검증 → 사실 확인 → 업무 규칙 검증

> 애플리케이션은 Structured Output과 스키마 검사로 응답 구조를 검증합니다. 검증 서비스는 검색 문서, DB 결과, 정답 데이터와 응답 내용을 비교합니다. 개발자는 실제 업무 조건을 포함한 평가 데이터셋으로 모델과 Prompt 변경 전후의 품질을 측정합니다.

## 7. AI Agent의 Tool 실행 권한을 어떻게 통제하나요?

**암기 키워드:** 최소 권한 → 서버 인가 → 사용자 승인 → 감사 로그

> 백엔드 서버는 사용자별로 허용된 Tool과 데이터 범위를 정하고 실행할 때마다 권한을 검증합니다. 서버는 중요 변경 작업에 별도의 승인 절차를 적용하고 실행 이력을 감사 로그에 기록합니다. 애플리케이션은 외부 문서와 Tool 결과를 데이터 영역으로 처리해 Prompt Injection 위험을 통제합니다.

## 8. AI 서비스 운영에서는 어떤 항목을 관리하나요?

**암기 키워드:** 품질 → 지연·비용 → 장애 대응 → 보안

> AI 서비스 운영자는 응답 정확도와 품질을 평가 데이터셋으로 관리합니다. API 서버는 모델 지연, Timeout, Tool 실행 오류와 토큰 비용을 계측하고 재시도 및 Fallback 정책을 적용합니다. 운영자는 모델과 Prompt를 변경할 때 회귀 테스트를 실행하고 사용자별 권한과 비용 한도를 관리합니다.

## 9. RAG와 Fine-tuning의 차이는 무엇인가요?

**암기 키워드:** 외부 근거 보강 / 모델 가중치 조정

> RAG는 외부 문서를 검색해 LLM의 입력에 관련 정보를 추가합니다. Fine-tuning은 학습 데이터로 모델의 가중치를 조정합니다. 개발자는 최신 문서 검색과 출처 제시에 RAG를 사용하고, 특정 작업의 출력 패턴 개선에 Fine-tuning을 검토합니다.

## 10. Text2SQL의 실행 안전성은 어떻게 확보하나요?

**암기 키워드:** 사용자 권한 → SQL 검증 → 최소 권한 조회 → 결과 검증

> 인증·인가 서비스는 사용자가 조회할 수 있는 데이터 범위를 결정합니다. SQL 검증기는 LLM이 생성한 SQL의 구문, 허용 테이블, 조회 전용 정책을 확인하고, DB 서비스는 최소 권한 계정과 시간·건수 제한을 적용해 조회합니다. 결과 검증기는 조회 결과를 정답 데이터와 업무 규칙에 대조합니다.

---

## 실무 기술 선택 5초 암기표

| 요구사항 | 적용 기술과 역할 |
|---|---|
| 모델 응답 생성 | 모델 SDK는 애플리케이션과 LLM을 연결한다. |
| 외부 문서 기반 답변 | RAG는 관련 문서를 검색해 모델 입력을 보강한다. |
| API·DB 조회 | Tool Calling은 모델의 함수 호출 요청을 서버에 전달한다. |
| AI 클라이언트의 공통 Tool | MCP는 도구를 표준 인터페이스로 제공한다. |
| 실행 분기·상태 복구 | LangGraph는 다단계 실행의 상태와 흐름을 관리한다. |
| 응답 구조 고정 | Structured Output은 모델의 출력 형식을 지정한다. |
| 품질 비교 | Evaluation은 정답 데이터셋으로 결과를 측정한다. |
| 자체 모델 운영 | vLLM은 모델 추론과 API 서빙을 수행한다. |
| Agent 권한·실행 관리 | Harness는 Agent의 도구 실행과 상태를 관리한다. |

## 면접 답변 작성 기준

- **개념과 경험:** 지원자는 개념을 설명한 뒤 본인이 직접 구현·검증한 사례를 별도로 말한다.
- **성과:** 지원자는 측정 기록으로 확인한 성능과 정확도 수치만 사용한다.
- **기술 선택:** 지원자는 해당 기술을 선택한 요구사항과 판단 근거를 제시한다.
- **보안:** 지원자는 사용자 권한과 실행 승인 기능을 서버 코드에서 어떻게 구현했는지 설명한다.

## 복습 방법

1. **1차:** 학습자는 질문을 읽고 암기 키워드 3개를 말한다.
2. **2차:** 학습자는 각 질문에 20~30초 동안 답한다.
3. **3차:** 학습자는 직접 구현한 코드와 테스트 사례를 연결해 답변을 확장한다.

## 공식 참고 자료

- [OpenAI Function calling](https://developers.openai.com/api/docs/guides/function-calling)
- [OpenAI Structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs)
- [OpenAI Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP LLM06:2025 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
