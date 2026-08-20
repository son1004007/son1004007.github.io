---
layout: post
title: "GitHub를 Control Plane으로 사용해 ChatGPT·Codex 개발과 배포를 반자동화하는 방법"
date: 2026-08-20
categories: [infrastructure]
tags: [GitHub, GitHub Actions, ChatGPT, Codex, AI Agent, CI/CD, DevOps, Automation]
---

## 배경

ChatGPT나 Codex를 이용해 실제 프로그램을 수정하다 보면 대화 안에서 코드를 만드는 것보다 더 어려운 문제가 생긴다.

- AI가 바뀌면 이전 작업 방법을 다시 설명해야 한다.
- 모바일에서 ChatGPT를 사용하고 실제 실행 환경은 사내 Linux 서버에 있을 수 있다.
- AI가 서버에 직접 접속할 수 없는 경우가 있다.
- 코드는 수정됐지만 실제 배포가 되었는지 확인하기 어렵다.
- AI가 "정상 동작한다"고 말해도 실환경에서 확인되지 않았을 수 있다.
- 고객사가 바뀔 때마다 SSH, CI/CD, 테스트 방법을 다시 설계하게 된다.

이 문제를 해결하기 위해 AI를 실행 플랫폼으로 사용하지 않고 **GitHub를 공통 Control Plane으로 사용하는 구조**를 사용한다.

이 글의 목표는 특정 AI 제품에 종속된 자동화가 아니다. 이 글 자체를 ChatGPT, Codex, Claude, Gemini 등 repository를 다룰 수 있는 AI에게 전달했을 때 비슷한 프로젝트 운영 구조를 다시 만들 수 있도록 하는 것이다.

---

## 결론부터: AI가 아니라 GitHub를 중심에 둔다

전체 구조는 다음과 같다.

```text
사용자
  │
  ├─ ChatGPT
  ├─ Codex
  ├─ Claude / Gemini / Other AI
  │
  ▼
GitHub
  ├─ AGENTS.md          : AI 작업 규칙
  ├─ ops/agent-ops.yaml : 실행 가능한 운영 계약
  ├─ Issue              : 작업 요청과 상태 포인터
  ├─ Branch / PR        : 변경 격리와 검토
  ├─ Actions            : 테스트와 배포 실행
  └─ Secrets            : 인증정보
  │
  ▼
Runner / Jenkins / GitHub Actions
  │
  ├─ build
  ├─ test
  ├─ container build
  ├─ preview deploy
  └─ runtime verification
  │
  ▼
실행 환경
  ├─ 고객사 Linux 서버
  ├─ 사내 서버
  ├─ NAS
  └─ Cloud
  │
  ▼
검증 결과를 다시 GitHub에 기록
```

사용자가 AI와 대화하면서 하는 일은 점점 줄어든다.

```text
목표 전달
-> AI가 GitHub 수정
-> GitHub Actions 실행
-> 실제 환경 검증
-> AI가 결과 확인
-> 실패하면 수정 후 다시 실행
-> Preview 완료
-> 사용자가 Production 승인
```

핵심은 **실제 실행 결과를 GitHub에 다시 돌려주는 것**이다.

---

## 왜 GitHub가 Control Plane인가

AI 대화는 작업 상태를 보존하는 데 적합하지 않다. 새로운 대화를 시작하거나 다른 AI를 사용하면 앞선 판단과 실행 결과를 다시 설명해야 한다.

반면 GitHub에는 다음이 남는다.

| 정보 | GitHub에서의 위치 |
|---|---|
| 프로젝트 규칙 | `AGENTS.md` |
| 실행/배포 계약 | `ops/agent-ops.yaml` |
| 요구사항 | docs / Issue |
| 변경 내용 | Commit / Branch / PR |
| 테스트 | GitHub Actions |
| 배포 결과 | workflow log / deployment |
| 현재 상태 | Issue 또는 status document |

따라서 새로운 AI가 들어와도 저장소를 읽으면 현재 상태부터 다시 확인할 수 있다.

```text
새로운 AI
-> repository read
-> AGENTS.md read
-> agent-ops.yaml read
-> latest Issue / PR / workflow read
-> 현재 상태 복구
-> 작업 계속
```

---

## AI와 Executor를 분리한다

이 방식에서 AI와 실행기는 역할이 다르다.

### AI가 담당할 것

- 요구사항 정리
- 코드와 설정 조사
- 구현 계획 작성
- branch 생성
- 코드/문서 수정
- commit/push
- workflow 결과 확인
- 실패 원인 분석
- 다음 수정

### GitHub Actions / Runner가 담당할 것

