---
layout: post
title: "GitHub를 Control Plane으로 사용해 ChatGPT·Codex 개발과 배포를 반자동화하는 방법"
date: 2026-08-20
categories: [infrastructure]
tags: [GitHub, GitHub Actions, GitLab, Gitea, ChatGPT, Codex, AI Agent, CI/CD, DevOps, Automation]
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

# 한 장으로 보는 전체 구조

처음 보면 GitHub, AI, Runner, 서버의 역할이 섞여 보일 수 있다. 가장 먼저 아래 구조만 이해하면 된다.

```text
┌──────────────────────────────────────────────────────────────┐
│                           사용자                             │
│        목표 전달 · 우선순위 결정 · 운영 반영 승인           │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    ChatGPT / Codex / Other AI                │
│                                                              │
│  요구사항 정리 → 코드 조사 → 수정 → GitHub 반영             │
│  실패 로그 분석 → 수정 → 다시 GitHub 반영                   │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    GitHub = Control Plane                    │
│                                                              │
│  AGENTS.md        AI가 따라야 할 규칙                        │
│  agent-ops.yaml   배포·검증 계약                             │
│  Issue            작업 요청 / 현재 상태 포인터              │
│  Branch / PR      변경 격리 / 검토                           │
│  Actions          실제 작업 실행 Trigger                     │
│  Secrets          인증정보 보관                              │
└──────────────────────────────┬───────────────────────────────┘
                               │ workflow 실행
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                GitHub Actions / Jenkins / Runner             │
│                                                              │
│  Build → Test → Docker Build → Deploy → Runtime Verify       │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                         실행 환경                            │
│                                                              │
│      Preview Server / Linux / NAS / Cloud / Customer VM      │
│                       Application + DB                       │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               │ health / API / DB / log
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                     검증 결과를 GitHub로                     │
│                                                              │
│ BUILD=PASS · TEST=PASS · DEPLOY=PASS · HEALTH=PASS           │
│ commit SHA · workflow run ID · runtime state                 │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               └──────────────► AI가 다시 읽음
```

핵심은 다음과 같다.

> **AI가 서버에 직접 명령을 내리는 것이 아니라, AI가 GitHub의 상태를 변경하고 GitHub의 Runner가 실제 명령을 실행한 뒤 그 결과를 다시 GitHub에 남긴다.**

---

# 실제로 한 번의 요청이 처리되는 순서

사용자가 모바일 ChatGPT에서 "이 기능 수정해서 테스트 서버에 반영해줘"라고 요청했다고 가정한다.

```text
[1] 사용자
    "기능 A를 수정하고 Preview에서 확인해줘"
             │
             ▼
[2] ChatGPT / Codex
    repository 조사
    AGENTS.md 확인
    agent-ops.yaml 확인
             │
             ▼
[3] AI 작업 Branch
    feature/function-a
    코드 + 테스트 수정
             │
             ▼
[4] GitHub Push
    commit SHA 생성
             │
             ▼
[5] GitHub Actions
    Build
      ↓
    Unit Test
      ↓
    Integration Test
      ↓
    Container Build
             │
             ▼
[6] Preview 배포
    운영과 분리된 Container / Port / DB
             │
             ▼
[7] 실환경 검증
    /health
    API smoke test
    DB row count
    container status
             │
             ▼
[8] GitHub에 결과 기록
    commit=abcdef1
    run=123456
    TEST=PASS
    DEPLOY_PREVIEW=PASS
    HEALTH=PASS
             │
             ▼
[9] AI가 결과 재확인
       ┌───────────────┐
       │   성공했나?   │
       └───────┬───────┘
           YES │ NO
               │  └────► 로그 분석 → 코드 수정 → 다시 [4]
               ▼
[10] 사용자 승인
     Production 반영 여부 결정
```

사용자가 직접 해야 하는 것은 점점 다음 세 가지에 가까워진다.

```text
무엇을 만들 것인가
무엇이 완료인가
운영에 반영해도 되는가
```

---

# 실패했을 때 어떻게 자동으로 다시 수정하는가

반자동화의 핵심은 첫 배포 성공이 아니라 **실패 결과를 AI에게 다시 전달할 수 있다는 점**이다.

