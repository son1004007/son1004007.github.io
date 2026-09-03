---
layout: post
title: "SSH -L 로컬 포트 포워딩: 네트워크 구조와 실무 예제"
description: "SSH -L 로컬 포트 포워딩의 동작 구조를 네트워크 흐름으로 설명하고, 내부 웹서비스·PostgreSQL·Jupyter 접근 예제와 CLI 서버의 웹 인증 구성, 보안 및 장애 대응 방법을 정리합니다."
date: 2026-09-03
categories: [infrastructure]
tags: [SSH, OpenSSH, Port Forwarding, Tunnel, Network, Linux]
tistory:
  publish: true
  category: ""
---

`ssh -L`은 먼저 **어느 노드가 어디로 접속하는지**를 이해하면 쉽습니다.

가장 중요한 구조는 다음과 같습니다.

```text
[Node A: ssh -L 명령을 실행한 노드]
Application
  |
  | 127.0.0.1:LOCAL_PORT
  v
SSH Client
  ||
  || SSH encrypted channel
  \/
[Node B: SSH_SERVER]
  |
  | TCP connection
  v
[TARGET_IP:TARGET_PORT]
```

즉 `ssh -L` 명령을 실행한 **Node A에 LOCAL_PORT가 열리고**, 그 포트로 들어온 연결은 **Node B인 SSH 서버를 경유한 뒤**, SSH 서버가 `TARGET_IP:TARGET_PORT`로 실제 TCP 연결을 만듭니다.

따라서 가장 중요한 기준은 다음 한 문장입니다.

> `TARGET_IP` 또는 `destination_host`는 내 PC 기준 목적지가 아니라, **SSH_SERVER가 접속할 수 있는 네트워크 기준의 목적지**입니다.

예를 들어 다음 명령을 실행한다고 가정합니다.

```bash
ssh -N \
  -L 127.0.0.1:18080:10.0.20.15:8080 \
  user@bastion.example.com
```

이 경우 연결 흐름은 다음과 같습니다.

```text
내 PC의 127.0.0.1:18080
  -> bastion.example.com SSH 연결
  -> Bastion이 10.0.20.15:8080으로 접속
```

내 PC가 `10.0.20.15`에 직접 접근할 수 없어도 됩니다. **Bastion이 해당 IP와 포트에 접근할 수 있으면**, 내 PC에서는 `127.0.0.1:18080`으로 접속해 그 서비스를 사용할 수 있습니다.

이 구조가 `ssh -L`의 핵심입니다.

## 문제점

실무에서는 다음과 같은 상황을 자주 만납니다.

- SSH 서버가 접근할 수 있는 내부망 서비스에 내 PC에서는 직접 접근할 수 없습니다.
- SSH 서버의 `127.0.0.1:8080`에서만 실행되는 웹 애플리케이션을 내 PC 브라우저에서 확인해야 합니다.
- 외부에서 직접 접근할 수 없는 PostgreSQL, MySQL 같은 내부 DB에 접속해야 합니다.
- 원격 서버의 Jupyter, 개발 서버, 관리 UI를 인터넷에 공개하고 싶지 않습니다.
- 방화벽을 열거나 서비스의 listen 주소를 `0.0.0.0`으로 변경하지 않고 임시로 접근해야 합니다.
- Bastion 또는 Jump Host를 통해서만 내부망 서비스에 접근할 수 있습니다.

이런 경우 서비스 포트를 외부에 직접 공개하는 대신, 이미 인증과 암호화가 적용된 SSH 연결을 통로로 사용할 수 있습니다.

## 핵심 개념과 문법

OpenSSH의 기본 문법은 다음과 같습니다.

```bash
ssh -L [bind_address:]local_port:destination_host:destination_port [user@]ssh_server
```

실무에서는 로컬 접근만 허용하도록 `bind_address`를 명시하는 편이 안전합니다.

```bash
ssh -L 127.0.0.1:local_port:destination_host:destination_port user@ssh_server
```