- build
- unit/integration test
- Docker build
- VPN 또는 내부망 접속
- 서버 배포
- health check
- API smoke test
- DB 상태 확인

예를 들어 ChatGPT가 고객사 서버로 SSH할 수 없더라도 GitHub Actions가 서버에 접근할 수 있다면 다음 구조가 가능하다.

```text
ChatGPT
-> GitHub 파일 수정/push
-> GitHub Actions
-> 승인된 네트워크 경로
-> 고객사 서버
-> 실행 결과
-> GitHub Actions log
-> ChatGPT가 log 확인
```

즉 **GitHub를 원격 실행의 중계 계층으로 사용**한다.

---

## 최소 Repository 구조

새 프로젝트에 아래 구조를 권장한다.

```text
project/
  AGENTS.md

  ops/
    agent-ops.yaml

  docs/
    00-project-context.md
    01-requirements.md
    02-architecture.md
    03-test-plan.md
    04-operation-and-deployment.md
    06-ai-operations.md

  .github/
    workflows/
      ai-control-plane-verify.yml
      preview-deploy.yml
      production-deploy.yml
```

프로젝트가 작다면 문서 수는 줄일 수 있다. 그러나 다음 세 개는 유지하는 편이 좋다.

```text
AGENTS.md
ops/agent-ops.yaml
docs/04-operation-and-deployment.md
```

---

## 1. AGENTS.md: AI가 따라야 할 규칙

AI마다 프롬프트를 다시 작성하지 않고 repository 자체에 규칙을 둔다.

최소한 다음 내용을 포함한다.

```markdown
# AGENTS.md

## Required reading

작업 전에 다음 순서로 확인한다.

1. AGENTS.md
2. ops/agent-ops.yaml
3. docs/00-project-context.md
4. docs/04-operation-and-deployment.md
5. docs/06-ai-operations.md
6. 현재 Issue/PR
7. 관련 workflow와 실제 코드/테스트

## Rules

- 확인하지 않은 host, 계정, credential, 배포 경로를 추측하지 않는다.
- main에 바로 개발하지 않는다.
- 변경은 별도 branch에서 수행한다.
- 테스트하지 않은 결과를 PASS라고 보고하지 않는다.
- secret을 코드, 문서, workflow log에 출력하지 않는다.
- Preview에서 먼저 검증한다.
- Production 배포와 운영 DB 변경은 사람 승인 후 실행한다.
```

Codex는 기본적으로 `AGENTS.md` 기반 작업 규칙과 잘 맞는다. 다른 AI도 첫 요청에서 이 파일을 읽으라고 지정하면 같은 기준을 적용할 수 있다.

---

## 2. agent-ops.yaml: AI가 읽을 수 있는 운영 계약

자연어 문서만 두면 AI가 매번 운영 방식을 다시 해석해야 한다. 따라서 machine-readable 파일을 하나 둔다.

```yaml
version: 1

project:
  name: example-service
  default_branch: main

control_plane:
  provider: github
  work_item: issue
  change_method: branch-pr
  evidence_source: github-actions

branches:
  production: main
  preview_patterns:
    - "feature/*"
    - "agent/*"

workflow:
  verify: ".github/workflows/ai-control-plane-verify.yml"
  preview_deploy: ".github/workflows/preview-deploy.yml"
  production_deploy: ".github/workflows/production-deploy.yml"

runtime:
  preview:
    target_ref: customer-preview
    isolation:
      separate_port: true
      separate_container_or_namespace: true
      separate_database_or_schema: true
  production:
    target_ref: customer-production

verification:
  required: true
  health_endpoint: "/health"
  success_markers:
    - "BUILD=PASS"
    - "TEST=PASS"
    - "DEPLOY_PREVIEW=PASS"
    - "HEALTH=PASS"

status:
  method: github-issue
  issue_number: null

secrets:
  repository: github-actions-secrets
  runtime: target-host-or-secret-manager
  commit_values_to_repository: false

network:
  runner_to_target: null

safety:
  production_deploy: human-approval
  production_data_write: human-approval
  destructive_operation: human-approval
  secret_output: forbidden
```

`target_ref`는 실제 IP가 아니다. 실제 서버 주소와 credential은 GitHub Secrets, Environment, 사내 secret manager 등에 보관한다.

---

## 3. Preview와 Production을 반드시 분리한다

AI 자동화에서 가장 위험한 형태는 AI가 수정한 코드를 바로 운영 환경에 반영하는 것이다.

권장 흐름은 다음과 같다.

```text
main
  = Production 기준

feature/* 또는 agent/*
  = AI 작업 branch

AI 작업 branch push
  -> CI
  -> Preview deploy
  -> runtime validation
  -> 사람이 결과 확인
  -> Production 승인
```

