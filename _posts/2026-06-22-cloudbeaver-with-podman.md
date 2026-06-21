---
layout: post
title: "Podman 기반 CloudBeaver 운영 검토"
date: 2026-06-22
categories: [infrastructure]
tags: [Podman, CloudBeaver, Database, Operations, Container]
---

## 문제점

분석가나 개발자가 운영 DB 또는 분석 DB에 접근해야 할 때, 로컬 PC에 DB 클라이언트를 각각 설치하고 네트워크를 개별로 허용하는 방식은 관리 비용이 큽니다.

특히 다음 상황에서는 웹 기반 DB 클라이언트가 필요해질 수 있습니다.

- 사용자 PC에서 DB 포트 접근이 제한됩니다.
- 분석 서버에서는 DB 접근이 가능하지만 개인 PC에서는 접근이 어렵습니다.
- 여러 사용자가 같은 분석 서버를 통해 DB를 조회해야 합니다.
- DBeaver 같은 클라이언트를 각자 설치하고 설정하는 방식이 번거롭습니다.
- SSH는 가능하지만 GUI 기반 DB 조회 환경이 필요합니다.

이때 CloudBeaver Community Edition을 분석 서버에 올리고, 사용자는 웹 브라우저로 접속하게 하는 구성을 검토할 수 있습니다.

## 원인

DB 접근 환경이 복잡해지는 원인은 대개 다음과 같습니다.

1. DB 접근 허용 네트워크가 제한되어 있습니다.
2. 사용자 PC마다 보안 정책과 설치 환경이 다릅니다.
3. 분석 서버는 DB 접근이 가능하지만 사용자별 편의 도구가 부족합니다.
4. Docker가 없고 Podman만 사용 가능한 서버 환경이 있습니다.
5. root 권한 없이 일반 계정으로 서비스를 운영해야 합니다.

이 경우 Docker Compose 기반 예제를 그대로 적용하기 어렵고, rootless Podman 기준으로 운영 방식을 다시 잡아야 합니다.

## 해결

기본 방향은 다음입니다.

```text
분석 서버에 CloudBeaver 컨테이너 실행
-> 웹 포트만 사용자에게 제공
-> DB 접속 정보는 CloudBeaver 내부에서 관리
-> 데이터 조회/분석 작업은 브라우저 기반으로 수행
```

다만 운영 기준을 명확히 해야 합니다.

- 누구에게 웹 접근을 허용할 것인가
- CloudBeaver 계정 관리를 어떻게 할 것인가
- DB 계정 권한은 읽기 전용으로 제한할 것인가
- 컨테이너 데이터 디렉터리는 어디에 둘 것인가
- 서버 재부팅 후 자동 기동이 필요한가

## 실행 방법

### 1. 작업 디렉터리 생성

root 권한 없이 일반 계정으로 운영한다면 홈 디렉터리 하위에 두는 것이 단순합니다.

```bash
mkdir -p ~/services/cloudbeaver/workspace
cd ~/services/cloudbeaver
```

### 2. Podman 이미지 실행

```bash
podman run -d \
  --name cloudbeaver \
  -p 8978:8978 \
  -v ~/services/cloudbeaver/workspace:/opt/cloudbeaver/workspace \
  dbeaver/cloudbeaver:latest
```

실행 상태를 확인합니다.

```bash
podman ps
podman logs -f cloudbeaver
```

### 3. 방화벽 또는 리버스 프록시 검토

내부 사용자에게만 제공한다면 서버의 8978 포트를 제한된 네트워크에만 허용합니다.

```text
http://server-address:8978
```

외부 노출이 필요하다면 Nginx 또는 Apache 리버스 프록시를 통해 HTTPS를 적용하는 것이 좋습니다.

예시 구조:

```text
사용자 브라우저 -> HTTPS 443 -> Nginx/Apache -> CloudBeaver 8978
```

### 4. DB 권한 제한

CloudBeaver가 편하다고 해서 강한 권한의 DB 계정을 그대로 등록하면 위험합니다.

권장 기준은 다음입니다.

- 조회 목적: read-only 계정 사용
- 분석 목적: 별도 schema 또는 sandbox DB 사용
- 운영 DB 직접 수정 권한 금지
- 사용자별 계정 또는 최소 권한 계정 사용
- 접속 정보 공유 금지

### 5. 데이터 디렉터리 백업

CloudBeaver 설정은 workspace에 저장됩니다. 따라서 컨테이너만 삭제하면 설정이 사라질 수 있습니다.

```bash
ls -la ~/services/cloudbeaver/workspace
```

백업 대상은 다음입니다.

```text
~/services/cloudbeaver/workspace
```

## 검증 방법

정상 운영 여부는 다음 기준으로 확인합니다.

- `podman ps`에서 CloudBeaver 컨테이너가 실행 중입니다.
- 브라우저에서 CloudBeaver 로그인 화면이 열립니다.
- DB 연결 테스트가 성공합니다.
- 등록된 DB 계정이 필요한 권한만 가지고 있습니다.
- 서버 재부팅 후 수동 또는 자동으로 재기동할 수 있습니다.

## 재발 방지 / 개선 방향

운영 목적으로 사용하려면 다음 개선이 필요합니다.

1. CloudBeaver 전용 서비스 디렉터리를 표준화합니다.
2. DB 연결 계정은 최소 권한 원칙을 적용합니다.
3. 가능하면 HTTPS 리버스 프록시를 적용합니다.
4. 접근 가능한 사용자 네트워크를 제한합니다.
5. workspace 디렉터리 백업 기준을 정합니다.
6. 컨테이너 재기동 절차를 문서화합니다.

systemd 사용자 서비스로 등록하면 재부팅 후 자동 기동도 가능합니다. 다만 초기 검토 단계에서는 수동 기동으로 충분한지 먼저 판단하는 것이 좋습니다.

## 포트폴리오 관점의 의미

이 사례는 단순히 DB 툴을 설치하는 문제가 아니라, 분석가와 개발자가 사용할 수 있는 공용 데이터 접근 환경을 어떻게 설계할지에 대한 판단입니다.

특히 다음 역량과 연결됩니다.

- rootless Podman 운영 이해
- 웹 기반 DB 클라이언트 도입 검토
- DB 접근 권한과 편의성의 균형 판단
- 분석 서버 기반 협업 환경 구성
- 운영 환경에서 보안과 생산성을 함께 고려하는 능력
