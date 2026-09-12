---
layout: post
title: "Secure Agent Platform을 백엔드보다 HTML 프로토타입부터 만드는 이유"
description: "서비스 목표를 고정하지 않고, 완성형 HTML 퍼블리싱 시안으로 화면과 사용자 흐름을 먼저 검증한 뒤 API 설계, 백엔드 구현, NAS 배포로 이어가는 개발 순서를 정리합니다."
date: 2026-09-13
categories: [career]
tags: [AI Agent, Product Design, HTML Prototype, FastAPI, API Design, CLI, GUI, Synology, Portfolio]
---

앞서 OpenAI Agents API를 활용한 `Secure Agent Platform Lab`을 설계하면서 FastAPI, 보안 정책, Human Approval, Observability, 장애 대응, Kubernetes 같은 기술 목표를 정리했다.

관련 글: [OpenAI Agents API로 Secure Agent Platform을 설계하는 이유와 구현 로드맵]({% post_url 2026-09-12-openai-agents-api-secure-agent-platform-roadmap %})

그런데 계획을 구체화할수록 한 가지 문제가 보였다. 기술 구성은 설명할 수 있는데 **완성된 서비스를 실제 사용자가 어떻게 쓰는지 한눈에 설명하기 어려웠다.**

이 상태에서 API와 백엔드부터 만들면 구현한 구조에 맞춰 서비스를 끼워 맞추기 쉽다. 그래서 개발 순서를 바꾸기로 했다.

## 현재 서비스 목표는 고정하지 않는다

현재 working hypothesis는 다음과 같다.

> NAS에서 상시 실행되며, Web GUI 또는 CLI를 통해 AI Agent에게 장애 조사와 보안 검토를 요청하고, Agent가 확인한 근거와 진행 상태를 보고, 위험한 작업은 사람이 명시적으로 승인하는 Agent 운영 플랫폼

하지만 이것을 최종 요구사항으로 고정하지 않는다.

HTML 시안을 보고 직접 사용 흐름을 검토했을 때 Incident Investigation이 핵심 기능으로 어색하거나, Security Review가 더 자연스럽거나, 다른 사용 시나리오가 더 가치 있다고 판단되면 **백엔드 구현 전에 목표를 변경한다.**

고정할 것은 특정 화면이 아니라 다음 engineering goals이다.

- AI Backend와 Agent integration
- 인증, 권한, 정책, Human Approval
- Observability와 Incident Response
- 성능과 장애 대응 증거
- Docker 기반 NAS 운영과 이후 Kubernetes 확장
- API, CLI, GUI가 동일한 기능과 보안 경로를 사용하는 구조

## 개발 순서를 바꾼다

이제 다음 순서로 진행한다.

```text
완성형 정적 HTML 퍼블리싱 시안
 -> 화면 / 사용자 흐름 / 기능 검토
 -> 서비스 목표 유지 또는 수정
 -> 확정된 사용자 행동 기준 API 설계
 -> 실제 Backend 구현
 -> CLI / GUI를 동일 API에 연결
 -> NAS 배포
 -> 사람 / 다른 AI / 자동화 클라이언트로 반복 테스트
```

핵심은 **화면을 백엔드 결과물의 껍데기로 만들지 않는 것**이다.

먼저 완성된 서비스처럼 보이는 정적 화면을 만든 뒤, 그 화면에서 실제로 필요한 사용자 행동만 남긴다.

## 먼저 만들 HTML 퍼블리싱 작업물

공개 GitHub 저장소의 `prototype/` 아래에 Backend나 OpenAI API 없이 열 수 있는 정적 HTML/CSS/JavaScript 시안을 만든다.

현재 예상하는 화면은 다음과 같다.

| 화면 | 사용자가 확인하거나 수행할 일 |
|---|---|
| Dashboard | 시스템 상태, 실행 중 Agent, 승인 대기, 최근 조사 결과 확인 |
| Sessions | Agent 작업 목록과 상태 확인 |
| New Agent Task | 장애 조사 또는 보안 검토 요청 |
| Session Detail | Agent 진행 과정, 확인 근거, 판단, 권장 조치 확인 |
| Approvals | 위험 작업 승인 또는 거부 |
| Incidents | Incident Drill 조사 및 결과 기록 확인 |
| Security Review | Repository/Container/Kubernetes 보안 검토 결과 확인 |
| System | API, Provider, DB 등 Runtime 상태 확인 |

화면은 Desktop뿐 아니라 모바일에서도 사용할 수 있게 만든다. 실제로 NAS에 올린 뒤 휴대폰에서 확인하고 조작하는 상황을 염두에 둔다.

다만 이 단계의 숫자, 장애 상태, Agent 결과는 모두 **Mock/Sample Data**다. 공개 포트폴리오에서 실제 운영 결과처럼 보이지 않도록 명확히 구분한다.

## 가장 중요한 화면은 Session Detail이다

서비스의 정체성이 가장 잘 드러나야 하는 화면은 Agent 작업 상세다.

예를 들어 사용자가 다음 작업을 요청했다고 가정한다.

```text
최근 30분 동안 API 5xx가 증가한 원인을 조사하고 대응안을 정리해줘.
```

화면에서는 단순히 AI 답변 한 문장을 보여주는 대신 다음과 같이 표현한다.

```text
Metrics analysis       COMPLETE
Log analysis           COMPLETE
Deployment analysis    COMPLETE
Dependency analysis    RUNNING

Finding
DB connection pool exhaustion suspected

Evidence
API latency increase
DB timeout increase
recent deployment unchanged

Recommended action
Restart application

Approval required
[Approve] [Reject]
```

