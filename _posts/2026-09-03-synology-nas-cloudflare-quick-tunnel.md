---
layout: post
title: "Synology NAS의 로컬 웹서비스를 Docker cloudflared Quick Tunnel로 외부에 공유하기"
description: "Synology NAS의 localhost 웹서비스를 인터넷에 직접 포트포워딩하지 않고 Docker로 cloudflared Quick Tunnel을 실행해 임시 외부 URL로 공유한 구성, 명령어, 검증 방법과 운영 전환 기준을 정리합니다."
date: 2026-09-03
categories: [infrastructure]
tags: [Synology, NAS, Docker, Cloudflare Tunnel, cloudflared, Quick Tunnel, Reverse Proxy, Networking]
tistory:
  publish: true
  category: ""
---

Synology NAS에서 개발 중인 웹 화면을 휴대폰으로 확인해야 했습니다. 처음에는 NAS의 포트를 인터넷에 직접 열거나 공유기 포트포워딩을 추가하는 방법을 생각할 수 있지만, 단기 퍼블리싱 검토에는 그보다 **로컬 웹서버는 NAS 내부에만 열고, `cloudflared`가 Cloudflare로 outbound 터널을 만드는 구조**가 더 단순했습니다.

여기서 한 가지를 정확히 구분해야 합니다.

> **`cloudflared`가 웹서버 역할까지 하는 것은 아닙니다.**  
> 실제 HTML/CSS/JS를 제공하는 로컬 웹서버가 별도로 실행되고, Docker 컨테이너의 `cloudflared`는 그 웹서버를 외부 URL에 연결하는 터널 역할만 합니다.

실제로 사용한 구조는 다음과 같습니다.

```text
GitHub repository
      |
      | 최신 정적 파일 배포
      v
Synology NAS
      |
      | 127.0.0.1:18088
      v
Local static web server
(Python http.server)
      ^
      |
      | Cloudflare Tunnel
      |
Docker: cloudflared
      |
      | outbound connection
      v
Cloudflare Network
      |
      v
https://<random>.trycloudflare.com
      |
      v
휴대폰 / 외부 PC 브라우저
```

이 방식으로 NAS 관리 포트나 애플리케이션 포트를 인터넷에 직접 노출하지 않고도 외부 브라우저에서 퍼블리싱 결과를 확인할 수 있었습니다.

## 문제점

정적 HTML 퍼블리싱 결과를 NAS에 올린 뒤 외부에서 확인하려면 다음 중 하나가 필요합니다.

- 공유기에서 포트포워딩을 추가한다.
- NAS의 Reverse Proxy와 DDNS를 구성한다.
- VPN/Tailscale에 모든 테스트 단말을 연결한다.
- 외부의 별도 정적 호스팅 서비스를 사용한다.
- NAS에서 외부로 터널을 만든다.

장기 서비스라면 고정 도메인과 인증 정책까지 설계하는 것이 맞지만, 화면을 빠르게 검토하는 단계에서는 공유기와 DSM 설정을 반복해서 바꾸고 싶지 않았습니다.

또한 정적 퍼블리싱 화면 때문에 NAS의 관리 서비스나 다른 컨테이너까지 외부에 노출하는 것도 피하고 싶었습니다.

## 선택한 구조

구조를 두 단계로 나눴습니다.

### 1. 웹서버는 NAS의 loopback에만 바인딩

배포된 정적 파일을 NAS의 전용 디렉터리에 두고 다음처럼 웹서버를 실행합니다.

```bash
python3 -m http.server 18088 \
  --bind 127.0.0.1 \
  --directory /path/to/static-site
```

핵심은 다음 부분입니다.

```text
--bind 127.0.0.1
```

따라서 NAS의 LAN IP나 공인 IP에서 `18088` 포트로 직접 접근할 수 있게 여는 것이 아니라, **NAS 내부 프로세스만 접근할 수 있는 웹서비스**로 둡니다.

먼저 NAS 내부에서 정상인지 확인합니다.

```bash
curl -fsS http://127.0.0.1:18088/
```

이 단계가 실패하면 Cloudflare Tunnel을 확인할 필요가 없습니다. 먼저 로컬 웹서버부터 해결해야 합니다.

## Docker로 cloudflared Quick Tunnel 실행

Cloudflare의 Quick Tunnel은 로컬 웹서비스를 임시 `*.trycloudflare.com` 주소로 연결할 수 있습니다.

