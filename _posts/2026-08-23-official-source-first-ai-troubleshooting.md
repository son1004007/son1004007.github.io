---
layout: post
title: "AI와 운영 문제를 해결할 때 공식 문서를 기본값으로 사용하는 방법"
date: 2026-08-23
categories: [infrastructure]
tags: [AI Agent, Troubleshooting, Official Documentation, Linux, Codex]
---

AI에게 익숙하지 않은 도구의 설치나 운영 문제를 맡기면 가장 위험한 순간은 "대충 아는 내용"을 확정적인 사실처럼 사용하는 때다. 특히 CLI 옵션, 보안 샌드박스, 인증, 컨테이너 런타임처럼 버전과 OS 정책에 따라 동작이 달라지는 영역은 기억이나 검색 결과만으로 변경하면 불필요한 우회 설정을 만들기 쉽다.

최근 Linux 환경의 AI CLI 샌드박스 문제를 다루면서 이 원칙을 다시 확인했다. 처음에는 컨테이너 런타임을 별도로 도입하는 방향도 검토했지만, 설치된 버전과 Ubuntu의 AppArmor 정책, Bubblewrap 동작을 공식 자료와 실제 런타임으로 다시 확인하자 더 단순한 해결 경로가 있었다.

이 경험을 계기로 AI-assisted development/operations의 기본 규칙을 다음처럼 정리했다.

> 모르는 부분, 기억이 불확실한 부분, 버전에 따라 달라질 가능성이 있는 부분은 추측하지 않고 공식 문서를 먼저 확인한다.

## 문제점

AI는 일반적인 기술 패턴을 빠르게 제안할 수 있지만 다음 상황에서는 오래된 지식이나 다른 버전의 동작을 섞을 가능성이 있다.

- CLI 문법이나 옵션이 버전별로 바뀐 경우
- Ubuntu, RHEL, Synology DSM처럼 OS 보안 정책이 다른 경우
- Docker, Bubblewrap, AppArmor, seccomp처럼 여러 보안 계층이 겹치는 경우
- OAuth, API Key, Device Login 등 인증 흐름이 바뀐 경우
- 공식 지원 방식과 커뮤니티 우회 방법이 함께 검색되는 경우
- 에러 메시지만 보고 원인을 단정하기 어려운 경우

이때 바로 설정을 변경하면 원래 필요하지 않았던 Docker 설치, 전역 보안 정책 완화, 위험한 bypass 옵션 같은 부작용이 생길 수 있다.

## 원인

가장 큰 원인은 "지식"과 "현재 환경의 사실"을 구분하지 않는 것이다.

예를 들어 어떤 도구가 과거에는 다음과 같은 명령을 사용했다고 하더라도:

```bash
some-cli sandbox linux /bin/true
```

현재 설치된 버전에서는 다음처럼 문법이 바뀌었을 수 있다.

```bash
some-cli sandbox -- /bin/true
```

이 차이는 기억으로 해결할 문제가 아니다. 설치된 바이너리의 `--version`, `--help`와 현재 공식 문서를 확인해야 한다.

운영 환경에서는 세 종류의 근거를 분리해야 한다.

1. **현재 환경의 사실**: 실제 서버의 버전, 설정, 에러, exit code
2. **제품의 지원 방식**: 공식 문서, 공식 release note, 공식 source repository
3. **해결 가능성**: 최소 변경의 read-only probe 또는 재현 테스트

## 해결

나는 이 절차를 **Official-Source-First** 규칙으로 사용한다.

### 1. 먼저 현재 환경을 관측한다

변경 전에 최소한 다음을 확인한다.

```bash
tool --version
tool --help
uname -a
```

필요하면 OS/package 상태도 읽기 전용으로 확인한다.

```bash
dpkg-query -W <package>
sysctl <relevant-key>
```

중요한 점은 아직 설정을 변경하지 않는 것이다.

### 2. 불확실하면 공식 문서를 기본 검색 대상으로 한다

검색 우선순위는 다음과 같이 둔다.

```text
1. 현재 설치된 바이너리의 --version / --help
2. 제품의 최신 공식 문서
3. 공식 release note / changelog
4. 공식 source repository의 문서와 issue
5. 재현 가능한 read-only runtime probe
6. Stack Overflow, Reddit, 블로그 등 커뮤니티 자료
```

커뮤니티 자료는 문제 발견에는 유용하지만 최종 운영 설정의 근거로 바로 사용하지 않는다.

### 3. 버전이 맞는지 확인한다

공식 문서가 최신 버전을 설명하고 서버에는 이전 버전이 설치되어 있을 수 있다.

따라서 다음 질문을 항상 확인한다.