```text
             ┌──────────────┐
             │ AI 코드 수정 │◄───────────────────────┐
             └──────┬───────┘                        │
                    │ push                            │
                    ▼                                 │
             ┌──────────────┐                        │
             │ GitHub Action│                        │
             └──────┬───────┘                        │
                    │                                 │
                    ▼                                 │
             ┌──────────────┐                        │
             │ Test / Deploy│                        │
             └──────┬───────┘                        │
                    │                                 │
             ┌──────▼───────┐                        │
             │ PASS / FAIL? │                        │
             └───┬──────┬───┘                        │
              PASS      FAIL                         │
               │          │                           │
               │          ▼                           │
               │   ┌──────────────┐                  │
               │   │ GitHub Log   │                  │
               │   │ 원인 / stack │                  │
               │   │ failed step  │                  │
               │   └──────┬───────┘                  │
               │          │                           │
               │          ▼                           │
               │   ┌──────────────┐                  │
               │   │ AI가 로그 읽음│──────────────────┘
               │   └──────────────┘       원인 반영 후 코드 수정
               │
               ▼
        ┌──────────────┐
        │ Preview 완료 │
        └──────────────┘
```

즉 FAIL 경로는 다음처럼 반복된다.

```text
FAIL
-> GitHub Log
-> AI가 로그 읽음
-> AI 코드 수정
-> push
-> GitHub Actions 재실행
-> PASS가 될 때까지 반복
```

따라서 AI에게 서버 Shell을 직접 제공하지 않아도 된다.

```text
AI
-> GitHub를 수정
-> GitHub Runner가 실제 환경에서 실행
-> 실행 결과를 GitHub가 보관
-> AI가 GitHub 결과를 읽음
```

이 피드백 루프가 만들어지면 사용자가 매번 SSH 접속해서 명령을 복사하고 결과를 AI에게 다시 붙여넣는 작업을 크게 줄일 수 있다.

---

# Preview와 Production을 왜 분리하는가

AI가 수정한 코드를 바로 운영에 배포하면 반자동화가 아니라 위험한 자동화가 된다.

권장 구조는 다음과 같다.

```text
                        GitHub
                          │
               ┌──────────┴──────────┐
               │                     │
               ▼                     ▼
       feature/* branch           main branch
               │                     │
               ▼                     ▼
        Preview Workflow       Production Workflow
               │                     │
               ▼                     ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│       PREVIEW           │   │       PRODUCTION        │
│                         │   │                         │
│ preview container       │   │ production container    │
│ preview port            │   │ production port         │
│ preview DB/schema       │   │ production DB           │
│ preview volume          │   │ production volume       │
└────────────┬────────────┘   └────────────┬────────────┘
             │                              ▲
             │ 자동 검증                    │
             ▼                              │ 사람 승인
      Build/Test/Health                     │
             │                              │
             └──────── PASS ────────────────┘
```

즉 기본 정책을 다음처럼 둔다.

```text
Preview
= AI가 반복적으로 수정·배포·검증 가능

Production
= 사람이 최종 승인
```

이를 자동화 Level로 표현하면 일반적으로 다음 조합이 적절하다.

```text
Preview    = L3
Production = L4
```

---

# GitHub가 하는 일과 하지 않는 일

GitHub를 Control Plane이라고 하면 GitHub가 모든 작업을 직접 처리한다고 오해할 수 있다.

```text
GitHub가 하는 일
────────────────────────────────
작업 상태 저장
코드 변경 이력 저장
Workflow Trigger
Secret 전달
Runner 실행 요청
실행 Log 보관
검증 결과 보관

GitHub가 직접 하지 않는 일
────────────────────────────────
고객 업무 요구사항 판단
AI 추론
서버 애플리케이션 실행 자체
DB를 항상 직접 운영
운영 승인 판단
```

실제 명령을 실행하는 것은 Runner다.

```text
GitHub
  │
  ├─ GitHub-hosted Runner
  ├─ Self-hosted Runner
  └─ Jenkins
          │
          ▼
       Target Server
```

---

# GitHub가 아니어도 가능한가: GitLab과 Gitea

이 구조의 본질은 GitHub라는 제품 자체가 아니다. 필요한 것은 다음 기능을 제공하는 **Git 기반 Control Plane**이다.

```text
Repository
+ Issue 또는 Work Item
+ Branch / PR 또는 MR
+ CI/CD Pipeline
+ Runner
+ Secret 관리
+ 실행 Log / Evidence
```