가장 단순한 형태는 다음과 같습니다.

```bash
cloudflared tunnel --url http://localhost:18088
```

NAS에서는 `cloudflared`를 별도 설치하기보다 Docker 컨테이너로 격리했습니다.

예시는 다음과 같습니다.

```bash
docker run -d \
  --name static-preview-tunnel \
  --network host \
  --read-only \
  --cap-drop ALL \
  --security-opt no-new-privileges:true \
  --tmpfs /tmp:rw,nosuid,nodev,size=16m \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate \
  --url http://127.0.0.1:18088
```

이 구성에서 각 역할은 다음과 같습니다.

| 구성 | 역할 |
|---|---|
| `python3 -m http.server` | 실제 HTML/CSS/JS 응답 |
| `127.0.0.1:18088` | NAS 내부에서만 접근 가능한 origin |
| Docker | `cloudflared` 프로세스 격리 |
| `cloudflared` | Cloudflare로 outbound 터널 생성 |
| `trycloudflare.com` | 임시 외부 접근 URL |

`--network host`를 사용한 이유는 컨테이너의 `cloudflared`가 NAS host의 `127.0.0.1:18088` origin에 접근하도록 하기 위해서입니다.

컨테이너에는 웹사이트 파일이나 NAS 관리 권한이 필요하지 않으므로 가능한 권한을 줄였습니다.

```text
read-only filesystem
capabilities drop
no-new-privileges
temporary /tmp only
```

## 외부 URL 확인

Quick Tunnel을 시작하면 `cloudflared` 로그에 임시 URL이 출력됩니다.

```bash
docker logs static-preview-tunnel
```

예를 들면 다음 형태입니다.

```text
https://random-words.trycloudflare.com
```

그 다음 두 구간을 각각 확인합니다.

### NAS 내부 origin 검증

```bash
curl -fsS http://127.0.0.1:18088/ >/dev/null
echo $?
```

### Cloudflare 경유 외부 검증

```bash
curl -fsS https://random-words.trycloudflare.com/ >/dev/null
echo $?
```

둘 다 성공해야 합니다.

```text
Local origin PASS
     +
Public tunnel PASS
     =
외부 브라우저 검토 가능
```

Cloudflare URL만 확인하면 origin 문제와 tunnel 문제를 분리하기 어렵기 때문에 이 두 단계 검증을 분리하는 편이 좋습니다.

## GitHub에서 NAS까지 자동 배포

퍼블리싱 화면을 수정할 때마다 NAS에 파일을 직접 복사하는 대신 다음 흐름으로 정리했습니다.

```text
GitHub main
   |
   | 검증
   v
정적 HTML/CSS/JS
   |
   | NAS deploy
   v
NAS 전용 site directory
   |
   v
127.0.0.1 local web server
   |
   v
cloudflared Quick Tunnel
   |
   v
외부 휴대폰 검토
```

배포 시에는 다음을 확인합니다.

1. GitHub의 최신 commit을 가져옵니다.
2. 지정된 정적 파일만 site 디렉터리에 복사합니다.
3. 기존 local HTTP server를 종료하고 새 파일 기준으로 다시 시작합니다.
4. `127.0.0.1`에서 HTTP 응답을 확인합니다.
5. 기존 Quick Tunnel 컨테이너를 교체합니다.
6. 새 `trycloudflare.com` URL을 얻습니다.
7. 외부 URL의 HTTP 응답까지 확인합니다.

화면 개발 과정에서는 JavaScript 문법과 필수 파일 존재 여부도 CI에서 먼저 검사하도록 했습니다.

## 이 방식의 장점

### 공유기 포트포워딩이 필요하지 않음

Cloudflare Tunnel은 NAS에서 Cloudflare 쪽으로 연결을 시작합니다.

```text
기존 포트포워딩
Internet -> Router -> NAS

Cloudflare Tunnel
NAS -> Cloudflare
          ^
          |
      Internet user
```

따라서 preview를 만들기 위해 공유기의 inbound port를 하나씩 추가할 필요가 없습니다.

### NAS origin 포트를 외부에 직접 열 필요가 없음

웹서버는 계속 다음 주소에만 존재합니다.

```text
127.0.0.1:18088
```