- 이 문서는 현재 설치 버전에 적용되는가?
- 옵션이 현재 `--help`에 실제로 존재하는가?
- release note에서 변경된 동작은 없는가?
- OS 버전에 따라 추가 정책이 필요한가?

공식 문서와 설치된 바이너리가 충돌하면 **현재 설치된 바이너리의 실제 동작을 우선 관측 사실로 기록하고**, 왜 차이가 생겼는지 release note나 source를 추가 확인한다.

### 4. 가장 작은 변경으로 검증한다

바로 운영 구성을 크게 바꾸지 않는다.

예를 들어 sandbox 문제가 발생했다면:

```text
나쁜 순서
에러 발생
 -> 전역 보안 설정 비활성화
 -> Docker 설치
 -> 전체 구조 변경

좋은 순서
에러 발생
 -> 버전/도움말 확인
 -> 공식 문서 확인
 -> 필요한 정책 한 개만 변경
 -> 최소 명령으로 exit code 확인
 -> 실제 read-only E2E 확인
```

### 5. 성공 조건을 수치 또는 상태로 남긴다

"되는 것 같다"가 아니라 다음처럼 남긴다.

```text
probe exit code = 0
read-only violation = no
workspace cleanup = yes
CI = success
```

이 기록이 있어야 이후 AI가 같은 문제를 다시 추측하지 않는다.

## 실행 방법

AI에게 기술 작업을 시킬 때 프롬프트나 저장소의 `AGENTS.md`에 다음 규칙을 넣어 두면 효과적이다.

```text
When a behavior is unfamiliar, version-sensitive, security-sensitive,
or not directly verified:

1. Do not guess.
2. Inspect the installed version/help/runtime state first.
3. Consult current official vendor documentation.
4. Check official release notes/source issues when docs and runtime differ.
5. Prefer the smallest reversible/read-only probe before configuration changes.
6. Do not weaken global security settings merely to make a tool work.
7. Record the exact evidence used for the decision.
```

한국어로는 다음 정도면 충분하다.

```text
잘 모르는 동작, 버전 의존 동작, 보안/인증/런타임 설정은 추측하지 않는다.
현재 버전과 실제 상태를 먼저 확인하고 공식 문서를 조회한다.
공식 문서와 실제 바이너리가 다르면 release note/source를 추가 확인한다.
가능하면 read-only 최소 재현 후에만 설정을 변경한다.
```

## 검증 방법

이 규칙이 실제로 동작하려면 문서만 작성해서는 부족하다.

저장소 수준에서는 다음을 같이 적용하는 것이 좋다.

- `AGENTS.md`에 Official-Source-First 규칙 추가
- 중요한 런타임 결정은 별도 문서에 근거 URL과 검증 결과 기록
- CI에서 제거된 과거 정책 문구가 다시 들어오는지 검사
- 실제 서버 작업은 read-only/status operation부터 수행
- 변경 후 E2E 증적을 Issue 또는 Actions log에 남김

즉 정책, 코드, 런타임 증거가 서로 일치해야 한다.

## 재발 방지 / 개선 방향

AI 작업에서 특히 다음 단어가 등장하면 공식 문서 확인을 자동 트리거로 보는 편이 안전하다.

```text
모르겠다
아마
예전에는
버전에 따라
permission denied
operation not permitted
unsupported
unknown option
authentication
sandbox
AppArmor
seccomp
Docker
OAuth
```

또한 운영 서버에서 다음 변경은 공식 근거 없이 바로 진행하지 않는다.

- kernel/sysctl 보안 정책 완화
- AppArmor/SELinux 전체 비활성화
- `--dangerously-*`, `--privileged` 같은 bypass
- Docker socket 노출
- root 권한 확대
- 인증 파일 전체 공유

가능하면 항상 **국소적인 허용 > 전역 보안 완화** 순서로 판단한다.

## 포트폴리오 관점의 의미

이 원칙은 단순히 "검색을 잘한다"는 의미가 아니다.

운영과 개발에서 중요한 것은 다음 능력이다.

- 현재 런타임과 문서의 차이를 구분하는 능력
- 보안 설정을 불필요하게 약화하지 않는 판단
- 최소 변경으로 원인을 검증하는 troubleshooting 방식
- AI의 제안을 그대로 실행하지 않고 근거를 확인하는 검증 습관
- 변경 후 재현 가능한 증적을 남기는 운영 방식

AI 도구를 많이 사용하는 환경일수록 중요한 것은 더 많은 명령을 빠르게 실행하는 것이 아니라, **불확실한 순간에 추측을 멈추고 가장 신뢰할 수 있는 근거로 전환하는 것**이다.