각 값의 의미는 다음과 같습니다.

| 항목 | 어느 노드 기준인가 | 의미 |
|---|---|---|
| `127.0.0.1` | `ssh -L` 명령을 실행한 로컬 노드 | 포트를 열 주소 |
| `local_port` | `ssh -L` 명령을 실행한 로컬 노드 | 애플리케이션이 접속할 포트 |
| `destination_host` | SSH 서버 기준 | SSH 서버가 접속할 대상 호스트/IP |
| `destination_port` | SSH 서버 기준 | 대상 서비스 포트 |
| `ssh_server` | 로컬 노드가 접속하는 서버 | 터널을 중계할 SSH 서버 |

특히 `destination_host`를 어느 노드 기준으로 해석하는지가 중요합니다.

```text
ssh -L 127.0.0.1:18080:127.0.0.1:8080 user@server.example.com
                         ^^^^^^^^^
```

여기서 두 번째 `127.0.0.1`은 **내 PC가 아니라 SSH 서버 자신의 loopback 주소**입니다.

반대로 다음처럼 내부 IP를 지정했다면,

```bash
ssh -L 127.0.0.1:18080:10.0.20.15:8080 user@bastion.example.com
```

`10.0.20.15:8080`에 접속하는 주체는 내 PC가 아니라 `bastion.example.com`입니다.

OpenSSH는 내 PC의 로컬 포트에서 연결을 받은 뒤 SSH 암호화 채널을 통해 SSH 서버로 전달하고, SSH 서버가 다시 `destination_host:destination_port`로 TCP 연결을 만듭니다.

즉 구조를 한 줄로 표현하면 다음과 같습니다.

```text
LOCAL_NODE:LOCAL_PORT
  -> SSH_SERVER
  -> SSH_SERVER가 접근 가능한 DESTINATION_HOST:DESTINATION_PORT
```

## CLI만 가능한 Linux 서버에서 웹 인증이 필요한 경우

`ssh -L`은 DB나 웹서비스 접근뿐 아니라 **GUI가 없는 Linux 서버에서 웹 로그인 화면이 필요한 자동화 작업**에도 유용합니다.

예를 들어 다음과 같은 경우입니다.

- CLI 전용 서버에서 Tistory/Kakao 로그인을 최초 한 번 수행해야 합니다.
- OAuth 로그인 중 브라우저 승인이 필요합니다.
- 2차 인증, OTP 승인, CAPTCHA 등 때문에 완전한 headless 로그인 자동화가 적절하지 않습니다.
- 로그인 이후에는 Playwright 같은 브라우저 자동화가 저장된 세션을 재사용해야 합니다.

가능하면 서비스가 공식적으로 device-code나 CLI 인증 방식을 제공할 때는 그 방식을 먼저 사용하는 것이 단순합니다. 하지만 **실제 브라우저 화면에서 로그인을 완료해야 하는 서비스라면**, 서버에 임시 브라우저를 띄우고 그 화면만 로컬 PC로 전달할 수 있습니다.

### 구성

```text
[내 PC]
Browser
  |
  | http://127.0.0.1:16080
  v
127.0.0.1:16080
  |
  | ssh -L
  v
[CLI Linux Server]
127.0.0.1:6080
[noVNC]
  |
  v
VNC / Virtual Display
  |
  v
Chromium / Playwright Browser
  |
  v
Tistory / Kakao / OAuth Provider
```

중요한 점은 **브라우저 프로세스와 로그인 세션은 Linux 서버에 존재하고, 화면만 SSH 터널을 통해 내 PC 브라우저에서 보는 것**입니다.

따라서 인증이 끝난 뒤 서버에 남은 브라우저 세션을 이후 자동화에서 그대로 사용할 수 있습니다.

### 1. Linux 서버에 임시 GUI 브라우저 준비

CLI 서버에서는 일반적으로 다음 구성 중 하나를 사용합니다.

```text
Xvfb 또는 가상 디스플레이
  + Chromium/Chrome
  + VNC server
  + noVNC
```