따라서 GitLab과 Gitea도 같은 구조를 만들 수 있다.

| 개념 | GitHub | GitLab | Gitea |
|---|---|---|---|
| 코드 저장소 | Repository | Project/Repository | Repository |
| 작업 단위 | Issue | Issue / Work Item | Issue |
| 변경 검토 | Pull Request | Merge Request | Pull Request |
| CI/CD | GitHub Actions | GitLab CI/CD | Gitea Actions |
| 실행기 | GitHub Runner | GitLab Runner | Gitea Runner / act_runner |
| CI 설정 | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `.gitea/workflows/*.yml` |
| 상태 증거 | Actions log / Issue | Pipeline/Job log / Issue | Actions log / Issue |

추상화하면 다음처럼 볼 수 있다.

```text
                 AI
                  │
                  ▼
        ┌─────────────────────┐
        │ Git Control Plane   │
        │                     │
        │ GitHub / GitLab /   │
        │ Gitea               │
        └──────────┬──────────┘
                   │
                   ▼
             CI/CD Runner
                   │
                   ▼
            Preview / Target
```

다만 **AI가 Control Plane을 직접 읽고 수정하는 Adapter 수준은 제품마다 다르다.**

### GitHub

ChatGPT에서 repository를 직접 연결해 코드와 문서를 읽을 수 있는 공식 통합이 있고, API/Connector를 사용할 수 있는 환경에서는 Issue, branch, 파일, workflow 결과를 같은 대화에서 다루기 쉽다.

따라서 다음 형태의 **ChatGPT 중심 모바일 반자동화**에는 현재 가장 단순하다.

```text
모바일 ChatGPT
-> GitHub 수정
-> GitHub Actions
-> Preview
-> Actions Log
-> ChatGPT가 다시 확인
```

### GitLab

GitLab 자체의 Control Plane 기능은 충분하다. GitLab CI/CD는 `.gitlab-ci.yml`의 job을 GitLab Runner가 실행하고, build/test/deploy 결과를 다시 GitLab에 기록할 수 있다. Merge Request와 Issue도 동일한 역할을 수행할 수 있다.

```text
AI / Codex
-> GitLab branch
-> Merge Request
-> GitLab CI/CD
-> GitLab Runner
-> Preview
-> Pipeline / Job Log
```

특히 고객사가 이미 GitLab Self-Managed를 사용하거나 소스와 Runner를 고객망 내부에 유지해야 한다면 **GitHub보다 GitLab이 더 적절할 수도 있다.**

다만 ChatGPT 관점에서는 현재 GitHub repository 통합과 같은 수준으로 GitLab 코드·CI 전체를 직접 다루는 경로를 기본 전제로 두지 않는 편이 안전하다. ChatGPT에는 GitLab Issues 동기화 기능이 존재하지만, 이것만으로 repository 코드 수정과 pipeline 제어까지 동일하게 된다고 가정하지 않는다.

따라서 ChatGPT가 orchestration을 담당해야 한다면 다음 중 하나가 추가로 필요할 수 있다.

```text
GitLab API
glab CLI
MCP / Custom Plugin
별도 Agent Runner
Codex가 실행되는 고객사 내부 서버
```

### Gitea

Gitea도 Gitea Actions와 별도 Runner를 제공하므로 같은 구조를 만들 수 있다. Gitea Actions는 GitHub Actions와 유사하고 상당 부분 호환되므로 소규모 사내 구축이나 비용을 최소화한 self-hosted 환경에 유리하다.

```text
AI / Codex
-> Gitea
-> .gitea/workflows
-> Gitea Runner
-> Preview
-> Actions Log
```

하지만 GitHub Actions와 완전히 동일하다고 가정하지 않는다. Event, context, 일부 Action 호환성에는 차이가 있을 수 있으므로 실제 workflow는 Gitea에서 검증해야 한다.

또한 ChatGPT에서 Gitea를 직접 제어하는 공식 연결을 전제로 하지 않고 API/MCP/CLI 또는 별도 Agent Adapter가 필요하다고 보는 편이 안전하다.

### 어떤 것을 선택할 것인가

```text
ChatGPT에서 모바일로 직접 관리하는 편의성이 최우선
-> GitHub 우선

고객사가 이미 GitLab을 표준으로 사용
또는 소스/Runner를 고객망 내부에 유지해야 함
-> GitLab 우선

작은 사내망 / 개인 서버 / 완전 self-hosted / 비용 최소화
그리고 Adapter를 직접 구성할 수 있음
-> Gitea도 가능
```