실행 환경에서도 필요한 경계를 분리한다.

```text
Production
- service container
- production port
- production DB
- production persistent volume

Preview
- preview container
- preview port
- preview DB/schema
- preview persistent volume
```

특히 DB를 사용하는 서비스라면 Preview 애플리케이션만 분리하고 DB는 운영 DB를 그대로 사용하는 실수를 피해야 한다.

---

## 4. Push를 테스트 Trigger로 사용한다

AI가 branch에 push하면 CI가 자동으로 검증한다.

예:

```yaml
name: Preview Verify

on:
  push:
    branches:
      - "feature/**"
      - "agent/**"
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: ./project-build-command

      - name: Test
        run: ./project-test-command
```

위 명령은 예시다. AI에게 **현재 repository를 조사해 실제 build/test 명령을 확인한 후 작성하도록 해야 한다.**

모르는 프로젝트에 `mvn test`, `npm test`, `pytest` 같은 명령을 임의로 넣지 않는다.

---

## 5. Preview 배포도 GitHub Actions가 실행한다

고객사 정책에 따라 연결 방법을 선택한다.

### 방법 A: GitHub-hosted Runner에서 직접 접근

```text
GitHub Actions
-> HTTPS / SSH
-> target
```

공개 endpoint가 허용되는 경우에 사용할 수 있다.

### 방법 B: VPN Overlay

```text
GitHub Actions
-> Tailscale / WireGuard / Customer VPN
-> Private Server
```

서버 SSH를 인터넷에 공개하지 않아도 된다.

### 방법 C: Self-hosted Runner

```text
GitHub
-> 내부 self-hosted runner
-> 내부 서버
```

외부 SaaS runner에서 고객망으로 접근할 수 없는 경우 적합하다.

### 방법 D: Jenkins를 Executor로 사용

```text
GitHub Issue / webhook
-> Jenkins
-> Codex 또는 deploy script
-> target
```

이미 Jenkins를 운영 중인 고객사라면 기존 CI/CD 체계를 유지하면서 GitHub를 작업 상태 관리 계층으로 사용할 수 있다.

---

## 6. 테스트 결과가 아니라 실환경 결과까지 다시 확인한다

CI가 통과했다고 실제 배포가 성공했다고 볼 수 없다.

권장 검증 단계는 다음과 같다.

```text
syntax/static check
-> unit test
-> integration test
-> container build
-> Preview deploy
-> health check
-> API smoke test
-> 필요한 경우 DB/runtime state 확인
```

Workflow 마지막에는 AI가 읽기 쉬운 marker를 남길 수 있다.

```text
BUILD=PASS
TEST=PASS
DEPLOY_PREVIEW=PASS
HEALTH=PASS
RUNTIME_STATE=PASS
```

AI는 workflow의 초록색 체크만 보는 것이 아니라 필요하면 job log를 확인해 어떤 검증이 실제 수행됐는지 확인한다.

---

## 7. GitHub Issue를 상태 포인터로 사용한다

여러 대화와 여러 AI가 같은 프로젝트를 다룬다면 "마지막 배포 상태"를 GitHub Issue에 기록해둘 수 있다.

```text
[Runtime] Latest Preview Deployment

branch: feature/example
commit: abcdef1
workflow_run: 123456
status: PASS
build: PASS
test: PASS
deploy: PASS
health: PASS
runtime_state: PASS
verified_at: 2026-08-20T20:00:00+09:00
```

다음 AI는 이 Issue를 먼저 읽고 최신 commit/workflow와 비교한다.

주의할 점은 Issue가 **증거 자체가 아니라 포인터**라는 것이다. Issue가 오래된 경우 실제 최신 workflow를 우선한다.

---

## 8. 자동화 수준을 단계적으로 높인다

처음부터 완전자동화를 만들지 않는다.

| Level | 동작 |
|---|---|
| L0 | AI가 작업 방법만 제안 |
| L1 | AI가 repository code/docs 수정 |
| L2 | Push 후 CI 자동 실행 |
| L3 | Preview 배포와 runtime 검증까지 자동 실행 |
| L4 | 사람 승인 후 Production 배포 |
| L5 | 사전에 승인된 저위험 작업을 Production까지 자동 처리 |

일반 프로젝트에서는 다음 수준을 권장한다.

```text
Preview = L3
Production = L4
```

즉 개발과 반복 테스트에 드는 관리 포인트는 줄이되 실제 운영 변경에 대한 사람의 승인권은 유지한다.

---

## 9. Secret을 GitHub repository에 저장하지 않는다

