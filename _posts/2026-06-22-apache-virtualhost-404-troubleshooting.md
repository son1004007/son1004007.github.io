---
layout: post
title: "Apache VirtualHost 404 장애 원인 분석"
date: 2026-06-22
categories: [infrastructure]
tags: [Apache, HTTPD, VirtualHost, 404, Troubleshooting]
---

## 문제점

Apache HTTP Server를 사용해 웹 서비스를 운영할 때, 설정 파일은 정상으로 보이지만 도메인 접속 시 계속 `404 Not Found`가 발생하는 상황이 있습니다.

대표적인 증상은 다음과 같습니다.

- `systemctl status httpd`는 정상입니다.
- `apachectl configtest` 또는 `httpd -t` 결과가 `Syntax OK`입니다.
- 80 포트는 정상적으로 listen 중입니다.
- 하지만 브라우저 또는 `curl` 요청은 `404 Not Found`를 반환합니다.
- 에러 로그에는 명확한 치명 오류가 없고 access log에 404만 남습니다.

이 경우 단순히 Apache가 죽은 문제가 아니라, 요청이 의도한 VirtualHost 또는 DocumentRoot, ProxyPass로 연결되지 않는 문제일 가능성이 높습니다.

## 원인

Apache 404 장애는 보통 다음 중 하나로 발생합니다.

1. 요청 Host 헤더가 VirtualHost의 `ServerName` 또는 `ServerAlias`와 맞지 않음
2. 기본 VirtualHost가 요청을 받아 다른 DocumentRoot로 처리함
3. DocumentRoot 경로에 실제 파일이 없음
4. Directory 권한 또는 SELinux 컨텍스트 문제
5. ProxyPass 경로와 실제 백엔드 경로가 맞지 않음
6. 80/443 포트 처리 주체가 Apache가 아닌 다른 서비스임
7. HTTPS 요청인데 443 VirtualHost가 없거나 인증서 설정이 빠짐

특히 여러 웹서버를 교체하면서 운영한 환경에서는 Nginx, Apache, Tomcat, Java 애플리케이션의 역할이 뒤섞여 원인 파악이 어려워질 수 있습니다.

## 해결

대응 순서는 다음처럼 가져가는 것이 좋습니다.

```text
포트 확인
-> Host 헤더 기준 VirtualHost 매칭 확인
-> DocumentRoot 확인
-> ProxyPass 확인
-> 로그 확인
-> 80/443 분리 확인
```

가장 중요한 점은 브라우저로만 확인하지 않고 `curl -H 'Host: ...'` 방식으로 VirtualHost 매칭을 직접 검증하는 것입니다.

## 실행 방법

### 1. Apache 설정 문법 확인

```bash
sudo apachectl configtest
# 또는
sudo httpd -t
```

정상 예시는 다음입니다.

```text
Syntax OK
```

문법이 정상이어도 VirtualHost 매칭 오류는 발생할 수 있습니다.

### 2. 포트 listen 확인

```bash
sudo ss -lntp | grep -E ':80|:443'
```

확인할 항목은 다음입니다.

- 80 포트를 Apache가 listen 중인지
- 443 포트를 Apache가 listen 중인지
- Nginx 등 다른 웹서버가 같은 포트를 사용 중인지
- 백엔드 애플리케이션 포트가 정상인지

### 3. VirtualHost 목록 확인

```bash
sudo apachectl -S
```

이 명령은 어떤 VirtualHost가 어떤 `ServerName`으로 등록되어 있는지 보여줍니다.

확인할 항목은 다음입니다.

- 요청 도메인과 `ServerName`이 일치하는지
- `ServerAlias`에 필요한 도메인이 포함되어 있는지
- 기본 VirtualHost가 의도한 설정인지
- 80과 443 각각에 VirtualHost가 있는지

### 4. Host 헤더로 직접 요청 테스트

```bash
curl -I -H 'Host: example.com' http://127.0.0.1/
curl -I -H 'Host: www.example.com' http://127.0.0.1/
```

외부 도메인 DNS나 방화벽 문제와 Apache 설정 문제를 분리하기 위해 로컬 요청부터 확인합니다.

### 5. DocumentRoot 확인

설정 파일에서 DocumentRoot를 확인합니다.

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example
</VirtualHost>
```

실제 경로가 있는지 확인합니다.

```bash
ls -la /var/www/example
```

정적 파일을 제공하는 구조라면 최소한 `index.html` 또는 라우팅을 처리할 파일이 있어야 합니다.

### 6. Directory 권한 확인

```bash
namei -l /var/www/example
ls -ld /var/www/example
```

Apache 실행 계정이 경로를 읽을 수 있어야 합니다.

SELinux가 활성화되어 있다면 컨텍스트도 확인합니다.

```bash
getenforce
ls -Z /var/www/example
```

필요 시 웹 루트 컨텍스트를 적용합니다.

```bash
sudo restorecon -Rv /var/www/example
```

### 7. ProxyPass 확인

백엔드 애플리케이션으로 넘기는 구조라면 ProxyPass 경로를 확인합니다.

```apache
ProxyPass /app http://127.0.0.1:8080/app
ProxyPassReverse /app http://127.0.0.1:8080/app
```

백엔드가 실제로 응답하는지 확인합니다.

```bash
curl -I http://127.0.0.1:8080/app
```

Apache 404인지, 백엔드 애플리케이션의 404인지도 구분해야 합니다.

### 8. 로그 확인

```bash
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log
```

VirtualHost별 로그 파일을 따로 지정했다면 해당 로그를 확인합니다.

```apache
ErrorLog logs/example-error.log
CustomLog logs/example-access.log combined
```

## 검증 방법

정상화 여부는 다음 기준으로 확인합니다.

```bash
curl -I -H 'Host: example.com' http://127.0.0.1/
curl -I http://example.com/
```

확인할 항목은 다음입니다.

- HTTP status가 200, 301, 302 등 의도한 값인지
- access log에 올바른 VirtualHost 로그가 남는지
- error log에 권한 또는 proxy 오류가 없는지
- 80/443 모두 의도한 서버가 응답하는지

## 재발 방지 / 개선 방향

1. VirtualHost는 도메인별로 파일을 분리합니다.
2. `apachectl -S` 결과를 운영 문서에 남깁니다.
3. 80과 443 설정을 같이 관리합니다.
4. Nginx와 Apache를 동시에 운영할 경우 포트 소유자를 명확히 합니다.
5. 정적 파일 제공과 백엔드 프록시 역할을 구분합니다.
6. 장애 대응 시 `curl -H Host` 테스트를 표준 절차에 포함합니다.

## 포트폴리오 관점의 의미

이 사례는 웹서버 장애를 단순 재시작으로 처리하지 않고, 네트워크 포트, VirtualHost 매칭, DocumentRoot, ProxyPass, 백엔드 응답을 분리해서 점검하는 역량을 보여줍니다.

특히 다음 역량과 연결됩니다.

- Apache HTTPD 운영 경험
- 웹서버와 WAS 역할 분리 이해
- 404 오류의 발생 계층 구분
- 운영 장애 상황에서의 점검 순서 수립
- Nginx/Apache/Tomcat 기반 서비스 운영 역량
