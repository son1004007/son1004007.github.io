---
layout: post
title: "CSRF·XSS·CORS·SSRF 차이: 웹 보안 면접 암기 노트와 흐름도"
description: "CSRF, XSS, CORS, SSRF의 동작 주체, 발생 과정, 방어 방법을 주어가 분명한 설명과 면접 답변, 흐름도 4개로 정리합니다."
date: 2026-10-09
permalink: /web-security-csrf-xss-cors-ssrf/
categories: [backend]
tags: [Spring Security, Web Security, CSRF, XSS, CORS, SSRF, Interview]
---

웹 보안 면접에서 네 용어를 구분하는 핵심은 **누가 무엇을 실행하는지**다.

- **CSRF:** 공격자는 사용자의 로그인 상태를 악용해 서버에 요청을 전송하도록 유도한다.
- **XSS:** 공격자는 웹페이지가 사용자 입력을 스크립트로 실행하도록 만든다.
- **CORS:** 브라우저는 출처가 다른 서버의 응답을 웹페이지의 JavaScript가 읽을 수 있는지 판단한다.
- **SSRF:** 공격자는 백엔드 서버가 공격자가 선택한 주소로 네트워크 요청을 보내도록 유도한다.

## 1. 30초 암기표

| 용어 | 영문 이름 | 동작 주체 | 핵심 동작 |
| --- | --- | --- | --- |
| **CSRF** | Cross-Site Request Forgery | 피해자의 브라우저 | 브라우저가 인증정보와 함께 위조 요청을 전송 |
| **XSS** | Cross-Site Scripting | 피해자의 브라우저 | 브라우저가 입력값을 스크립트로 실행 |
| **CORS** | Cross-Origin Resource Sharing | 브라우저 | 브라우저가 교차 출처 응답의 JavaScript 접근을 제어 |
| **SSRF** | Server-Side Request Forgery | 백엔드 서버 | 서버가 공격자가 지정한 주소에 요청을 전송 |

**암기 키워드:** CSRF = **요청 위조**, XSS = **스크립트 실행**, CORS = **응답 읽기 정책**, SSRF = **서버 요청 위조**.

## 2. CSRF: 로그인한 사용자의 요청을 위조하는 공격

**정의:** 공격자는 사용자의 브라우저에 조작된 요청을 발생시켜 정상 서비스의 상태 변경 기능을 실행하도록 유도한다.

**발생 과정**

1. 사용자는 정상 서비스에 로그인하고 세션 쿠키를 보유한다.
2. 공격자는 사용자가 악성 웹페이지를 열도록 유도한다.
3. 브라우저는 쿠키 정책이 허용하는 상황에서 로그인 쿠키를 포함한 요청을 정상 서비스로 보낸다.
4. 서버가 요청의 출처와 CSRF 토큰을 검증하면 서버는 공격자의 위조 요청을 차단한다.

**Spring 실무 방어**

- **Spring Security의 CSRF 보호 기능**은 세션 기반 상태 변경 요청의 CSRF 토큰을 검증한다.
- **서버**는 필요에 따라 `Origin`, `Referer`, Fetch Metadata 헤더도 검사한다.
- **브라우저**는 `SameSite` 쿠키 속성에 따라 교차 사이트 요청의 쿠키 전송을 제한한다.
- **개발자**는 상태 변경 기능을 POST·PUT·PATCH·DELETE 등 용도에 맞는 HTTP 메서드로 구현한다.
- **설계자**는 쿠키에 담긴 JWT처럼 브라우저가 인증정보를 자동 전송하는 방식에도 CSRF 방어를 적용한다.

**면접 답변:** "CSRF는 공격자가 로그인한 사용자의 브라우저를 이용해 서버에 위조 요청을 보내는 공격입니다. Spring Security는 CSRF 토큰을 검증해 상태 변경 요청을 보호합니다."

## 3. XSS: 웹페이지가 악성 스크립트를 실행하는 취약점