저장소에는 secret의 **이름과 위치만** 기록한다.

```yaml
network:
  ssh_key_secret: CUSTOMER_PREVIEW_SSH_KEY
```

다음과 같은 실제 값은 commit하지 않는다.

```text
password
API token
private SSH key
실제 고객사 credential
private certificate key
```

가능한 저장 위치는 다음과 같다.

- GitHub Actions Secrets
- GitHub Environment Secrets
- Organization Secrets
- 고객사 Secret Manager
- target host의 권한 제한 runtime env

Workflow에서도 `echo $SECRET` 같은 방식으로 값을 출력하지 않는다.

---

## 10. 다른 고객사에 적용할 때 먼저 확인할 것

기술적으로 가능한 것과 고객 정책상 가능한 것은 다르다.

다음 항목을 먼저 확인한다.

```text
1. GitHub 사용 가능 여부
2. GitHub-hosted runner 허용 여부
3. 소스 외부 저장 제한 여부
4. 고객망 접근 방식
5. self-hosted runner 또는 Jenkins 사용 가능 여부
6. Preview 환경을 분리할 수 있는지
7. 운영 DB와 테스트 DB를 분리할 수 있는지
8. Secret 저장 위치
9. CI log에 기록하면 안 되는 정보
10. Production 배포 승인자와 rollback 방법
```

이 중 핵심 정보가 확인되지 않았다면 AI는 배포 자동화를 추측해서 완성하지 않고 `UNKNOWN` 또는 `BLOCKED`로 남기는 것이 맞다.

---

## 11. AI가 달라져도 같은 구조를 사용한다

### ChatGPT

GitHub connector/API를 사용할 수 있다면 대화 중 repository 수정, Issue 확인, Actions 결과 조회를 수행할 수 있다.

```text
사용자 요청
-> ChatGPT repository 조사
-> branch/code 수정
-> GitHub push
-> Actions 실행
-> ChatGPT가 workflow 결과 확인
```

### Codex

repository에 직접 접근할 수 있으므로 다음 방식으로 수행한다.

```text
repository clone
-> AGENTS.md
-> agent-ops.yaml
-> implementation
-> local test
-> branch/push
-> CI/Preview result 확인
```

### Claude / Gemini / 기타 AI

GitHub API, `gh` CLI, MCP, IDE integration 등 사용 가능한 연결 방식이 다를 뿐 동일한 계약을 적용한다.

중요한 것은 **AI별 프롬프트를 표준으로 삼지 않고 repository contract를 표준으로 삼는 것**이다.

---

## 12. AI에게 이 글을 전달할 때 사용할 Bootstrap Prompt

다른 고객사 또는 새로운 프로젝트에서 이 글을 ChatGPT나 Codex에게 전달한 뒤 다음 요청을 사용할 수 있다.

```text
이 글의 "GitHub Control Plane 기반 AI 반자동화" 구조를 현재 repository에 적용해줘.

목표:
- ChatGPT, Codex, 다른 AI가 같은 repository contract를 사용해야 한다.
- GitHub를 작업 상태와 실행 증거의 Control Plane으로 사용한다.
- AI는 branch에서 코드를 수정하고 push한다.
- GitHub Actions 또는 기존 CI가 build/test를 실행한다.
- 가능한 경우 별도 Preview 환경에 배포하고 runtime까지 검증한다.
- Production 배포 및 운영 데이터 변경은 사람 승인을 유지한다.

먼저 구현하지 말고 현재 repository와 실행 환경을 조사해 다음을 CONFIRMED / UNKNOWN으로 구분해줘.

1. default/production branch
2. 실제 build command
3. 실제 test command
4. 현재 CI/CD
5. 배포 target
6. runner에서 target까지의 네트워크 경로
7. Preview 환경 분리 가능 여부
8. DB/schema/volume 분리 방법
9. health/smoke 검증 방법
10. secret 보관 위치
11. rollback 방법
12. 운영 승인 필요 항목

조사 후 다음 파일을 현재 프로젝트에 맞게 작성하거나 기존 파일과 병합해줘.

- AGENTS.md
- ops/agent-ops.yaml
- docs/00-project-context.md
- docs/04-operation-and-deployment.md
- docs/06-ai-operations.md
- .github/workflows/ai-control-plane-verify.yml
- 프로젝트에 필요한 preview deploy workflow

규칙:
- 기존 AGENTS.md 또는 CI/CD를 무작정 덮어쓰지 말고 diff/병합한다.
- 확인하지 않은 host, 계정, credential, port, path를 추측하지 않는다.
- secret 값은 repository에 commit하지 않는다.
- main/production에서 직접 개발하지 않는다.
- Preview가 필요한 경우 운영과 port/container/network/DB/schema/volume 중 필요한 경계를 분리한다.
- 테스트하지 않은 결과는 PASS라고 쓰지 않는다.
- 완료 여부는 commit SHA, workflow run, test output, runtime response 같은 증거로 보고한다.
- 자동화 목표는 Preview L3, Production L4를 기본값으로 한다.

마지막에 다음을 보고해줘.

- 추가/수정 파일
- 현재 자동화 Level
- 자동으로 수행 가능한 범위
- 사람이 해야 하는 설정
- GitHub Secrets/Environment에 추가할 secret 이름(값은 제외)
- Preview 검증 방법
- Production 승인/rollback 방법
- 아직 UNKNOWN/BLOCKED인 항목
```

