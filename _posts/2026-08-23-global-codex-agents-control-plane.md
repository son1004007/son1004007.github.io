---
layout: post
title: "여러 프로젝트와 서버에서 Codex 운영 기준을 자동으로 이어받게 만든 방법"
date: 2026-08-23
categories: [infrastructure]
tags: [Codex, AGENTS.md, AI Agent, GitHub, Control Plane, Synology, Linux]
---

AI를 여러 프로젝트와 여러 실행 환경에서 사용하다 보면 코드 작성보다 더 귀찮은 문제가 생긴다.

새 채팅이나 새 Codex 세션을 시작할 때마다 다음 내용을 다시 설명해야 하는 문제다.

- 어떤 저장소가 기준 저장소인지
- 무엇을 먼저 읽어야 하는지
- 모르는 기술 동작은 공식 문서부터 확인해야 한다는 규칙
- 서버 작업은 어디를 통해 실행해야 하는지
- 테스트와 실행 증거 없이 완료라고 하면 안 된다는 규칙
- 프로젝트 상태는 중앙 문서가 아니라 각 프로젝트 저장소에서 확인해야 한다는 원칙

프로젝트가 몇 개 없을 때는 반복 설명으로 버틸 수 있지만 저장소와 실행 환경이 늘어나면 같은 설명을 여러 곳에 복사하게 된다. 그러면 결국 문서끼리 상태가 달라진다.

이번에는 이 문제를 **중앙 Control + 프로젝트별 AGENTS.md + 실행 환경의 전역 AGENTS.md** 구조로 정리했다.

## 목표

사용자가 매번 긴 운영 프롬프트를 쓰지 않아도 다음 정도의 요청으로 작업을 이어갈 수 있게 하는 것이 목표였다.

```text
이 프로젝트 계속 진행해줘.
기존 기준으로 진행해줘.
서버에서 검증까지 해줘.
```

Agent는 이전 대화를 요구하기 전에 GitHub 저장소와 실행 증거를 이용해 현재 상태를 복원해야 한다.

## 중앙 저장소에는 상태가 아니라 '찾는 방법'만 둔다

중앙 Control 저장소에 모든 프로젝트의 현재 상태를 복사하면 관리 포인트만 늘어난다.

그래서 중앙에는 다음만 둔다.

```text
CONTROL.md
  - 공통 작업 시작 절차
  - Source of Truth 규칙
  - Official-Source-First
  - 검증/완료 판정 원칙

repository registry
  - 저장소 역할
  - 먼저 읽을 파일
  - 필요한 경우 실행 환경 매핑
```

실제 프로젝트 상태는 계속 각 저장소가 소유한다.

```text
중앙 Control
  -> 어떤 저장소와 문서를 읽을지 결정

각 프로젝트 저장소
  -> 코드
  -> AGENTS.md
  -> AI_CONTEXT / CURRENT_STATE / WORKS / TASKS
  -> 테스트
  -> 배포/복구 절차
```

이렇게 하면 중앙 저장소가 stale 상태를 복제하지 않는다.

## 프로젝트에서 중앙 Control을 다시 발견하게 한다

중앙 저장소만 잘 만들어도 다른 프로젝트에서 직접 작업을 시작하면 중앙 규칙을 모를 수 있다.

그래서 반복적으로 사용하는 프로젝트의 root `AGENTS.md`에는 짧은 전역 포인터를 넣었다.

개념적으로는 다음과 같다.

```text
Global control repository의 CONTROL.md를 먼저 확인한다.
그 후 현재 repository의 AGENTS.md와 현재 상태 문서로 돌아온다.
현재 repository가 자신의 코드와 상태에 대한 Source of Truth다.
```

새 프로젝트에 사용하는 bootstrap template에도 같은 포인터를 넣었다.

따라서 앞으로 새로 만든 프로젝트는 처음부터 같은 운영 계약을 갖는다.