Agent의 내부 Chain of Thought를 보여주는 것이 목적이 아니다.

사용자가 볼 것은 다음이다.

- Agent가 어떤 작업을 수행했는가
- 어떤 근거를 확인했는가
- 현재 상태는 무엇인가
- 어떤 결론을 냈는가
- 어떤 조치를 권장하는가
- 그 조치가 자동 실행 가능한가, 승인해야 하는가

이렇게 해야 일반 챗봇이 아니라 **통제 가능한 Agent 운영 서비스**라는 차이가 보인다.

## Human Approval도 화면에서 먼저 검증한다

Human Approval은 나중에 붙이는 보안 기능이 아니라 서비스의 핵심 UX로 본다.

예를 들어 Agent가 서비스 재시작을 요청한다면 사용자는 최소한 다음을 확인할 수 있어야 한다.

```text
요청 Agent
요청 작업
요청 이유
근거
Risk level
예상 영향

[Approve] [Reject]
```

정적 HTML 단계에서 이 흐름이 어색하다면 API와 데이터 모델도 수정해야 한다.

즉 화면 설계가 실제 Backend Domain 설계에 영향을 주게 한다.

## HTML 검토 후 API를 설계한다

기존에는 `/api/v1/sessions`, `/approvals` 같은 Endpoint를 먼저 생각했다.

이제는 순서를 반대로 한다.

```text
사용자가 화면에서 무엇을 하는가?
 -> 그 행동에 필요한 데이터는 무엇인가?
 -> 하나의 API operation으로 어떻게 표현할 것인가?
 -> CLI에서도 같은 operation을 어떻게 사용할 것인가?
```

예를 들어 화면 검토 후 다음 행동이 유지됐다고 가정하면 그때 API를 확정한다.

```text
Dashboard status       -> GET  /api/v1/system/summary
List sessions          -> GET  /api/v1/sessions
Create session         -> POST /api/v1/sessions
Session detail         -> GET  /api/v1/sessions/{id}
Watch events           -> GET  /api/v1/sessions/{id}/events
Approval decision      -> POST /api/v1/approvals/{id}/decision
```

이 목록 역시 현재는 예시다. HTML 검토 결과 필요 없는 기능은 제거하고, 빠진 기능은 추가한다.

## CLI도 같은 설계를 사용한다

CLI는 별도 서비스가 아니다.

화면에서 확정된 기능을 API로 만들고, 같은 기능을 `sapctl`에서 호출한다.

```text
sapctl system status
sapctl session list
sapctl session create --message "..."
sapctl session show <id>
sapctl session watch <id>
sapctl approval list
sapctl approval approve <id>
sapctl approval reject <id>
```

따라서 최종 구조는 다음과 같다.

```text
Accepted user action
        |
        v
FastAPI REST/SSE
   |        |        |
   v        v        v
Browser   sapctl   AI/Automation
```

GUI와 CLI에서 서로 다른 정책이나 로직이 동작하지 않게 하는 것이 중요하다.

## NAS 구현도 한 단계 뒤로 미룬다

Synology NAS를 상시 Integration/Test Runtime으로 사용하는 방향은 유지한다.

다만 NAS에 기능을 하나씩 올리기 전에 HTML 프로토타입을 먼저 완성한다.

수정된 단계는 다음과 같다.

```text
Phase 0     FastAPI / Test / Docker baseline                  완료
Phase 0.25  Product definition + HTML publishing prototype    다음 단계
Phase 0.5   NAS Compose + API / CLI / GUI baseline
Phase 1     OpenAI Agents API minimal E2E
Phase 2     JWT/OIDC + RBAC + Human Approval
Phase 3     Observability
Phase 4     Incident Investigation
Phase 5     Reliability / Performance
Phase 6     Kubernetes
Phase 7     Kafka Event Architecture
Phase 8     Security Review Agent
```

Phase 0.25가 끝나기 전에는 이후 Backend Endpoint를 필요 이상으로 확정하지 않는다.

## HTML 단계에서 확인할 질문

퍼블리싱 결과를 보면서 다음 질문에 답한다.

1. Dashboard와 Session Detail만 보고 이 서비스가 무엇인지 설명할 수 있는가?
2. 사용자가 가장 먼저 해야 할 행동이 명확한가?
3. 조사와 실제 변경 작업이 명확하게 구분되는가?
4. Human Approval의 필요성을 기술 지식 없이도 이해할 수 있는가?
5. 화면에서 하는 행동을 CLI 명령으로도 자연스럽게 표현할 수 있는가?
6. 실제 사용자 판단에 필요하지 않은 화면이나 정보는 없는가?
7. Incident Investigation이 정말 첫 번째 핵심 서비스인가?
8. 지금의 서비스 목표를 그대로 유지할 것인가, 변경할 것인가?

이 질문에 답한 뒤에만 다음 API와 Backend 범위를 확정한다.

## 이번 변경에서 얻고 싶은 것

이번 프로젝트에서 증명하고 싶은 것은 단순히 기술을 많이 연결했다는 사실이 아니다.

```text
서비스 목적 정의
 -> 사용자 흐름 설계
 -> API Boundary 결정
 -> Backend Architecture
 -> Security Policy
 -> 운영 / 관측 / 장애 대응
 -> 실제 배포와 검증
```

이라는 전체 의사결정 과정을 공개 증거로 남기는 것이 목표다.

그래서 현재 목표가 나중에 바뀌더라도 그것을 실패로 보지 않는다. **프로토타입과 검증 근거를 통해 더 나은 서비스 방향으로 변경했다면 그 판단 과정 자체가 Architecture와 Product Engineering의 근거**가 된다.
