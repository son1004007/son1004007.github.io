---
layout: post
title: "Nginx + Tomcat + Let’s Encrypt 기반 웹 서비스 운영 구조"
date: 2026-06-22
categories: [infrastructure]
tags: [Nginx, Tomcat, SSL, Let’s Encrypt, Reverse Proxy, Operations]
---

## 문제점

웹 서비스를 운영하다 보면 Apache, Nginx, Tomcat, Java 애플리케이션, SSL 인증서 설정이 섞이면서 구조가 복잡해지는 경우가 많습니다.

특히 기존 서버에 여러 설정이 누적되어 있으면 다음 문제가 발생합니다.

- 어떤 프로세스가 80/443 포트를 점유하는지 불명확합니다.
- 정적 파일 제공과 WAS 프록시 역할이 섞입니다.
- Apache 설정과 Nginx 설정이 동시에 남아 있습니다.
- Tomcat은 정상인데 외부 도메인에서는 404, 502, 504가 발생합니다.
- SSL 인증서 갱신 경로와 웹서버 reload 방식이 정리되어 있지 않습니다.
- 장애 발생 시 어느 계층부터 확인해야 하는지 모호합니다.

이런 상태에서는 설정을 조금씩 수정할수록 장애 원인이 더 불분명해집니다. 따라서 운영 구조를 단순화하고 역할을 명확히 나누는 것이 우선입니다.

## 원인

구조가 복잡해지는 원인은 대개 다음과 같습니다.

1. 초기에는 단순 정적 웹으로 시작했지만 이후 Java WAS가 추가됩니다.
2. Apache와 Nginx를 번갈아 사용하면서 설정 파일이 누적됩니다.
3. SSL 적용 시점에 임시 설정이 운영 설정으로 남습니다.
4. 80, 443, 8080, 8009 등 포트 역할이 문서화되지 않습니다.
5. 웹서버와 WAS의 책임 경계가 정리되지 않습니다.
6. 인증서 갱신 자동화와 서비스 reload 절차가 빠집니다.

문제의 핵심은 특정 도구가 아니라 **역할 분리 부재**입니다.

## 해결

권장 구조는 단순합니다.

```text
Client
  -> HTTPS 443
  -> Nginx
  -> Tomcat 8080
  -> Java Web Application
```

각 구성요소의 역할은 다음처럼 분리합니다.

| 구성요소 | 역할 |
|---|---|
| Nginx | 80/443 수신, SSL 종료, 정적 파일 제공, Reverse Proxy |
| Let’s Encrypt | TLS 인증서 발급 및 갱신 |
| Tomcat | Java 애플리케이션 실행 |
| Spring Boot / WAR | 비즈니스 로직과 화면/API 처리 |
| systemd | Nginx, Tomcat 프로세스 관리 |

Apache를 꼭 써야 할 이유가 없다면, Nginx를 외부 진입점으로 고정하고 Apache 설정은 제거하거나 비활성화하는 것이 좋습니다.

## 실행 방법

### 1. 포트 점유 상태 확인

먼저 어떤 프로세스가 외부 포트를 사용 중인지 확인합니다.

```bash
sudo ss -lntp | grep -E ':80|:443|:8080|:8009'
```

확인 기준은 다음입니다.

```text
80   -> Nginx
443  -> Nginx
8080 -> Tomcat
8009 -> 필요하지 않으면 비활성화 검토
```

Nginx와 Apache가 동시에 80/443을 사용하려고 하면 구조가 불안정해집니다. 하나만 외부 진입점으로 선택해야 합니다.

### 2. 서비스 상태 확인

```bash
systemctl status nginx
systemctl status tomcat
```

배포 방식에 따라 Tomcat 서비스 이름은 다를 수 있습니다.

```bash
systemctl list-units --type=service | grep -Ei 'tomcat|java|was'
```

### 3. Tomcat 직접 응답 확인

Nginx 설정 전에 Tomcat이 로컬에서 정상 응답하는지 확인합니다.

```bash
curl -I http://127.0.0.1:8080/
```

애플리케이션 context path가 있다면 해당 경로로 확인합니다.

```bash
curl -I http://127.0.0.1:8080/app/
```