**정의:** 웹 애플리케이션이 외부 입력을 실행 가능한 HTML·JavaScript로 해석할 때 브라우저에서 악성 스크립트가 실행된다.

**발생 과정**

1. 공격자는 게시글이나 검색 파라미터 등에 악성 스크립트로 해석될 수 있는 문자열을 전달한다.
2. 웹 애플리케이션은 해당 문자열을 HTML 또는 DOM에 삽입한다.
3. 브라우저는 해당 값을 실행 가능한 코드로 해석한다.
4. 브라우저는 피해자의 로그인 상태에서 스크립트가 요청한 동작을 실행할 수 있다.

**XSS 유형**

| 유형 | 동작 |
| --- | --- |
| Stored XSS | 서버가 저장한 입력값이 다른 사용자의 화면에 출력될 때 실행 |
| Reflected XSS | 요청에 포함된 입력값이 응답에 반영될 때 실행 |
| DOM-based XSS | 브라우저의 JavaScript가 외부 입력을 DOM에 삽입할 때 실행 |

**Spring 및 프런트엔드 실무 방어**

- **템플릿 엔진과 프런트엔드 코드**는 HTML 본문·속성·URL 등 출력 맥락에 맞는 인코딩을 적용한다.
- **JavaScript 코드**는 일반 텍스트를 출력할 때 `textContent`를 사용한다.
- **HTML 편집 기능**은 검증된 HTML sanitizer로 허용 태그와 속성을 정리한다.
- **웹 서버**는 CSP(Content Security Policy)를 추가 방어 수단으로 제공한다.

```javascript
// 브라우저는 userInput을 일반 텍스트로 출력한다.
messageElement.textContent = userInput;
```

**면접 답변:** "XSS는 사용자 입력이 브라우저에서 악성 스크립트로 실행되는 취약점입니다. 웹 애플리케이션은 출력 위치에 맞게 데이터를 인코딩하고 안전한 DOM API를 사용해 방어합니다."

## 4. CORS: 브라우저가 교차 출처 응답 접근을 제어하는 정책

**정의:** 브라우저는 CORS 정책과 서버의 응답 헤더를 사용해 출처가 다른 웹페이지의 JavaScript에 API 응답 접근 권한을 부여한다.

**Origin 구성:** 프로토콜(scheme) + 호스트(host) + 포트(port).

**발생 과정**

1. `https://front.example`에서 실행 중인 JavaScript는 `https://api.example`에 요청한다.
2. 브라우저는 요청 메서드와 헤더 등의 조건에 따라 사전 요청(Preflight, `OPTIONS`)을 전송한다.
3. API 서버는 `Access-Control-Allow-Origin` 등 필요한 CORS 헤더를 응답한다.
4. 브라우저는 CORS 헤더를 검사한 뒤 JavaScript의 응답 접근을 허용하거나 제한한다.

**Spring 실무 설정**

- **개발자**는 실제 프런트엔드의 Origin·Method·Header를 허용목록에 등록한다.
- **Spring Security**는 `CorsConfigurationSource` 또는 Spring MVC의 CORS 설정을 통합해 사전 요청을 처리한다.
- **브라우저**는 자격 증명을 포함한 교차 출처 요청에 대해 서버가 명시적으로 허용한 Origin을 기준으로 응답 접근을 판단한다.
- **백엔드 서버**는 CORS 설정과 독립적으로 사용자 인증과 API 인가를 검사한다.

**요청 구분:** 브라우저는 사전 요청이 필요한 경우 OPTIONS로 먼저 허용 범위를 확인한다. 브라우저는 단순 교차 출처 요청을 서버에 전송한 뒤 응답의 CORS 헤더를 검사할 수도 있다.

**면접 답변:** "CORS는 브라우저가 서로 다른 출처의 API 응답에 대한 JavaScript 접근을 제어하는 정책입니다. API 서버는 허용 출처와 메서드 등의 정보를 응답 헤더로 전달합니다."