즉 **Repository Contract는 공통으로 유지하고 플랫폼별 Adapter만 바꾼다.**

```text
공통
AGENTS.md
agent-ops.yaml
Preview / Production 정책
검증 기준
승인 기준

플랫폼별
GitHub Actions  <-> GitLab CI/CD <-> Gitea Actions
GitHub API      <-> GitLab API   <-> Gitea API
PR              <-> MR           <-> PR
```

이렇게 하면 향후 고객사에 따라 Git 플랫폼이 바뀌어도 자동화 원칙 자체를 다시 설계할 필요가 없다.

---

# 왜 GitHub가 Control Plane인가

이 글에서는 실제 적용 편의 때문에 GitHub를 기본 예제로 사용한다.

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

새로운 AI가 들어오면 다음 순서로 상태를 복구한다.

```text
새로운 AI
     │
     ▼
Repository
     │
     ├─ AGENTS.md
     ├─ agent-ops.yaml
     ├─ docs
     ├─ Current Issue / PR
     └─ Latest Workflow
     │
     ▼
현재 상태 복구
     │
     ▼
기존 방식으로 작업 계속
```

---

# 최소 Repository 구조

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

# 1. AGENTS.md: AI가 따라야 할 규칙

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

Codex는 `AGENTS.md` 기반 작업 규칙과 잘 맞는다. 다른 AI도 첫 요청에서 이 파일을 읽도록 지정하면 같은 기준을 적용할 수 있다.

---

# 2. agent-ops.yaml: AI가 읽을 수 있는 운영 계약

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
      separate_persistent_storage_when_needed: true
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

GitLab이나 Gitea에 적용하는 경우 `control_plane.provider`, workflow 경로, CI 명칭을 해당 플랫폼에 맞게 변경한다. 운영 원칙과 승인 기준은 유지한다.

---

# 3. Push를 테스트 Trigger로 사용한다

AI가 branch에 push하면 CI가 자동으로 검증한다.

```text
AI가 code 수정
    │
    ▼
git push
    │
    ▼
GitHub Actions
    │
    ├─ Build
    ├─ Unit Test
    ├─ Integration Test
    └─ Security / Static Check
    │
    ▼
PASS이면 Preview Deploy
```

Workflow 예시는 다음과 같다.

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

# 4. Preview 배포도 GitHub Actions가 실행한다

고객사 정책에 따라 연결 방법을 선택한다.

```text
                       GitHub Actions
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
    Public SSH/HTTPS     VPN Overlay          Self-hosted
                        Tailscale             Runner
                        WireGuard                 │
                        Customer VPN              │
          │                   │                    │
          └───────────────────┴────────────────────┘
                              │
                              ▼
                        Target Server
```

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

GitLab 또는 Gitea를 사용하는 경우에도 이 계층은 각각 GitLab Runner, Gitea Runner 또는 기존 Jenkins로 치환할 수 있다.

---

# 5. 테스트 결과가 아니라 실환경 결과까지 다시 확인한다

CI가 통과했다고 실제 배포가 성공했다고 볼 수 없다.

```text
Source
  │
  ▼
Syntax / Static
  │
  ▼
Unit Test
  │
  ▼
Integration Test
  │
  ▼
Container Build
  │
  ▼
Preview Deploy
  │
  ▼
Health Check
  │
  ▼
API Smoke Test
  │
  ▼
DB / Runtime State
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

# 6. GitHub Issue를 상태 포인터로 사용한다

여러 대화와 여러 AI가 같은 프로젝트를 다룬다면 "마지막 배포 상태"를 GitHub Issue에 기록해둘 수 있다.

```text
                 ┌──────────────────────────────┐
                 │ GitHub Issue                 │
                 │ Latest Preview Deployment    │
                 │                              │
                 │ branch: feature/example      │
                 │ commit: abcdef1              │
                 │ run: 123456                  │
                 │ status: PASS                 │
                 └──────────────┬───────────────┘
                                │
                 ┌──────────────▼───────────────┐
                 │ 다음 ChatGPT / Codex가 읽음 │
                 └──────────────┬───────────────┘
                                │
                                ▼
                   Latest commit/run과 비교
                                │
                    ┌───────────┴───────────┐
                    │                       │
                  동일                    다름
                    │                       │
                    ▼                       ▼
               작업 계속            실제 workflow 우선