Tomcat이 직접 응답하지 않으면 Nginx 설정을 고쳐도 외부 서비스는 정상화되지 않습니다.

### 4. Nginx Reverse Proxy 설정

기본 구조는 다음과 같습니다.

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/letsencrypt;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

운영 환경에서는 도메인, context path, 정적 파일 경로, 업로드 용량 제한 등을 서비스에 맞게 조정해야 합니다.

### 5. Nginx 설정 검증

```bash
sudo nginx -t
sudo systemctl reload nginx
```

설정 변경 후 바로 재시작하기보다 `nginx -t`로 문법을 먼저 검증합니다.

### 6. Let’s Encrypt 인증서 발급

Certbot을 사용하는 경우 일반적인 흐름은 다음과 같습니다.

```bash
sudo certbot certonly --webroot \
  -w /var/www/letsencrypt \
  -d example.com \
  -d www.example.com
```

Nginx 플러그인을 사용할 수도 있습니다.

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

다만 운영 서버에서는 자동으로 Nginx 설정을 수정하는 방식보다, 직접 관리하는 설정 파일을 기준으로 인증서만 발급받는 방식이 더 예측 가능할 때가 많습니다.

### 7. 인증서 자동 갱신 확인

```bash
systemctl list-timers | grep certbot
sudo certbot renew --dry-run
```

갱신 후 Nginx reload가 필요한지 확인합니다.

```bash
sudo systemctl reload nginx
```

### 8. 외부 접속 확인

```bash
curl -I http://example.com
curl -I https://example.com
```

기대 결과는 다음입니다.

```text
http  -> 301 또는 308 redirect to https
https -> 200, 302, 또는 애플리케이션이 의도한 응답
```

## 검증 방법

운영 구조가 정상인지 확인하려면 계층별로 점검합니다.

### 프로세스

```bash
systemctl is-active nginx
systemctl is-active tomcat
```

### 포트

```bash
sudo ss -lntp | grep -E ':80|:443|:8080'
```

### 로컬 WAS

```bash
curl -I http://127.0.0.1:8080/
```

### 외부 HTTP/HTTPS

```bash
curl -I http://example.com
curl -I https://example.com
```

### 인증서

```bash
sudo certbot certificates
```

### 로그

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
sudo journalctl -u tomcat -f
```

장애가 발생하면 외부 URL부터 보는 것이 아니라, 포트 → Nginx → Tomcat → 애플리케이션 순서로 좁히는 것이 좋습니다.

## 재발 방지 / 개선 방향

운영 구조를 안정화하려면 다음 기준을 적용합니다.

1. 외부 진입점은 Nginx 하나로 고정합니다.
2. Apache를 사용하지 않으면 stop/disable 처리하고 설정 파일도 별도로 백업합니다.
3. 80/443/8080 포트 역할을 문서화합니다.
4. Nginx 설정 파일은 도메인 단위로 분리합니다.
5. SSL 인증서 발급/갱신/검증 절차를 문서화합니다.
6. Tomcat 직접 응답 확인 절차를 장애 대응 문서에 포함합니다.
7. 배포 후에는 `nginx -t`, `curl`, 로그 확인을 표준 검증 절차로 둡니다.

예시 운영 체크리스트는 다음입니다.

```text
[ ] nginx -t 성공
[ ] nginx active
[ ] tomcat active
[ ] 80 -> nginx
[ ] 443 -> nginx
[ ] 8080 -> tomcat
[ ] http -> https redirect
[ ] https 정상 응답
[ ] certbot renew --dry-run 성공
```

## 포트폴리오 관점의 의미

이 글은 단순히 Nginx 설정 예시를 정리한 것이 아니라, 운영 서버에서 웹서버, WAS, SSL 인증서의 책임을 분리하고 장애 대응 가능한 구조로 정리하는 역량을 보여줍니다.

특히 다음 역량과 연결됩니다.

- Nginx Reverse Proxy 운영 경험
- Tomcat 기반 Java 웹 서비스 운영 이해
- Let’s Encrypt 인증서 적용 및 갱신 구조 이해
- 80/443/8080 포트 역할 분리
- Apache/Nginx/Tomcat이 섞인 환경을 단순화하는 판단력
- 장애 발생 시 계층별로 원인을 좁히는 운영 역량