컨테이너로 이 구성을 격리해도 됩니다. 인증용 브라우저는 장시간 외부에 공개해 두기보다 **필요할 때만 임시로 실행하고 인증이 끝나면 종료**하는 편이 안전합니다.

### 2. noVNC를 서버의 localhost에만 열기

VNC 서버가 `127.0.0.1:5901`에서 실행 중이라고 가정하면 noVNC의 `novnc_proxy`는 다음과 같이 localhost에만 바인딩할 수 있습니다.

```bash
./utils/novnc_proxy \
  --vnc localhost:5901 \
  --listen localhost:6080
```

이렇게 하면 `6080` 포트는 인터넷에 직접 노출되지 않습니다.

### 3. 내 PC에서 SSH 로컬 포워딩

내 PC에서 다음 명령을 실행합니다.

```bash
ssh -NT \
  -o ExitOnForwardFailure=yes \
  -L 127.0.0.1:16080:127.0.0.1:6080 \
  user@cli-server.example.com
```

이번 명령도 앞에서 설명한 구조와 동일합니다.

```text
내 PC 127.0.0.1:16080
  -> SSH
  -> cli-server.example.com
  -> 서버 자신의 127.0.0.1:6080
```

내 PC 브라우저에서는 다음 주소를 엽니다.

```text
http://127.0.0.1:16080/
```

그러면 실제로는 원격 Linux 서버에서 실행 중인 브라우저 화면을 보게 됩니다.

### 4. Tistory/Kakao 같은 웹 로그인을 직접 완료

이제 원격 브라우저 화면에서 정상적인 로그인 절차를 진행합니다.

```text
아이디/비밀번호 입력
  -> 필요한 경우 2차 인증
  -> CAPTCHA가 나오면 사용자가 직접 처리
  -> Tistory 관리 화면 등 인증 완료 상태 확인
```

OTP나 CAPTCHA를 자동 우회하는 것이 아니라 **사람이 필요한 인증 단계만 원격 브라우저에서 정상적으로 수행**하는 구조입니다.

### 5. 인증 세션을 서버에 저장

Playwright는 로그인된 Browser Context의 cookies, local storage 등 인증 상태를 `storageState` 파일로 저장하고 이후 새 Context에서 다시 사용할 수 있습니다.

예를 들면 다음과 같습니다.

```javascript
await context.storageState({
  path: '/var/lib/browser-auth/storage-state.json'
});
```

이후 자동화에서는 저장한 상태를 다시 로드할 수 있습니다.

```javascript
const context = await browser.newContext({
  storageState: '/var/lib/browser-auth/storage-state.json'
});
```

그러면 정상적인 세션 유효기간 동안에는 매번 사람이 로그인하지 않고 headless 브라우저가 인증 상태를 재사용할 수 있습니다.

### 6. 세션 파일은 인증정보로 취급

`storageState` 같은 파일에는 세션 쿠키 등 계정 접근에 사용할 수 있는 정보가 포함될 수 있습니다.

따라서 다음 원칙을 지키는 것이 중요합니다.

```text
Git repository에 commit하지 않기
로그나 CI artifact에 출력하지 않기
서버의 전용 디렉터리에만 저장
파일 권한을 최소화하기 (예: 0600)
자동화 컨테이너에도 필요한 파일만 mount
세션이 만료되면 기존 파일을 폐기하고 다시 인증
```

Playwright 공식 문서도 저장된 인증 상태 파일에 민감한 쿠키와 헤더가 포함될 수 있으므로 repository에 넣지 말 것을 권고합니다.

### 7. 운영 시에는 인증과 게시를 분리

Tistory 게시 자동화를 예로 들면 다음 구조가 실용적입니다.

```text
최초 또는 세션 만료 시
사용자
  -> ssh -L
  -> 임시 noVNC
  -> 원격 Chromium에서 로그인
  -> storageState 저장

평상시
Scheduler
  -> Headless Playwright
  -> storageState 로드
  -> Tistory 관리 화면 접근
  -> 글 작성/수정
  -> 결과 확인
```

