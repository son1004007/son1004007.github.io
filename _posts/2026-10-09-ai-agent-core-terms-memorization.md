---
layout: post
title: "AI 개발 핵심 용어 암기 노트 - LLM, MCP, LangGraph, Harness"
description: "Prompt, Context, Skill, MCP, LangChain, LangGraph, vLLM, Harness의 의미와 사용 기준을 주어가 분명한 한 줄 정의로 암기합니다."
date: 2026-10-09
categories: [career]
tags: [AI, LLM, Agent, MCP, LangChain, LangGraph, vLLM, Harness, Study]
---

이 문서는 AI 서비스 개발에 필요한 용어를 **주어 + 동작 + 목적**이 분명한 문장으로 정리한다.

## 1. 핵심 용어 한 줄 정의

| 용어 | 암기할 정의 |
|---|---|
| LLM (Large Language Model) | LLM은 입력된 문맥을 바탕으로 언어를 처리하고 생성하는 대규모 언어 모델이다. |
| Prompt | Prompt는 사용자가 LLM에 제공하는 질문, 지시, 예시 등의 입력이다. |
| Context | Context는 LLM이 응답 생성에 참고하는 대화, 문서, 코드, 도구 결과 등의 정보다. |
| Context Window | Context Window는 모델이 한 번의 처리에서 다룰 수 있는 토큰의 최대 범위다. |
| Token | Token은 모델이 텍스트를 처리할 때 사용하는 기본 단위다. |
| Agent | Agent는 LLM의 판단과 도구 실행을 연결하여 목표를 수행하는 시스템이다. |
| Tool Calling | Tool Calling은 LLM이 도구의 이름과 인자를 지정해 애플리케이션에 실행을 요청하는 방식이다. |
| Skill | Skill은 Agent가 작업을 수행할 때 참고하는 지침, 자료, 스크립트의 묶음이다. |
| MCP (Model Context Protocol) | MCP는 AI 애플리케이션이 외부 도구와 데이터를 사용하는 방식을 표준화한 프로토콜이다. |
| RAG (Retrieval-Augmented Generation) | RAG는 외부 자료를 검색해 LLM에 근거를 제공하고 답변을 생성하는 방식이다. |
| LangChain | LangChain은 LLM, 도구, Agent를 조합해 AI 애플리케이션을 개발하는 프레임워크다. |
| LangGraph | LangGraph는 Agent의 실행 흐름, 분기, 상태 저장과 재개를 관리하는 프레임워크다. |
| vLLM | vLLM은 LLM을 효율적으로 실행하고 API로 제공하는 추론·서빙 엔진이다. |
| Agent Harness | Agent Harness는 Agent의 모델 호출, 도구 실행, 상태 및 권한을 관리하는 실행 환경이다. |
| Harness Engineering | Harness Engineering은 Agent의 실행 환경, 작업 규칙, 테스트와 검증 체계를 설계·개선하는 활동이다. |

## 2. 비슷한 용어 구별하기

| 구분 | 핵심 차이 |
|---|---|
| Prompt / Context | Prompt는 LLM에 질문과 지시를 전달한다. Context는 LLM에 판단 근거를 제공한다. |
| Skill / MCP | Skill은 Agent에 작업 절차를 제공한다. MCP는 AI 애플리케이션에 외부 도구 연결 규약을 제공한다. |
| Tool Calling / MCP | Tool Calling은 모델의 도구 실행 요청을 표현한다. MCP는 클라이언트와 도구 서버의 통신을 표준화한다. |
| LangChain / LangGraph | LangChain은 LLM 앱 구성 요소를 통합한다. LangGraph는 여러 작업의 흐름과 상태를 제어한다. |
| LangGraph / Harness | LangGraph는 상태 기반 워크플로를 관리한다. Harness는 Agent의 전체 실행 환경을 관리한다. |
| LLM / vLLM | LLM은 응답을 생성한다. vLLM은 모델의 추론 요청을 처리하고 결과를 제공한다. |

## 3. 언제 사용하는가?

| 요구사항 | 선택할 기술과 이유 |
|---|---|
| 사용자 질문을 모델에 전달 | 애플리케이션은 모델 SDK로 LLM을 호출한다. |
| 기존 문서를 근거로 답변 생성 | RAG는 관련 문서를 검색해 Context를 보강한다. |
| API 또는 DB 기능 사용 | Tool Calling은 모델의 기능 호출 요청을 전달한다. |
| 여러 AI 클라이언트에 외부 도구 제공 | MCP Server는 도구와 데이터를 표준 인터페이스로 제공한다. |
| 반복 작업 지침 재사용 | Skill은 Agent에 공통 절차와 참고 자료를 제공한다. |
| 모델과 도구 결합 | LangChain은 LLM 애플리케이션의 구성 요소를 연결한다. |
| 단계별 분기·반복·상태 복구 | LangGraph는 워크플로의 실행 흐름과 상태를 관리한다. |
| Agent 실행 및 보안 통제 | Harness는 모델 호출, 도구 실행, 권한과 작업 상태를 관리한다. |
| 자체 LLM 서버 구축 | vLLM은 GPU 등의 연산 환경에서 모델 추론을 수행한다. |

## 4. 암기 점검

1. **LLM에 질문과 지시를 전달하는 입력은?** → Prompt
2. **LLM에 응답 생성의 참고 정보를 제공하는 것은?** → Context
3. **AI 애플리케이션과 외부 도구 사이의 연결을 표준화하는 것은?** → MCP
4. **Agent에 재사용할 작업 절차를 제공하는 것은?** → Skill
5. **LLM이 외부 기능의 실행을 요청하는 방식은?** → Tool Calling
6. **Agent의 실행 흐름과 상태를 관리하는 것은?** → LangGraph
7. **자체 LLM의 추론과 API 서빙을 담당하는 것은?** → vLLM
8. **Agent의 실행 루프와 권한을 관리하는 환경은?** → Agent Harness

## 5. 다음 단계 학습

- [AI 서비스 핵심 원리 - RAG, Tool Calling, Agent, 보안과 평가]({% post_url 2026-10-09-ai-service-engineering-core-principles %})
- [AI 서비스 개발 면접 질문 10개 - 핵심 키워드와 30초 답변]({% post_url 2026-10-09-ai-service-engineering-interview-questions %})

## 6. 공식 참고 자료

- [MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [LangChain Overview](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [vLLM Documentation](https://docs.vllm.ai/en/stable/)
- [OpenAI Harness Engineering](https://openai.com/index/harness-engineering/)