```

상태 Issue 예시는 다음과 같다.

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

Issue는 **증거 자체가 아니라 포인터**다. Issue가 오래된 경우 실제 최신 workflow를 우선한다.

GitLab에서는 Issue/Work Item과 Pipeline/Job, Gitea에서는 Issue와 Actions Run을 같은 역할로 사용할 수 있다.

---

# 7. 자동화 수준을 단계적으로 높인다

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
                 자동화 수준

L0   방법 제안
 │
L1   코드 수정
 │
L2   CI 자동 실행
 │
L3   Preview 배포 + 실환경 검증    ◄── AI 기본 자동화 범위
 │
L4   Production 배포               ◄── 사람 승인
 │
L5   제한적 완전자동화
```

권장 기본값:

```text
Preview = L3
Production = L4
```

---

# 8. Secret을 GitHub repository에 저장하지 않는다

저장소에는 secret의 **이름과 위치만** 기록한다.

```yaml
network:
  ssh_key_secret: CUSTOMER_PREVIEW_SSH_KEY
```

실제 값은 commit하지 않는다.

```text
password
API token
private SSH key
실제 고객사 credential
private certificate key
```

가능한 저장 위치:

- GitHub Actions Secrets / Environment Secrets
- GitLab CI/CD Variables 또는 외부 Secret Manager
- Gitea Actions Secrets
- 고객사 Secret Manager
- target host의 권한 제한 runtime env

Workflow에서도 secret 값을 그대로 출력하지 않는다.

---

# 9. 다른 고객사에 적용할 때 먼저 확인할 것

기술적으로 가능한 것과 고객 정책상 가능한 것은 다르다.

```text
고객사 Repository
       │
       ▼
┌────────────────────────────┐
│ 어떤 Git 플랫폼인가?      │
│ GitHub / GitLab / Gitea    │
└──────────┬─────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ SaaS Runner 사용 가능한가? │
└──────────┬──────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ 서버까지 어떤 경로가 있는가?│
│ Public / VPN / Internal      │
└──────────┬───────────────────┘
           │
           ▼
┌────────────────────────────┐
│ Preview를 분리할 수 있는가?│
└──────────┬─────────────────┘
           │
           ▼
┌────────────────────────────┐
│ DB / Secret / Rollback 확인│
└──────────┬─────────────────┘
           │
           ▼
      자동화 범위 결정
```

실제 확인 항목은 다음과 같다.

```text
1. Git 플랫폼과 고객사 표준
2. Hosted runner 허용 여부
3. 소스 외부 저장 제한 여부
4. 고객망 접근 방식
5. self-hosted runner 또는 Jenkins 사용 가능 여부
6. Preview 환경을 분리할 수 있는지
7. 운영 DB와 테스트 DB를 분리할 수 있는지
8. Secret 저장 위치
9. CI log에 기록하면 안 되는 정보
10. Production 배포 승인자와 rollback 방법
11. ChatGPT/Codex가 해당 Git 플랫폼에 접근하는 방법
```

핵심 정보가 확인되지 않았다면 AI는 배포 자동화를 추측해서 완성하지 않고 `UNKNOWN` 또는 `BLOCKED`로 남긴다.

---

# 10. AI가 달라져도 같은 구조를 사용한다

AI 제품별 차이는 Git 플랫폼에 접근하는 방법에만 두고, 프로젝트 계약은 공통으로 유지한다.

```text
                  Repository Contract
               AGENTS.md + agent-ops.yaml
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
         ChatGPT        Codex      Claude/Gemini
             │            │            │
       Connector/API    git/CLI     API/MCP/CLI
             │            │            │
             └────────────┼────────────┘
                          │
                          ▼
                 GitHub / GitLab / Gitea
```

### ChatGPT

GitHub connector/API를 사용할 수 있다면 대화 중 repository 수정, Issue 확인, Actions 결과 조회를 수행하기 쉽다.

```text
사용자 요청
-> ChatGPT repository 조사
-> branch/code 수정
-> GitHub push
-> Actions 실행
-> ChatGPT가 workflow 결과 확인
```