자동 게시 중 로그인 페이지로 리다이렉트되거나 세션이 만료된 것이 확인되면, 비밀번호 재시도나 CAPTCHA 우회를 반복하는 대신 **게시 작업을 중단하고 재인증이 필요하다고 처리**하는 것이 안전합니다.

### SSH 포워딩 자체가 서버 정책으로 막혀 있다면

서버에서 `AllowTcpForwarding` 등이 정책상 비활성화되어 있으면 위 `ssh -L` 방식은 사용할 수 없습니다.

이 경우 포워딩 제한을 무조건 해제하기보다 다음 순서로 판단하는 편이 좋습니다.

1. 관리자가 허용한다면 특정 인증용 목적지만 `PermitOpen` 등으로 제한적으로 허용합니다.
2. 신뢰할 수 있는 사설 LAN 안의 서버라면 noVNC를 **LAN 주소에 임시로만 바인딩**하고 방화벽으로 접근 출발지를 제한하는 방식을 검토합니다.
3. 인증이 완료되면 임시 브라우저와 VNC/noVNC 서비스를 즉시 종료합니다.

즉 **웹 인증을 위해 CLI 서버 자체에 데스크톱 환경을 상시 공개할 필요는 없습니다.** 필요한 시점에만 임시 브라우저를 띄우고, `ssh -L`로 화면 접근 경로를 제한한 뒤, 인증 상태만 안전하게 보존하는 방식으로 구성할 수 있습니다.

## 예제 1. SSH 서버의 로컬 웹서비스 접속

원격 서버에서 다음 서비스가 실행 중이라고 가정합니다.

```text
SSH Server: server.example.com:22
Web App:    127.0.0.1:8080
```

외부에서는 `8080`에 직접 접근할 수 없고 SSH만 허용되어 있습니다.

### 네트워크 구성

```text
[내 PC]
Browser
  |
  | http://127.0.0.1:18080
  v
127.0.0.1:18080
  |
  | SSH encrypted channel
  v
[server.example.com:22]
sshd
  |
  | TCP connection from SSH server
  v
127.0.0.1:8080
[Web Application]
```

명령은 다음과 같습니다.

```bash
ssh -N -L 127.0.0.1:18080:127.0.0.1:8080 user@server.example.com
```

이후 내 PC 브라우저에서 접속합니다.

```text
http://127.0.0.1:18080
```

실제 흐름은 다음과 같습니다.

1. 브라우저가 내 PC의 `127.0.0.1:18080`에 접속합니다.
2. SSH 클라이언트가 해당 연결을 받습니다.
3. 데이터가 SSH 암호화 채널을 통해 `server.example.com`으로 전달됩니다.
4. SSH 서버가 자신의 `127.0.0.1:8080`으로 연결합니다.
5. 응답이 같은 경로를 반대로 돌아옵니다.

`-N`은 원격 쉘 명령을 실행하지 않고 포트 포워딩만 사용하겠다는 의미입니다.

## 예제 2. Bastion을 경유해 내부 PostgreSQL 접속

이번에는 DB가 SSH 서버와 다른 내부 호스트에 있다고 가정합니다.

```text
Internet
   |
   v
[Bastion]
bastion.example.com:22
   |
   | Internal Network
   v
[PostgreSQL]
db.internal.example:5432
```

내 PC에서 로컬 포트 `15432`를 사용합니다.

```bash
ssh -N \
  -L 127.0.0.1:15432:db.internal.example:5432 \
  user@bastion.example.com
```

### 네트워크 흐름

```text
[내 PC]
psql / DBeaver
  |
  | 127.0.0.1:15432
  v
SSH Client
  ||
  || encrypted SSH connection
  \/
[Bastion]
  |
  | db.internal.example:5432
  v
[PostgreSQL]
```

DB 클라이언트에서는 실제 DB 주소 대신 로컬 주소를 사용합니다.