## 5. SSRF: 백엔드 서버의 네트워크 요청을 악용하는 공격

**정의:** 공격자는 URL 입력 기능을 이용해 백엔드 서버가 공격자가 지정한 네트워크 대상에 요청하도록 유도한다.

**발생 과정**

1. 사용자는 이미지 미리보기, 파일 다운로드 또는 Webhook 등의 기능에 URL을 제출한다.
2. 백엔드 서버는 입력받은 URL을 처리하며 네트워크 요청을 생성한다.
3. 공격자가 내부 서비스 주소를 입력한 경우 서버는 자신의 네트워크 접근 범위 안에서 해당 주소에 접근할 수 있다.
4. 서버가 목적지 검증과 송신 네트워크 정책을 적용하면 허용된 대상에 대해서만 요청을 전송한다.

**Spring 실무 방어**

- **서버**는 외부 요청 대상을 사전에 정한 목적지 허용목록(allowlist)으로 제한한다.
- **서버**는 URL의 scheme, 호스트, DNS 해석 결과, 실제 연결 IP를 검사한다.
- **HTTP 클라이언트**는 리다이렉트를 처리할 때 변경된 목적지를 다시 검증한다.
- **인프라**는 내부망·클라우드 메타데이터 서비스 접근을 송신 네트워크 정책으로 제한한다.
- **개발자**는 `RestClient`, `WebClient`, `RestTemplate` 등이 외부 URL을 호출하는 경로를 점검한다.

**면접 답변:** "SSRF는 공격자가 서버의 외부 요청 기능을 악용해 서버가 지정된 주소로 네트워크 요청을 보내게 만드는 공격입니다. 서버는 목적지 허용목록, DNS·IP 검증, 리다이렉트 검증과 네트워크 정책으로 방어합니다."

## 6. 면접 질문 8개와 모범 답변

| 질문 | 바로 말할 답변 |
| --- | --- |
| **CSRF와 XSS의 차이는 무엇인가요?** | **CSRF 공격자**는 사용자 브라우저가 인증된 요청을 보내도록 유도합니다. **XSS 공격자**는 사용자 브라우저가 악성 스크립트를 실행하도록 만듭니다. |
| **CORS와 CSRF의 역할은 무엇인가요?** | **CORS 정책**은 교차 출처 응답 읽기를 제어합니다. **CSRF 보호 기능**은 로그인 상태를 이용한 위조 요청을 검사합니다. |
| **XSS 환경에서 CSRF 방어는 어떻게 영향을 받나요?** | **XSS로 실행된 동일 출처 스크립트**는 CSRF 토큰에 접근하거나 인증된 요청을 보낼 수 있습니다. **웹 애플리케이션**은 XSS와 CSRF 방어를 함께 적용합니다. |
| **JWT 사용 시 CSRF 대응 기준은 무엇인가요?** | **개발자**는 브라우저가 JWT를 자동 전송하는지 확인합니다. **쿠키 기반 JWT 인증 서비스**는 CSRF 보호를 적용합니다. |
| **CORS 요청과 Preflight는 어떻게 동작하나요?** | **브라우저**는 필요한 경우 OPTIONS 사전 요청을 보내고, **서버**는 허용 정보를 응답합니다. **브라우저**는 응답 헤더를 검사해 JavaScript 접근을 제어합니다. |
| **SSRF와 CSRF의 요청 주체는 누구인가요?** | **CSRF**에서는 사용자 브라우저가 요청을 전송합니다. **SSRF**에서는 백엔드 서버가 요청을 전송합니다. |
| **XSS의 대표 방어 방법은 무엇인가요?** | **애플리케이션**은 출력 맥락에 맞는 인코딩, 안전한 DOM API, HTML sanitizer 및 CSP를 사용합니다. |
| **Spring 백엔드에서 SSRF는 어떻게 방어하나요?** | **백엔드 서버**는 목적지 허용목록과 DNS·IP·리다이렉트 검증을 수행하고, **인프라**는 송신 네트워크 정책을 적용합니다. |