GitLab이나 Gitea에서는 사용 가능한 App/Plugin/API/MCP 범위에 따라 동일한 작업을 직접 수행할 수 있는지 먼저 확인한다.

### Codex

repository에 직접 접근할 수 있으므로 Git provider에 대한 제약이 상대적으로 작다.

```text
repository clone
-> AGENTS.md
-> agent-ops.yaml
-> implementation
-> local test
-> branch/push
-> CI/Preview result 확인
```

GitHub에서는 `gh`, GitLab에서는 `glab`, Gitea에서는 git과 REST API 또는 적절한 CLI를 사용할 수 있다.

### Claude / Gemini / 기타 AI

Git API, CLI, MCP, IDE integration 등 사용 가능한 연결 방식이 다를 뿐 동일한 계약을 적용한다.

중요한 것은 **AI별 프롬프트나 Git 제품을 표준으로 삼지 않고 repository contract를 표준으로 삼는 것**이다.

---

# 11. AI에게 이 글을 전달할 때 사용할 Bootstrap Prompt

다른 고객사 또는 새로운 프로젝트에서 **이 글의 URL 또는 본문을 AI에게 전달한 뒤** 다음 요청을 사용한다.

```text
이 글의 "Git 기반 Control Plane AI 반자동화" 구조를 현재 repository에 적용해줘.

GitHub를 기본 예제로 사용하지만 고객사가 GitLab 또는 Gitea를 사용한다면 같은 역할을 해당 플랫폼의 Issue/MR/PR/CI/CD/Runner/API로 매핑해줘.

목표:
- ChatGPT, Codex, 다른 AI가 같은 repository contract를 사용해야 한다.
- Git 플랫폼을 작업 상태와 실행 증거의 Control Plane으로 사용한다.
- AI는 별도 branch에서 코드를 수정하고 push한다.
- GitHub Actions, GitLab CI/CD, Gitea Actions 또는 기존 CI가 build/test를 실행한다.
- 가능한 경우 별도 Preview 환경에 배포하고 runtime까지 검증한다.
- Production 배포 및 운영 데이터 변경은 사람 승인을 유지한다.

먼저 구현하지 말고 현재 repository와 실행 환경을 조사해 다음을 CONFIRMED / UNKNOWN으로 구분해줘.

1. Git provider와 repository 접근 방식
2. default/production branch
3. 실제 build command
4. 실제 test command
5. 현재 CI/CD
6. 배포 target
7. runner에서 target까지의 네트워크 경로
8. Preview 환경 분리 가능 여부
9. DB/schema/volume 분리 방법
10. health/smoke 검증 방법
11. secret 보관 위치
12. rollback 방법
13. 운영 승인 필요 항목
14. 현재 AI가 repository/issue/CI log를 읽고 쓸 수 있는 Adapter

조사 후 다음 파일을 현재 프로젝트에 맞게 작성하거나 기존 파일과 병합해줘.

- AGENTS.md
- ops/agent-ops.yaml
- docs/00-project-context.md
- docs/04-operation-and-deployment.md
- docs/06-ai-operations.md
- 현재 Git 플랫폼에 맞는 verify workflow/pipeline
- 프로젝트에 필요한 preview deploy workflow/pipeline

규칙:
- 기존 AGENTS.md 또는 CI/CD를 무작정 덮어쓰지 말고 diff/병합한다.
- 확인하지 않은 host, 계정, credential, port, path를 추측하지 않는다.
- secret 값은 repository에 commit하지 않는다.
- main/production에서 직접 개발하지 않는다.
- Preview가 필요한 경우 운영과 port/container/network/DB/schema/volume 중 필요한 경계를 분리한다.
- 테스트하지 않은 결과는 PASS라고 쓰지 않는다.
- 완료 여부는 commit SHA, workflow/pipeline run, test output, runtime response 같은 증거로 보고한다.
- 자동화 목표는 Preview L3, Production L4를 기본값으로 한다.
- 현재 AI가 Git provider에 직접 접근할 수 없다면 그것을 숨기지 말고 필요한 API/MCP/CLI/Runner Adapter를 제안한다.

마지막에 다음을 보고해줘.

- 추가/수정 파일
- 현재 자동화 Level
- 자동으로 수행 가능한 범위
- 사람이 해야 하는 설정
- Secret store에 추가할 secret 이름(값은 제외)
- Preview 검증 방법
- Production 승인/rollback 방법
- Git provider와 AI 사이의 Adapter 방식
- 아직 UNKNOWN/BLOCKED인 항목
```