```bash
psql -h 127.0.0.1 -p 15432 -U appuser appdb
```

DBeaver 같은 GUI 도구에서도 다음처럼 입력하면 됩니다.

```text
Host: 127.0.0.1
Port: 15432
```

여기서 중요한 조건은 **Bastion 서버가 `db.internal.example:5432`에 실제로 접근할 수 있어야 한다는 것**입니다.

내 PC가 내부 DB를 직접 볼 수 있을 필요는 없습니다.

## 예제 3. SSH 포트가 22가 아닐 때 Jupyter 접속

SSH 서버가 `2222` 포트를 사용하고 Jupyter가 원격 서버의 `127.0.0.1:8888`에서 실행 중이라고 가정합니다.

```bash
ssh -p 2222 -N \
  -L 127.0.0.1:18888:127.0.0.1:8888 \
  user@server.example.com
```

브라우저에서는 다음 주소로 접속합니다.

```text
http://127.0.0.1:18888
```

각 포트의 의미는 서로 다릅니다.

```text
2222  = SSH 서버 접속 포트
18888 = 내 PC에서 열리는 로컬 포트
8888  = 원격 서버의 Jupyter 포트
```

`-p`와 `-L`의 포트를 혼동하지 않는 것이 중요합니다.

## 예제 4. 하나의 SSH 연결로 여러 서비스 포워딩

`-L`은 여러 번 사용할 수 있습니다.

```bash
ssh -N \
  -L 127.0.0.1:15432:db.internal.example:5432 \
  -L 127.0.0.1:18080:app.internal.example:8080 \
  -L 127.0.0.1:19090:monitor.internal.example:9090 \
  user@bastion.example.com
```

이 경우 내 PC에서는 다음처럼 접근합니다.

```text
127.0.0.1:15432 -> PostgreSQL
127.0.0.1:18080 -> Internal Web App
127.0.0.1:19090 -> Monitoring Service
```

여러 내부 서비스를 잠깐 확인할 때 SSH 연결을 각각 만들 필요가 없습니다.

## 자주 사용하는 옵션

| 옵션 | 의미 | 실무 사용 기준 |
|---|---|---|
| `-L` | Local port forwarding | 핵심 옵션 |
| `-N` | 원격 명령을 실행하지 않음 | 터널 전용 연결에 권장 |
| `-T` | pseudo-terminal 할당 안 함 | 비대화형 터널에서 사용 가능 |
| `-p` | SSH 서버 포트 지정 | SSH가 22가 아닐 때 |
| `-i` | SSH private key 지정 | 특정 키를 사용할 때 |
| `-v`, `-vv`, `-vvv` | 디버그 로그 증가 | 연결 실패 분석 |
| `-f` | 인증 후 background 전환 | Unix 계열에서 터널을 백그라운드로 둘 때 |
| `-g` | 다른 호스트가 로컬 포워딩 포트에 접속하도록 허용 | 보안상 특별한 이유가 없으면 사용하지 않음 |

터널 전용으로는 다음처럼 사용할 수 있습니다.

```bash
ssh -NT \
  -o ExitOnForwardFailure=yes \
  -L 127.0.0.1:15432:db.internal.example:5432 \
  user@bastion.example.com
```

`ExitOnForwardFailure=yes`는 요청한 포워딩 리스너를 만들지 못했을 때 SSH 연결을 종료하도록 합니다.

다만 이것이 최종 대상 서비스의 지속적인 정상 접속까지 보장하는 것은 아닙니다. 예를 들어 로컬 리스너는 만들어졌지만 이후 DB가 중지되면 실제 DB 연결은 실패할 수 있습니다.

## ~/.ssh/config로 명령어 줄이기

반복해서 사용하는 터널은 `~/.ssh/config`에 저장하면 편합니다.