## 7. 실무 검증 체크리스트

- [ ] **CSRF:** 테스트는 로그인 쿠키가 있는 요청의 CSRF 토큰 누락·불일치 차단 결과를 검증한다.
- [ ] **XSS:** 테스트는 외부 입력값이 화면에 안전한 텍스트로 출력되는지 확인한다.
- [ ] **CORS:** 브라우저 테스트는 허용한 Origin의 응답 읽기와 제한한 Origin의 응답 접근 차단을 확인한다.
- [ ] **SSRF:** 테스트는 허용목록에 등록한 목적지의 호출 성공과 내부·제한 주소에 대한 차단을 검증한다.
- [ ] **공통:** 개발자는 브라우저 정책, 서버 인증·인가, 송신 네트워크 통제를 각각 검증한다.

## 8. 네 가지 흐름을 그림으로 이해하기

아래 그림은 요청을 시작하는 주체, 요청을 처리하는 위치, 개발자가 적용할 방어 기술을 구분한다.

### 8-1. CSRF: 공격자 사이트 → 브라우저 → 정상 서비스

![CSRF 공격자는 웹페이지를 통해 사용자의 브라우저가 인증 쿠키를 포함한 요청을 전송하도록 유도한다. 정상 서비스는 CSRF 토큰을 검증한다.]({{ '/assets/diagrams/web-security-csrf-flow.svg' | relative_url }})

**암기:** **사용자 브라우저**가 로그인 쿠키와 함께 위조 요청을 전송한다.

### 8-2. XSS: 악성 입력 → 취약한 화면 → 브라우저

![XSS 공격자는 외부 입력을 전달하고 취약한 웹페이지는 문자열을 실행 가능한 코드로 해석한다. 방문자의 브라우저가 악성 스크립트를 실행한다.]({{ '/assets/diagrams/web-security-xss-flow.svg' | relative_url }})

**암기:** **사용자 브라우저**가 데이터로 받은 문자열을 실행 가능한 코드로 해석한다.

### 8-3. CORS: 웹페이지 → API 서버 → 브라우저의 응답 접근 판단

![CORS 정책에 따라 웹페이지 JavaScript가 다른 출처의 API에 요청하고 API 서버가 허용 응답 헤더를 보낸다. 브라우저가 JavaScript의 응답 접근 여부를 판단한다.]({{ '/assets/diagrams/web-security-cors-flow.svg' | relative_url }})

**암기:** **브라우저**가 응답 헤더를 확인해 JavaScript의 응답 접근을 판단한다.

### 8-4. SSRF: 입력 URL → 백엔드 서버 → 네트워크 대상

![SSRF 공격자는 URL을 입력하고 백엔드 서버가 지정된 주소로 네트워크 요청을 보낸다. 서버는 목적지와 송신 경로를 검증한다.]({{ '/assets/diagrams/web-security-ssrf-flow.svg' | relative_url }})

**암기:** **백엔드 서버**가 공격자가 지정한 주소로 네트워크 요청을 전송한다.

## 9. 마지막 10초 요약

```text
CSRF: 브라우저가 인증된 위조 요청 전송 → CSRF Token / Origin
XSS : 브라우저가 입력 문자열을 코드로 실행 → Output Encoding / Safe DOM
CORS: 브라우저가 교차 출처 응답 읽기 제어 → Allow-Origin / Method
SSRF: 백엔드 서버가 지정된 주소로 요청 전송 → Allowlist / DNS·IP / Egress
```

함께 읽기: [Spring Security 인증·인가와 면접 핵심 질문](https://github.com/son1004007/son1004007.github.io/blob/main/_posts/2026-10-09-spring-security-authentication-authorization-interview-guide.md)

## 공식 참고 문서

- [OWASP: CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP: XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN: Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [Spring Security: CORS](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)
- [OWASP: SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