이 프롬프트의 목적은 AI가 바로 YAML을 만들어내게 하는 것이 아니라 **현재 프로젝트의 실제 상태를 먼저 조사한 뒤 같은 운영 구조를 프로젝트에 맞게 구성하게 하는 것**이다.

---

# 12. 새로운 AI에게 이어서 작업시키는 Prompt

이미 이 구조가 적용된 repository라면 더 짧게 요청할 수 있다.

```text
이 repository의 AI 작업 규칙을 먼저 읽고 현재 상태부터 복구해줘.

읽기 순서:
1. AGENTS.md
2. ops/agent-ops.yaml
3. docs/00-project-context.md
4. docs/04-operation-and-deployment.md
5. docs/06-ai-operations.md
6. 현재 Issue/PR/MR
7. 최신 workflow/pipeline run

문서의 상태와 실제 commit/workflow/pipeline이 다르면 실제 실행 증거를 우선하고 차이를 보고해줘.
그 후 내가 요청한 작업을 기존 branch/Preview/검증/승인 정책에 맞게 수행해줘.
```

이 정도만 전달하면 AI 제품이나 Git provider가 바뀌더라도 프로젝트 운영 방식을 처음부터 다시 설명하는 비용을 줄일 수 있다.

---

# 실제 적용에서 중요했던 점

개인 서비스와 원격 서버에 이 방식을 적용하면서 특히 효과가 있었던 부분은 다음과 같았다.

첫째, 운영 서비스를 사용하는 동안 별도 branch와 Preview 포트, 별도 DB를 만들어 AI가 계속 수정하고 배포해도 운영 사용을 중단하지 않을 수 있었다.

둘째, AI가 서버에 직접 접근하지 못해도 CI/CD Runner를 다시 실행해 실제 DB row count, container 상태, health endpoint 등을 간접 확인할 수 있었다.

셋째, 첫 workflow가 실패했을 때 AI가 job log를 읽고 수정한 뒤 다시 push하는 반복 작업이 가능했다.

```text
구현
-> push
-> FAIL
-> log 분석
-> 코드 수정
-> push
-> PASS
```

이 구조가 반복되면 사용자는 모든 명령을 직접 실행하는 작업자보다 **요구사항과 승인 지점을 관리하는 역할**에 가까워진다.

---

# 재발 방지와 운영 원칙

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
-> test / CI pipeline

현재 실제 상태
-> workflow/pipeline evidence / status Issue
```

문서를 계속 늘리는 것이 목적이 아니다. 같은 실수가 발생하지 않도록 가장 적절한 위치에 규칙을 옮기는 것이 중요하다.

---

# 최종 요약 도식

마지막으로 전체 흐름을 한 줄로 정리하면 다음과 같다.

```text
┌────────┐    ┌─────────┐    ┌─────────────────┐    ┌────────┐    ┌─────────┐
│ 사용자 │ -> │   AI    │ -> │ Git Control Plane│ -> │ Runner │ -> │ Preview │
└────────┘    └─────────┘    │ GitHub/GitLab/  │    └────────┘    └────┬────┘
                              │ Gitea            │                      │
                              └───────┬──────────┘                      │
                                      ▲                                 │
                                      │       Test/Health/DB 결과       │
                                      └─────────────────────────────────┘
                                             │
                                             ▼
                                       AI가 재판단
                                             │
                                   ┌─────────┴─────────┐
                                   │                   │
                                 FAIL                PASS
                                   │                   │
                             AI 코드 수정          사람 승인
                                   │                   │
                                   └── push 재실행     ▼
                                                 Production
```

이 구조의 핵심은 다음 한 문장으로 정리할 수 있다.

> **AI가 서버를 직접 제어하도록 만드는 것이 아니라, Git 기반 Control Plane에 의도를 기록하고 승인된 Runner가 실행하며 그 증거를 다시 Control Plane으로 돌려보내 AI가 다음 판단을 하게 만든다.**

고객사가 달라져도 `Git provider + repository contract + runner + evidence`라는 중심 구조는 유지하고, GitHub/GitLab/Gitea와 네트워크 연결, 배포 target만 고객 환경에 맞게 교체하면 된다.