```sshconfig
Host work-db-tunnel
    HostName bastion.example.com
    User user
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 127.0.0.1:15432 db.internal.example:5432
    ExitOnForwardFailure yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

이후에는 다음만 실행하면 됩니다.

```bash
ssh -N work-db-tunnel
```

여러 포워딩도 추가할 수 있습니다.

```sshconfig
Host work-services
    HostName bastion.example.com
    User user
    LocalForward 127.0.0.1:15432 db.internal.example:5432
    LocalForward 127.0.0.1:18080 app.internal.example:8080
    LocalForward 127.0.0.1:19090 monitor.internal.example:9090
    ExitOnForwardFailure yes
```

## 검증 방법

터널이 안 될 때는 `SSH 접속`, `로컬 포트`, `SSH 서버에서 대상 서비스 접근`을 분리해서 확인해야 합니다.

### 1. SSH 자체가 되는지 확인

```bash
ssh -v user@bastion.example.com
```

SSH 연결 자체가 실패한다면 `-L`보다 먼저 인증, 방화벽, SSH 포트를 확인해야 합니다.

### 2. 내 PC에서 포트가 열렸는지 확인

Linux에서는 다음처럼 확인할 수 있습니다.

```bash
ss -lntp | grep 15432
```

예상 형태는 다음과 같습니다.

```text
127.0.0.1:15432 LISTEN
```

Windows PowerShell에서는 다음처럼 확인할 수 있습니다.

```powershell
netstat -ano | findstr :15432
```

### 3. 로컬 포트로 실제 요청 확인

웹서비스라면 다음처럼 확인합니다.

```bash
curl -v http://127.0.0.1:18080/
```

PostgreSQL이라면 다음처럼 접속합니다.

```bash
psql -h 127.0.0.1 -p 15432 -U appuser appdb
```

### 4. SSH 서버에서 최종 목적지 접근 확인

Bastion에 로그인한 뒤 대상 서비스까지 연결 가능한지 확인합니다.

```bash
nc -vz db.internal.example 5432
```

또는 웹서비스라면 다음처럼 확인할 수 있습니다.

```bash
curl -I http://app.internal.example:8080/
```

SSH 터널이 정상이어도 Bastion에서 대상 서버로 갈 수 없다면 연결은 실패합니다.

## 자주 발생하는 오류

### `bind: Address already in use`

내 PC의 `local_port`를 다른 프로세스가 이미 사용 중인 경우입니다.

해결 방법은 사용 중인 프로세스를 확인하거나 다른 로컬 포트를 선택하는 것입니다.

```bash
ssh -N \
  -L 127.0.0.1:25432:db.internal.example:5432 \
  user@bastion.example.com
```

목적지 포트가 `5432`라고 해서 로컬 포트도 반드시 `5432`일 필요는 없습니다.

### `channel ... open failed: administratively prohibited`

SSH 서버 정책에서 TCP forwarding이 제한된 경우에 발생할 수 있습니다.

서버 관리자는 `sshd_config`의 다음 항목을 확인해야 합니다.

```text
AllowTcpForwarding
PermitOpen
DisableForwarding
```

또한 `authorized_keys`의 키별 옵션으로 포워딩 목적지가 제한되어 있을 수도 있습니다.

정책상 금지된 환경에서는 설정을 우회하지 말고 서버 접근 정책에 맞게 허용 범위를 조정해야 합니다.

### 터널은 열렸는데 `Connection refused`

이 경우 SSH 자체보다는 최종 목적지 서비스를 확인해야 합니다.

예를 들어 다음을 점검합니다.

```text
1. 대상 프로세스가 실행 중인가?
2. destination_port가 맞는가?
3. 대상 서비스가 어떤 주소에 listen 중인가?
4. Bastion에서 destination_host:destination_port에 접근 가능한가?
5. 내부 방화벽 또는 보안 그룹이 차단하고 있지 않은가?
```

### 연결이 자주 끊기는 경우

필요하면 SSH keepalive를 설정할 수 있습니다.

```sshconfig
ServerAliveInterval 30
ServerAliveCountMax 3
```

다만 네트워크 장애 자체를 해결하는 옵션은 아니므로, 반복적으로 끊긴다면 VPN, NAT timeout, 방화벽, SSH 서버 로그를 함께 확인해야 합니다.

## 보안상 주의점

### 1. 가능하면 `127.0.0.1`에만 바인딩

다음처럼 명시하는 것을 권장합니다.

```bash
-L 127.0.0.1:15432:db.internal.example:5432
```

반대로 다음처럼 모든 인터페이스에 포트를 열면 다른 장비에서 내 PC의 포워딩 포트에 접근할 수 있는 상태가 될 수 있습니다.

```bash
-L 0.0.0.0:15432:db.internal.example:5432
```

`-g`도 로컬 포워딩 포트를 다른 호스트에 개방하는 방향의 옵션이므로 필요성을 명확히 확인하고 사용해야 합니다.

### 2. SSH 암호화 구간을 정확히 이해

`ssh -L`이 암호화하는 핵심 구간은 다음입니다.

```text
내 PC
  == SSH encrypted ==>
