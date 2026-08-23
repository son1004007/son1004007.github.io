---
layout: post
title: "ChatGPT Custom Instructions로 GitHub 작업 기준을 자동으로 불러오게 만드는 방법"
date: 2026-08-24
categories: [infrastructure]
tags: [ChatGPT, Custom Instructions, GitHub, GitHub Copilot, AI Agent, Control Plane, AGENTS.md]
---

여러 ChatGPT 채팅에서 같은 GitHub 프로젝트를 반복해서 다루다 보면 매번 같은 설명을 다시 해야 하는 문제가 생긴다.

예를 들어 다음 내용을 새 채팅마다 다시 전달하게 된다.

- 어떤 저장소가 전체 작업 기준인지
- 어떤 프로젝트 저장소를 실제 Source of Truth로 봐야 하는지
- `AGENTS.md`, `AI_CONTEXT.md`, `CURRENT_STATE.md`, `WORKS.md`, `TASKS.md` 중 무엇을 먼저 읽어야 하는지
- 원격 서버 작업은 어떤 제어 저장소와 안전 규칙을 따라야 하는지
- 모르는 기술 동작은 공식 문서를 먼저 확인해야 한다는 원칙
- 테스트나 실행 증거 없이 완료라고 판단하면 안 된다는 규칙

이 문제를 해결하기 위해 GitHub 쪽에는 중앙 Control 저장소와 프로젝트별 `AGENTS.md`를 두고, 서버의 Codex에는 `$CODEX_HOME/AGENTS.md`를 두었다. 하지만 일반 ChatGPT의 새 채팅은 Codex처럼 `$CODEX_HOME/AGENTS.md`를 자동 탐색하지 않는다.

그래서 ChatGPT에는 **Custom Instructions를 아주 얇은 bootstrap layer로 사용**했다.

이 글은 [여러 프로젝트와 서버에서 Codex 운영 기준을 자동으로 이어받게 만든 방법]({% post_url 2026-08-23-global-codex-agents-control-plane %})에서 구성한 GitHub/서버 Control 구조를 일반 ChatGPT 새 채팅까지 확장한 기록이다.

## 문제점

GitHub에 중앙 정책 저장소가 있어도 일반 ChatGPT 새 채팅은 그 저장소의 존재를 자동으로 알 수 없다.

따라서 이런 요청을 했을 때:

```text
포트폴리오 계속 진행해줘.
CISA 서비스 상태 확인해줘.
Office에서 테스트해줘.
```

ChatGPT가 이전 대화를 기준으로 추측하거나, 사용자에게 이미 GitHub에 기록된 상태를 다시 설명해 달라고 요구할 수 있다.

중앙 Control의 목적은 오히려 반대다.

```text
사용자가 과거 상태를 다시 설명
        X

ChatGPT가 GitHub에서 현재 상태 복원
        O
```

즉 ChatGPT가 **어디서 시작해야 하는지 한 번만 알려주는 계정 수준 bootstrap**이 필요했다.

## 원인

Codex와 일반 ChatGPT는 지침을 발견하는 위치가 다르다.

Codex CLI는 사용자 환경의 global `AGENTS.md`와 프로젝트의 `AGENTS.md`를 이용할 수 있다. 반면 일반 ChatGPT 새 채팅은 내 GitHub의 특정 private repository를 임의로 찾아 `CONTROL.md`부터 읽는다는 보장이 없다.

그래서 역할을 다음처럼 분리했다.

```text
ChatGPT Custom Instructions
  -> 중앙 Control의 위치만 알려줌

GitHub 중앙 Control
  -> 어떤 저장소를 어떤 순서로 읽을지 결정

각 프로젝트 repository
  -> 코드와 실제 현재 상태의 Source of Truth

Office / NAS Codex
  -> $CODEX_HOME/AGENTS.md로 동일한 중앙 Control 발견
```

핵심은 **Custom Instructions에 프로젝트 상태를 복사하지 않는 것**이다.

프로젝트 상태까지 Custom Instructions에 넣으면 시간이 지나면서 GitHub와 내용이 달라지고 다시 관리 포인트가 생긴다.

## GitHub 공식 문서에서도 확인되는 유사한 구조

이 구조를 만든 뒤 GitHub 공식 문서를 확인해 보니, GitHub Copilot도 instruction을 한 파일에 모두 넣기보다 **사용자 수준, repository 수준, path 수준, agent 수준으로 나누는 구조**를 공식 지원하고 있었다.

GitHub 공식 문서:

- [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)
- [Adding repository custom instructions for GitHub Copilot in your IDE](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions-in-your-ide/add-repository-instructions-in-your-ide)
- [Adding custom instructions for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Support for different types of custom instructions](https://docs.github.com/en/copilot/reference/custom-instructions-support)
- [Customize Copilot for your project](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-copilot-overview)

### Repository-wide instructions

GitHub은 `.github/copilot-instructions.md`를 repository 전체에 적용되는 지속적인 지침으로 정의한다.

이 파일에는 프로젝트 구조, 코딩 규칙, build/test 방법처럼 해당 repository를 이해하고 수정할 때 반복적으로 필요한 정보를 둘 수 있다.

개념적으로 내가 사용하는 root `AGENTS.md`와 비슷한 역할이다.

```text
repository
  -> .github/copilot-instructions.md
  -> AGENTS.md
  -> code / tests / local state docs
```

다만 둘은 같은 파일 형식이 아니다. 사용하는 AI와 실행 환경이 지원하는 instruction 종류를 확인해야 한다.

### Path-specific instructions

GitHub은 `.github/instructions/**/*.instructions.md` 파일과 `applyTo` glob을 이용해 특정 경로에만 지침을 적용할 수 있도록 한다.

예를 들어 Java와 frontend 코드의 규칙이 다르다면 전역 문서를 거대하게 만드는 대신 다음처럼 범위를 분리할 수 있다.

```text
.github/instructions/
  backend.instructions.md
  frontend.instructions.md
```

이는 중앙 Control에 모든 프로젝트 세부 규칙을 넣지 않고 **가장 가까운 실제 작업 범위에 세부 정책을 둔다**는 현재 설계와 같은 방향이다.

### Agent instructions와 AGENTS.md

GitHub 공식 문서는 Copilot agent가 `AGENTS.md`를 agent instruction으로 사용할 수 있다고 설명한다.

하위 디렉터리에 여러 `AGENTS.md`가 있다면 작업 위치와 가까운 파일이 더 구체적인 지침 역할을 한다.

```text
root AGENTS.md
  -> 전체 repository 규칙

backend/AGENTS.md
  -> backend에 더 구체적인 규칙

backend/payment/AGENTS.md
  -> payment 작업에 가장 구체적인 규칙
```

이 점은 현재 사용 중인 **전역 Control은 공통 원칙만 소유하고, 실제 프로젝트 repository의 local `AGENTS.md`가 더 구체적인 제한을 소유하는 방식**과 잘 맞는다.

### Copilot CLI에는 사용자 수준 전역 instructions도 있다

GitHub Copilot CLI 공식 문서는 다음 위치를 사용자 수준 instruction으로 정의한다.

```text
$HOME/.copilot/copilot-instructions.md
$HOME/.copilot/instructions/**/*.instructions.md
```

이 지침은 여러 repository에 걸쳐 적용된다.

역할만 비교하면 현재 구조의 다음 두 요소와 유사하다.

```text
ChatGPT
 -> Custom Instructions

Codex
 -> $CODEX_HOME/AGENTS.md
```

즉 **사용자 수준에는 얇은 공통 bootstrap을 두고, repository 수준에는 구체적인 프로젝트 지침을 두는 패턴** 자체가 GitHub Copilot의 공식 customization 모델에서도 확인된다.

### 다른 파일을 참조하는 기능도 공식 지원한다

Copilot CLI 문서에서는 `.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md` 등에서 `@`와 상대 경로를 사용해 다른 파일을 포함할 수 있다고 설명한다.

```text
@docs/build-and-test.md
@docs/security-policy.md
```

이 기능은 한 문서에 세부 내용을 복사하지 않고 원본 문서를 참조하는 데 유용하다.

내 중앙 Control 설계도 같은 원칙을 사용하지만 구현 방식은 다르다.

```text
Custom Instructions
 -> CONTROL.md를 확인하라고 지시
 -> registry를 읽음
 -> 실제 Source of Truth로 이동
```

일반 ChatGPT에서는 GitHub Copilot CLI의 `@relative/path` 포함 동작을 그대로 가정하지 않는다. **GitHub 접근이 가능한 ChatGPT에게 자연어 instruction으로 해당 파일을 실제 확인하도록 요구하는 bootstrap**이다.

### 모든 제품 표면이 같은 instruction을 지원하는 것은 아니다

GitHub은 별도 support matrix를 제공한다. GitHub.com Copilot Chat, Copilot cloud agent, code review, VS Code, JetBrains, Copilot CLI 등에서 지원하는 instruction 종류가 서로 다르다.

```text
AGENTS.md가 GitHub의 한 기능에서 지원됨
 !=
모든 AI/모든 실행환경에서 자동 적용됨
```

따라서 현재 구조에서는 특정 제품의 자동 탐색 기능 하나에만 의존하지 않고 다음을 함께 사용한다.

```text
ChatGPT Custom Instructions
 + repository-local AGENTS.md
 + central CONTROL.md
 + Codex runtime global AGENTS.md
 + local security fallback
```

이 방식은 중복처럼 보일 수 있지만 실제로는 **서로 다른 AI 실행 표면 사이의 bootstrap gap을 메우는 최소 포인터**다.

## 해결

ChatGPT의 Custom Instructions에는 중앙 GitHub Control을 찾기 위한 bootstrap만 넣었다.

현재 OpenAI 도움말 기준으로 Custom Instructions는 Web, Desktop, iOS, Android에서 사용할 수 있고, 활성화하면 채팅 전반에 적용된다.

OpenAI 공식 문서:

- [ChatGPT Custom Instructions](https://help.openai.com/en/articles/8096356-chatgpt-custom-instructions)
- [Projects in ChatGPT](https://help.openai.com/en/articles/10169521-projects-in-chatgpt)

Android에서는 현재 다음 순서로 설정할 수 있다.

```text
ChatGPT Settings
 -> Customize ChatGPT
 -> Enable customization ON
 -> Custom Instructions
```

Web/Desktop에서는 `Settings -> Personalization`에서 Custom Instructions를 설정한다.

## 실행 방법

Custom Instructions에는 아래와 같은 bootstrap을 넣는다.

```text
[GitHub AI Work Bootstrap]

내 GitHub 저장소, 개발, 서버, 배포, Agent 작업 또는 이전 작업의 연속성을 다루는 요청에서는 GitHub 접근이 가능하면 먼저 다음 전역 Control을 확인한다.

son1004007/ai-agent-workflow-playbook/CONTROL.md

그 다음 CONTROL.md의 repository registry를 따라 실제 대상 저장소를 찾고, 해당 저장소의 AGENTS.md와 AI_CONTEXT.md, CURRENT_STATE.md, WORKS.md, TASKS.md 등 존재하는 최신 상태 문서를 확인한다.

프로젝트 코드와 현재 상태는 각 대상 repository를 Source of Truth로 사용한다. GitHub commit, PR, Issue, Actions 또는 실제 runtime evidence로 확인 가능한 내용은 사용자에게 다시 설명해 달라고 요구하지 말고 직접 확인한다.

원격 실행이 필요하면 CONTROL.md의 mapping을 따라 device-control을 확인하고 기존 보안, sandbox, 배포 정책을 유지한다.

기술 동작이 불확실하거나 버전, 인증, 보안, sandbox, OS/runtime에 의존하면 추측하지 말고 Official-Source-First를 적용한다.

테스트나 실제 실행 증거 없이 완료를 주장하지 않는다. 전역 Control에 접근할 수 없으면 local AGENTS.md와 안전 규칙을 사용하며, 이를 이유로 보안 제한을 완화하지 않는다.
```

여기에는 서버 IP, SSH 계정, 토큰, 프로젝트의 현재 진행률 같은 mutable/private 정보를 넣지 않는다.

그 정보는 각각의 Source of Truth에서 읽어야 한다.

```text
Custom Instructions
        |
        v
ai-agent-workflow-playbook/CONTROL.md
        |
        +-> config/repositories.yml
        |
        +-> engineering-career-portfolio
        |      -> AGENTS.md
        |      -> AI_CONTEXT / WORKS / TASKS
        |
        +-> cisa-playbook
        |      -> AGENTS.md
        |      -> service/runbook docs
        |
        +-> device-control
               -> remote runtime policy/evidence
```

## Project Instructions와의 차이

ChatGPT Project를 사용하는 경우 프로젝트별 지침도 따로 설정할 수 있다.

OpenAI 공식 도움말 기준으로 Project Instructions는 **그 프로젝트 안에서만 적용되고 전역 Custom Instructions보다 우선한다.**

```text
일반 채팅
 -> global Custom Instructions

특정 ChatGPT Project 안의 채팅
 -> Project Instructions
 -> 필요 시 global bootstrap과 같은 중앙 Control 사용
```

프로젝트별로 완전히 다른 규칙이 필요하지 않다면 Project Instructions에도 세부 프로젝트 상태를 복사하지 않고 다음 정도의 포인터만 두는 편이 관리하기 쉽다.

```text
GitHub 개발/운영 작업에서는
son1004007/ai-agent-workflow-playbook/CONTROL.md를 먼저 확인하고,
그 registry와 대상 repo의 local AGENTS.md를 따른다.
```

## 검증 방법

설정을 넣은 뒤에는 새 ChatGPT 채팅에서 실제로 복원 흐름이 동작하는지 확인한다.

```text
내 포트폴리오 현재 진행 상태 확인해줘.
```

기대한 동작은 다음과 같다.

```text
Custom Instructions
 -> CONTROL.md
 -> repositories.yml
 -> engineering-career-portfolio
 -> AGENTS.md
 -> AI_CONTEXT / WORKS / TASKS
 -> latest commit / PR / Actions
 -> 필요한 경우 device-control / Office runtime
```

다음과 같은 응답 패턴이면 bootstrap이 제대로 활용되지 않았을 가능성이 있다.

```text
이전에 어디까지 했는지 다시 설명해주세요.
서버가 어디인지 알려주세요.
프로젝트 repo 이름을 다시 알려주세요.
```

물론 GitHub 접근 권한이 없는 ChatGPT 환경에서는 private Control repository를 실제로 읽을 수 없다. 이 경우에는 접근 불가 사실을 표시하고 사용자가 제공한 현재 자료나 공개 repository 범위에서만 판단해야 한다.

## 재발 방지 / 개선 방향

이 구조에서 관리해야 할 원본은 최대한 줄인다.

```text
1. 전역 작업 정책
   -> ai-agent-workflow-playbook/CONTROL.md

2. repository 관계
   -> config/repositories.yml

3. 프로젝트 현재 상태
   -> 각 프로젝트 repository

4. Codex 실행환경 bootstrap
   -> $CODEX_HOME/AGENTS.md managed block

5. 일반 ChatGPT bootstrap
   -> Custom Instructions
```

Custom Instructions에는 **중앙 Control을 찾는 규칙만 유지**한다.

새 repository를 만들거나 기존 repository를 다시 활성화할 때는 중앙 registry와 해당 repository의 `AGENTS.md`를 갱신한다. Custom Instructions는 보통 다시 수정할 필요가 없다.

GitHub Copilot까지 같은 운영 철학을 적용한다면 공식 지원 위치를 그대로 활용할 수 있다.

```text
Copilot user-level
 -> $HOME/.copilot/copilot-instructions.md

Repository-wide
 -> .github/copilot-instructions.md

Path-specific
 -> .github/instructions/**/*.instructions.md

Agent-specific
 -> AGENTS.md
```

다만 모든 파일에 동일한 내용을 복사하는 것은 피하고, 각 계층에는 자기 범위에 필요한 최소 지침과 원본 문서 포인터만 두는 편이 낫다.

기술 동작을 잘 모를 때의 공통 판단 방식은 [Official-Source-First 원칙]({% post_url 2026-08-23-official-source-first-ai-troubleshooting %})을 그대로 사용한다.

## 주의점

Custom Instructions에 다음 정보를 넣지 않는 편이 좋다.

- 비밀번호, API key, SSH private key
- 서버 IP와 외부 공개가 불필요한 접속 정보
- 고객사명과 내부 시스템 식별자
- 현재 연봉, 개인정보 등 개발 workflow와 무관한 민감정보
- 프로젝트의 수시로 변하는 현재 commit/status

Custom Instructions는 계정 전반의 많은 대화에 영향을 줄 수 있는 위치이므로 **세부 상태 저장소가 아니라 bootstrap contract**로 취급하는 것이 적절하다.

또한 Custom Instructions가 있다고 해서 GitHub 접근 권한 자체가 생기는 것은 아니다. private repository를 확인해야 하는 작업에서는 해당 ChatGPT 환경이 GitHub에 접근할 수 있어야 한다.

GitHub Copilot의 공식 instruction 파일이 존재한다는 사실도 일반 ChatGPT가 그 파일을 자동으로 읽는다는 의미는 아니다. 제품과 실행 표면별 공식 지원 범위를 분리해서 판단해야 한다.

## 포트폴리오 관점의 의미

이 작업은 단순한 ChatGPT 설정 편의 기능이 아니다.

여러 AI와 여러 실행 환경이 있는 개발 workflow에서 중요한 것은 프롬프트를 길게 작성하는 것이 아니라 다음을 명확히 만드는 것이다.

- 정책의 단일 Source of Truth
- 프로젝트별 상태의 소유권
- Agent가 작업 시작 시 context를 복원하는 bootstrap 절차
- 실행 환경별 안전 경계
- 실제 test/runtime evidence에 기반한 완료 판정
- 사람이 반복해서 설명해야 하는 관리 포인트의 축소

최종 구조는 다음과 같다.

```text
ChatGPT Custom Instructions
          |
          v
GitHub Global Control
          |
          +-------> Project AGENTS.md
          |              |
          |              v
          |        Project Source of Truth
          |
          +-------> device-control
                         |
             +-----------+-----------+
             |                       |
             v                       v
      Ubuntu/Codex              NAS/Codex
      global AGENTS             global AGENTS
```

GitHub Copilot을 추가한다면 공식적으로 지원하는 user/repository/path/agent instruction 계층을 같은 Source-of-Truth 원칙에 맞춰 연결할 수 있다.

결과적으로 일반 ChatGPT 새 채팅, GitHub repository에서 시작한 Agent, Ubuntu 개발 서버의 Codex, NAS에서 격리 실행되는 Codex가 모두 **같은 중앙 운영 기준을 발견하되 실제 프로젝트 상태는 각자의 Source of Truth에서 읽는 구조**가 된다.