오래된 모든 저장소를 한 번에 수정하지는 않았다. 오래된 프로젝트를 다시 활성화할 때 전역 포인터와 registry를 추가하는 **on-first-touch** 방식으로 관리한다.

## Codex 실행 환경에도 전역 AGENTS.md를 둔다

Repository-level `AGENTS.md`만으로도 대부분의 작업은 해결되지만 서버에서 어떤 repository를 열더라도 동일한 기본 규칙을 먼저 읽게 하고 싶었다.

OpenAI가 설명하는 Codex instruction 구성에서는 `$CODEX_HOME`의 `AGENTS.md` 또는 `AGENTS.override.md`를 사용자 지침으로 읽고, 이어서 Git/project root에서 현재 작업 디렉터리까지의 프로젝트 지침을 추가한다.

관련 공식 설명:

- [Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)
- [Introducing Codex](https://openai.com/index/introducing-codex/)

그래서 Linux 개발 서버와 Synology NAS의 Codex 환경 모두에 `$CODEX_HOME/AGENTS.md` managed block을 설치했다.

전역 파일은 프로젝트 상태를 포함하지 않는다. 다음 원칙만 갖는다.

```text
1. GitHub 접근이 가능하면 중앙 CONTROL.md를 먼저 확인
2. 다시 현재 repository의 local AGENTS.md로 돌아오기
3. 프로젝트 상태와 코드는 현재 repository를 Source of Truth로 사용
4. 불확실한 기술 동작은 Official-Source-First 적용
5. repository와 runtime evidence에서 복원 가능한 내용을 사용자에게 반복 질문하지 않기
6. sandbox/auth/security 정책을 문제 해결 목적으로 함부로 완화하지 않기
7. 중앙 Control에 접근할 수 없으면 local 규칙으로 fail-safe하게 동작
```

기존 사용자가 만들어 둔 `$CODEX_HOME/AGENTS.md`가 있을 수도 있기 때문에 파일 전체를 덮어쓰지 않았다.

```text
기존 사용자 지침

<!-- managed block start -->
Global control bootstrap
<!-- managed block end -->
```

처럼 관리 구간만 추가하거나 교체하도록 구현했다.

## 두 실행 환경의 격리 방식은 그대로 유지했다

전역 지침을 추가했다고 해서 실행 격리까지 하나로 통일하지 않았다.

Ubuntu 개발 환경은 이미 검증된 구조를 유지한다.

```text
clean canonical checkout
 -> detached disposable git worktree
 -> Codex native Bubblewrap read-only sandbox
 -> 변경 여부 검사
 -> worktree remove/prune
```

NAS에서는 OS 특성상 Codex native Linux sandbox를 억지로 맞추지 않고 기존 외부 컨테이너 경계를 유지한다.

```text
persistent application runtime
 -> disposable snapshot
 -> read-only Docker/Container Manager boundary
 -> temporary Codex HOME
 -> Codex
 -> snapshot 변경 여부 검사
```

NAS runner는 호스트의 Codex 인증/사용자 디렉터리를 임시 컨테이너 HOME에 복사하므로, 호스트 `$CODEX_HOME/AGENTS.md`의 managed global instruction도 컨테이너 실행에 전달된다.

핵심은 **공통 정책은 공유하되 각 환경에서 검증된 격리 방법은 유지하는 것**이다.

## 파일이 존재하는지만 확인하지 않았다

`AGENTS.md` 파일을 생성했다고 해서 Codex가 실제로 읽었다고 단정하면 안 된다.

그래서 managed block에 검증용 marker를 하나 넣었다.

```text
DEVICE_GLOBAL_CONTROL_MARKER=global-control-v1
```

그 다음 실제 read-only Codex task에는 다음 조건을 줬다.

```text
AGENTS.md와 CODEX_HOME 파일을 직접 열거나 검색하지 않는다.
이미 초기 context에 로드된 instruction만 이용한다.
DEVICE_GLOBAL_CONTROL_MARKER 값을 보고한다.
파일은 수정하지 않는다.
```

결과적으로 두 환경 모두 Codex가 다음 값을 반환했다.

```text
DEVICE_GLOBAL_CONTROL_MARKER=global-control-v1
```

Ubuntu 개발 환경에서는 추가로 다음 조건도 확인됐다.

```text
read_only_violation=no
worktree_cleanup=yes
agent_exit_code=0
```

NAS 외부 컨테이너 환경에서도 다음 조건을 확인했다.

```text
read_only_violation=no
agent_exit_code=0
```

즉 **파일 배치 확인이 아니라 실제 Codex context ingestion까지 E2E로 검증**했다.

## 검증 중 stale checkout도 발견했다

첫 Ubuntu E2E는 Codex까지 도달하지 못했다.

원인은 sandbox가 아니라 canonical checkout이 GitHub main보다 한 commit 뒤에 있었기 때문이다.

preflight는 이 상태에서 작업을 계속하지 않고 실패시켰다.

```text
local commit != GitHub main commit
 -> agent 실행 차단
```

canonical checkout을 fast-forward로 동기화하고 clean 상태를 다시 확인한 뒤 E2E를 재실행했고 통과했다.

이 과정은 오히려 현재 구조가 의도대로 동작한다는 증거였다. 전역 정책을 읽게 만드는 것보다 중요한 것은 **잘못된 source 상태에서 Agent가 실행되지 않는 것**이기 때문이다.

## 최종 구조

현재 구조를 단순화하면 다음과 같다.

```text
Central AI workflow/control
        |
        +-> repository registry
        +-> Official-Source-First
        +-> bootstrap template
        |
        v
Project root AGENTS.md
        |
        +-> project state / code / tests
        |
        v
Runtime $CODEX_HOME/AGENTS.md
        |
        v
Codex
        |
        +-> Ubuntu: disposable worktree + bwrap
        |
        +-> NAS: disposable snapshot + external container isolation
```

실제 적용 시 Codex는 전역 지침과 더 구체적인 repository-local 지침을 함께 받는다.

중앙 Control은 프로젝트 상태를 대신하지 않고 **어디서 현재 사실을 찾아야 하는지를 알려주는 index**로만 유지한다.

## 운영 원칙

이 구조를 유지하기 위한 규칙은 몇 가지로 제한했다.

1. 중앙 Control에 프로젝트의 mutable 상태를 복사하지 않는다.
2. 프로젝트별 코드는 해당 repository가 Source of Truth다.
3. 새 프로젝트 template에는 global control pointer를 자동 포함한다.
4. 기존 프로젝트는 다시 작업할 때 on-first-touch 방식으로 연결한다.
5. 실행 환경의 global AGENTS는 managed block으로 갱신한다.
6. 전역 지침 적용 여부도 실제 Agent E2E로 검증한다.
7. Global Control 접근 실패가 보안 정책 완화의 이유가 되어서는 안 된다.
8. 모르는 기술 동작은 [Official-Source-First]({% post_url 2026-08-23-official-source-first-ai-troubleshooting %}) 원칙을 따른다.

## 결과

이전에는 새 채팅이나 다른 Agent에서 프로젝트를 이어가려면 운영 규칙을 다시 설명해야 했다.

지금은 다음 흐름을 목표로 한다.

```text
현재 repository 확인
 -> local AGENTS.md에서 global control 발견
 -> 중앙 Control에서 읽을 위치 결정
 -> 현재 repository의 최신 상태 복원
 -> 필요할 때만 remote runtime 확인
 -> 구현
 -> 검증
 -> 상태 문서 갱신
```

완전히 모든 AI 환경에서 자동 상속되는 단일 GitHub 기능이 있는 것은 아니다. 따라서 repository pointer와 runtime global instruction을 함께 사용했다.

결과적으로 **GitHub에서 시작해도, Ubuntu 개발 서버의 Codex에서 시작해도, NAS의 격리된 Codex에서 시작해도 동일한 기본 운영 철학을 발견할 수 있는 구조**가 됐다.