SSH Server
  -- destination protocol -->
Destination Service
```

즉 SSH 서버 이후의 두 번째 구간까지 SSH가 자동으로 암호화해 주는 것은 아닙니다.

예를 들어 Bastion에서 PostgreSQL 서버로 가는 구간의 암호화 여부는 PostgreSQL의 TLS 설정 등 최종 애플리케이션 프로토콜에 따라 결정됩니다.

### 3. 내부망 접근 권한을 우회 수단으로 사용하지 않기

SSH 포트 포워딩은 Bastion이 가진 네트워크 접근성을 내 PC에서 사용할 수 있게 합니다.

따라서 편리하지만 잘못 사용하면 네트워크 분리 정책을 사실상 우회하는 통로가 될 수 있습니다. 회사나 고객사 환경에서는 승인된 대상과 포트에만 사용해야 합니다.

서버 측에서는 필요하면 `PermitOpen`이나 키별 `permitopen` 제한을 사용해 허용 목적지를 좁힐 수 있습니다.

## `-L`, `-R`, `-D` 차이

세 옵션은 방향이 다릅니다.

| 옵션 | 목적 |
|---|---|
| `-L` | 내 PC에서 원격/내부 서비스에 접속 |
| `-R` | 원격 서버에서 내 PC 또는 내 PC가 접근 가능한 서비스에 접속 |
| `-D` | 내 PC에 SOCKS 프록시를 열어 동적으로 목적지를 선택 |

일반적으로 **내 PC에서 내부 DB나 원격 웹서비스를 열고 싶다면 `-L`**부터 생각하면 됩니다.

## 실무용 기본 명령

특별한 이유가 없다면 다음 형태를 기본값으로 사용하면 이해하기 쉽습니다.

```bash
ssh -NT \
  -o ExitOnForwardFailure=yes \
  -L 127.0.0.1:LOCAL_PORT:DESTINATION_HOST:DESTINATION_PORT \
  user@SSH_SERVER
```

예를 들어 내부 PostgreSQL이라면 다음과 같습니다.

```bash
ssh -NT \
  -o ExitOnForwardFailure=yes \
  -L 127.0.0.1:15432:db.internal.example:5432 \
  user@bastion.example.com
```

기억해야 할 핵심은 한 줄입니다.

```text
내 PC의 LOCAL_PORT에 접속하면
SSH_SERVER를 경유해서
SSH_SERVER가 DESTINATION_HOST:DESTINATION_PORT로 연결한다.
```

이 구조만 정확히 이해하면 `ssh -L`의 대부분의 실무 상황을 해석할 수 있습니다.

## 공식 문서

- OpenSSH `ssh(1)`: <https://man.openbsd.org/ssh>
- OpenSSH `ssh_config(5)`: <https://man.openbsd.org/ssh_config>
- OpenSSH `sshd_config(5)`: <https://man.openbsd.org/sshd_config>
- Playwright Authentication: <https://playwright.dev/docs/auth>
- noVNC: <https://github.com/novnc/noVNC>