외부 사용자는 NAS의 이 포트로 직접 접근하는 것이 아니라 Cloudflare URL을 통해 접근합니다.

### 임시 화면 공유에 빠름

도메인을 별도로 준비하지 않아도 URL이 바로 발급되므로 다음 용도에는 편리합니다.

- 모바일 화면 검토
- 퍼블리싱 결과 공유
- 개발 중인 정적 UI 확인
- 브라우저 호환성 확인
- 짧게 유지하는 demo/preview

## 주의점: Quick Tunnel은 운영 서비스가 아님

Cloudflare 공식 문서에서도 Quick Tunnel은 **testing/development 용도**로 안내합니다.

특히 다음 특성이 있습니다.

- 새 tunnel process를 시작하면 랜덤 `trycloudflare.com` 주소가 바뀔 수 있습니다.
- SLA가 보장되는 production endpoint가 아닙니다.
- 동시 in-flight request 제한이 있습니다.
- Server-Sent Events(SSE)는 지원하지 않습니다.

따라서 다음 단계로 넘어가면 Quick Tunnel을 그대로 운영 서비스로 사용하지 않는 것이 좋습니다.

```text
Quick Tunnel
  -> UI 검토 / 개발 / 임시 공유

Named Cloudflare Tunnel
  -> 고정 hostname
  -> 정식 접근정책
  -> 필요 시 Cloudflare Access
  -> 지속 서비스
```

공식 문서:

- [Cloudflare Quick Tunnels](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/)
- [Cloudflare Tunnel Setup](https://developers.cloudflare.com/tunnel/setup/)

## 보안상 특히 주의할 점

Tunnel을 쓴다고 해서 **아무 내부 서비스를 공개해도 안전해지는 것은 아닙니다.**

이번 방식은 공개해도 되는 synthetic 정적 UI를 확인하는 용도로 사용했습니다.

다음 대상은 인증 없이 Quick Tunnel로 바로 공개하지 않는 편이 좋습니다.

- DSM 관리자 화면
- NAS 관리 API
- 데이터베이스
- SSH
- 개인정보가 있는 애플리케이션
- 사용자별 진행 상태를 저장하는 내부 서비스
- 회사/고객 데이터가 보이는 시스템

preview 용도라면 다음 기준을 적용할 수 있습니다.

```text
공개 가능한 정적 데이터만 사용
origin은 loopback에만 bind
외부 API 호출 최소화
secret을 HTML/JS에 넣지 않음
관리 서비스와 별도 runtime 사용
필요 없으면 tunnel 즉시 종료
```

검색 노출 자체가 필요 없는 검토 페이지라면 HTML에 다음을 추가할 수도 있습니다.

```html
<meta name="robots" content="noindex,nofollow,noarchive">
```

다만 이것은 접근통제가 아닙니다. 민감한 서비스라면 Named Tunnel과 실제 인증/접근정책을 사용해야 합니다.

## 장애를 나눠서 확인하는 방법

### 페이지 자체가 열리지 않는 경우

```bash
curl http://127.0.0.1:18088/
```

부터 확인합니다.

실패한다면 정적 웹서버 문제입니다.

### NAS에서는 열리지만 외부 URL이 안 되는 경우

```bash
docker ps
docker logs static-preview-tunnel
```

으로 `cloudflared` 상태와 발급 URL을 확인합니다.

### 재배포 후 URL이 달라진 경우

Quick Tunnel의 정상적인 특성입니다. 새로운 `cloudflared` 프로세스가 시작되면 새 랜덤 URL이 생성될 수 있습니다.

고정 URL이 필요하면 Quick Tunnel이 아니라 Named Tunnel로 전환합니다.

## 정리

이번에 사용한 구성은 다음 한 줄로 요약할 수 있습니다.

> **NAS에는 localhost 웹서버만 열고, Docker로 실행한 `cloudflared`가 outbound Quick Tunnel을 만들어 임시 외부 URL로 연결한다.**

이 방식은 NAS에 정적 퍼블리싱 결과를 올려 실제 휴대폰 화면을 검토할 때 상당히 편리했습니다.

다만 `trycloudflare.com` Quick Tunnel은 어디까지나 개발/검토용입니다. 화면과 기능이 안정된 뒤에는 고정 hostname, 인증, 접근정책을 갖춘 Named Tunnel 또는 다른 정식 배포 구조로 전환하는 것이 적절합니다.