이 프롬프트의 목적은 AI가 바로 YAML을 만들어내게 하는 것이 아니라 **현재 프로젝트의 실제 상태를 먼저 조사한 뒤 같은 운영 구조를 프로젝트에 맞게 구성하게 하는 것**이다.

---

## 13. 새로운 AI에게 이어서 작업시키는 Prompt

이미 이 구조가 적용된 repository라면 더 짧게 요청할 수 있다.

```text
이 repository의 AI 작업 규칙을 먼저 읽고 현재 상태부터 복구해줘.

읽기 순서:
1. AGENTS.md
2. ops/agent-ops.yaml
3. docs/00-project-context.md
4. docs/04-operation-and-deployment.md
5. docs/06-ai-operations.md
6. 현재 Issue/PR
7. 최신 workflow run

문서의 상태와 실제 commit/workflow가 다르면 실제 실행 증거를 우선하고 차이를 보고해줘.
그 후 내가 요청한 작업을 기존 branch/Preview/검증/승인 정책에 맞게 수행해줘.
```

이 정도만 전달하면 AI 제품이 바뀌더라도 프로젝트 운영 방식을 처음부터 다시 설명하는 비용을 줄일 수 있다.

---

## 실제 적용에서 중요했던 점

개인 서비스와 원격 서버에 이 방식을 적용하면서 특히 효과가 있었던 부분은 다음과 같았다.

첫째, 운영 서비스를 사용하는 동안 별도 branch와 Preview 포트, 별도 DB를 만들어 AI가 계속 수정하고 배포해도 운영 사용을 중단하지 않을 수 있었다.

둘째, AI가 서버에 직접 접근하지 못해도 GitHub Actions를 다시 실행해 실제 DB row count, container 상태, health endpoint 등을 간접 확인할 수 있었다.

셋째, 첫 workflow가 실패했을 때 AI가 job log를 읽고 수정한 뒤 다시 push하는 반복 작업이 가능했다.

```text
구현
-> push
-> FAIL
-> log 분석
-> 수정
-> push
-> PASS
```

이 구조가 반복되면 사용자는 모든 명령을 직접 실행하는 작업자보다 **요구사항과 승인 지점을 관리하는 역할**에 가까워진다.

---

## 재발 방지와 운영 원칙

반자동화를 사용하면서 발견한 문제를 어디에 기록할지도 중요하다.

```text
한 번만 필요한 사실
-> Issue / project context

프로젝트에서 항상 지켜야 하는 규칙
-> AGENTS.md

배포나 실행에 필요한 구조화된 값
-> ops/agent-ops.yaml

반복되는 절차
-> script / Skill

자동으로 판정할 수 있는 규칙
-> test / GitHub Actions

현재 실제 상태
-> workflow evidence / status Issue
```

문서를 계속 늘리는 것이 목적이 아니다. 같은 실수가 발생하지 않도록 가장 적절한 위치에 규칙을 옮기는 것이 중요하다.

---

## 정리

이 구조의 핵심은 다음 한 문장으로 정리할 수 있다.

> AI가 서버를 직접 제어하도록 만드는 것이 아니라, GitHub에 의도를 기록하고 승인된 Runner가 실행하며 그 증거를 다시 GitHub로 돌려보내 AI가 다음 판단을 하게 만든다.

이를 통해 다음과 같은 흐름을 만들 수 있다.

```text
모바일 ChatGPT에서 요구 전달
-> GitHub 변경
-> 자동 Test
-> Preview Deploy
-> 실환경 검증
-> ChatGPT에서 결과 확인
-> 추가 개선
-> 최종 Production 승인
```

고객사가 달라져도 `GitHub + repository contract + runner + evidence`라는 중심 구조는 유지하고, 네트워크 연결과 배포 target만 고객 환경에 맞게 교체하면 된다.
