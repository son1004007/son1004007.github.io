---
layout: post
title: "AI 개발 핵심 용어 암기 노트 - LLM, MCP, LangGraph, Harness"
description: "Prompt, Context, Skill, MCP, LangChain, LangGraph, vLLM, Harness 등 AI 개발 용어를 한 줄 정의와 차이점으로 빠르게 복습합니다."
date: 2026-10-09
categories: [career]
tags: [AI, LLM, Agent, MCP, LangChain, LangGraph, vLLM, Harness, Study]
---

외울 때는 **용어 → 한 줄 정의 → 비슷한 용어와의 차이** 순서로 확인한다.

## 1. 핵심 용어

| 용어 | 한 줄 정의 |
|---|---|
| LLM (Large Language Model) | 언어를 이해하고 생성하도록 학습된 대규모 모델 |
| Prompt | LLM에 전달하는 입력(질문, 지시, 예시 등) |
| Context | LLM이 응답을 생성할 때 참고하도록 제공된 정보 |
| Context Window | 모델이 한 번의 처리에서 다룰 수 있는 토큰 범위 |
| Token | 모델이 텍스트를 나누어 처리하는 단위 |
| Agent | LLM의 판단과 도구 실행을 결합해 목표를 수행하는 시스템 |
| Tool Calling | LLM이 외부 도구 호출을 요청하고 실행 결과를 받는 방식 |
| Skill | 재사용할 작업 지침, 참고 자료, 스크립트 등의 묶음 |
| MCP (Model Context Protocol) | AI 애플리케이션과 외부 도구·데이터를 연결하는 표준 프로토콜 |
| RAG (Retrieval-Augmented Generation) | 외부 자료를 검색해 LLM 입력에 보강한 뒤 답변을 생성하는 방식 |
| LangChain | LLM, 도구, Agent 등을 조합하는 애플리케이션 프레임워크 |
| LangGraph | 상태, 분기, 반복, 재개를 제어하는 Agent 워크플로 프레임워크 |
| vLLM | LLM을 효율적으로 추론하고 API로 제공하는 서빙 엔진 |
| Agent Harness | Agent의 실행 루프, 도구 사용, 권한 및 상태를 관리하는 실행 환경 |
| Harness Engineering | Agent가 안정적으로 작업하도록 실행 환경과 검증 체계를 설계·개선하는 활동 |

## 2. 헷갈리기 쉬운 차이

- **Prompt vs Context**: 무엇을 요청하는가 vs 무엇을 참고하는가. Prompt도 Context의 일부가 될 수 있다.
- **Skill vs MCP**: 어떻게 작업하는가 vs 외부 기능에 어떻게 연결하는가.
- **LangChain vs LangGraph**: LLM 앱 구성 vs 복잡한 실행 흐름·상태 제어.
- **LangGraph vs Harness**: 워크플로 제어 프레임워크 vs Agent 전체 실행 환경. LangGraph를 Harness의 구성 요소로 쓸 수 있다.
- **LLM vs vLLM**: 답을 생성하는 모델 vs 그 모델을 실행·서빙하는 엔진.

## 3. 언제 쓰는가

| 목적 | 우선 검토할 방법 |
|---|---|
| LLM에 질문하고 응답 받기 | 모델 API/SDK 직접 호출 |
| 여러 모델·도구·Agent 조합 | LangChain |
| 조건 분기, 반복 검증, 상태 복구 | LangGraph |
| GitHub, DB 등 외부 기능을 표준 방식으로 연결 | MCP |
| Agent 작업 규칙 재사용 | Skill |
| Agent의 실행·권한·테스트 환경 관리 | Harness |
| 자체 모델 서버 운영 | vLLM |

**주의:** 모두 필수 구성 요소는 아니다. 단순한 기능은 FastAPI와 모델 SDK만으로도 구현할 수 있다. vLLM은 GPT나 Claude의 비공개 모델을 직접 실행하는 방법이 아니다.

## 4. 암기 점검

1. LLM에 전달하는 **지시**는? → **Prompt**
2. LLM에 제공하는 **참고 정보**는? → **Context**
3. AI와 외부 기능의 **표준 연결 규약**은? → **MCP**
4. Agent의 **작업 규칙 묶음**은? → **Skill**
5. Agent의 **분기·반복·상태 관리**는? → **LangGraph**
6. LLM의 **효율적인 추론·서빙**은? → **vLLM**
7. Agent의 **실행 환경과 제어 체계**는? → **Harness**

## 5. 공식 참고 자료

- [MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [LangChain Overview](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [vLLM Documentation](https://docs.vllm.ai/en/stable/)
- [OpenAI Harness Engineering](https://openai.com/index/harness-engineering/)
