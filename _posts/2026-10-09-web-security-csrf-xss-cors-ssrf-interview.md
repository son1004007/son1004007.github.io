---
layout: post
title: "CSRF·XSS·CORS·SSRF 차이: 웹 보안 면접 암기 노트와 흐름도"
description: "CSRF, XSS, CORS, SSRF의 뜻과 공격 또는 동작 위치, 방어 기준을 비교하고 Spring 백엔드 면접 답변과 흐름도 4개로 정리합니다."
date: 2026-10-09
permalink: /web-security-csrf-xss-cors-ssrf/
categories: [backend]
tags: [Spring Security, Web Security, CSRF, XSS, CORS, SSRF, Interview]
---

**한 문장으로 구분:** CSRF는 **요청 위조**, XSS는 **브라우저에서 악성 스크립트 실행**, SSRF는 **서버의 요청 기능 악용**이다. CORS는 이들과 달리 공격명이 아니라 **브라우저의 다른 출처 간 응답 접근 정책**이다.

이 글은 웹 보안을 면접에서 짧고 정확하게 설명하기 위한 학습 노트다. 맨 아래에 각 개념의 실제 흐름을 **그림 4개**로 정리했다.

## 1. 30초 암기표

| 약어 | 풀어 쓰면 | 한 줄 암기 | 주된 실행·판단 위치 |
| --- | --- | --- | --- |
| **CSRF** | Cross-Site Request Forgery | **로그인한 사용자의 요청을 위조** | 사용자 브라우저 → 서버 |
| **XSS** | Cross-Site Scripting | **외부 입력이 스크립트로 실행** | 사용자 브라우저 |
| **CORS** | Cross-Origin Resource Sharing | **다른 출처의 응답을 읽어도 되는지 판단** | 브라우저의 보안 정책 |
| **SSRF** | Server-Side Request Forgery | **서버가 원치 않는 주소로 요청하도록 유도** | 백엔드 서버 |

암기 문장: **CSRF = 요청 / XSS = 스크립트 / CORS = 출처 / SSRF = 서버 요청**.

> **주의:** CORS는 취약점 자체가 아니다. 잘못된 CORS 설정이 정보 노출의 원인이 될 수는 있지만, CORS는 인증·인가나 CSRF 방어를 대체하지 않는다.

## 2. CSRF: 로그인 상태로 요청을 위조

- **뜻:** 공격자가 사용자의 브라우저를 유도해, 사용자가 의도하지 않은 상태 변경 요청을 정상 서비스에 보내게 하는 공격.
- **예시:** 로그인된 사용자에게 악의적인 페이지를 열게 하여, 조건이 맞는 경우 인증 쿠키가 첨부된 계정 정보 변경 요청이 발생하도록 유도한다.
- **성립 조건:** 브라우저가 자동 전송하는 인증 정보(대표적으로 쿠키), 방어가 부족한 상태 변경 엔드포인트 등이 함께 있어야 한다.
- **방어:** CSRF Token 검증, Origin/Referer 검증, Fetch Metadata 검토, SameSite 쿠키 보조 방어. 상태 변경을 GET으로 구현하지 않는다.
- **Spring 기준:** 세션 쿠키를 쓰는 MVC·폼 로그인에서는 Spring Security의 CSRF 방어를 기본적으로 유지하고, POST/PUT/PATCH/DELETE 등 상태 변경 요청을 검증한다.

**면접 한 줄:** "CSRF는 사용자의 로그인 상태를 악용해 의도하지 않은 요청을 보내는 공격으로, 세션 기반 서비스에서 CSRF 토큰과 출처 검증으로 방어합니다."

**주의할 오해:** API나 JWT를 사용한다는 이유만으로 CSRF가 사라지는 것은 아니다. JWT를 쿠키에 넣어 브라우저가 자동 전송하는 구조라면 위험을 다시 평가해야 한다. `SameSite` 하나만으로 모든 배포 환경의 CSRF 방어가 해결되는 것도 아니다.

## 3. XSS: 입력된 문자열이 브라우저에서 코드로 실행

- **뜻:** 신뢰할 수 없는 입력이 화면에 안전하지 않게 삽입되어 브라우저에서 악성 스크립트로 실행되는 취약점.
- **예시:** 게시글·검색 결과·프로필 입력이 HTML이나 DOM으로 해석되어 사용자 화면에서 원치 않는 동작이 실행된다.
- **유형:** Stored(저장형), Reflected(반사형), DOM-based(DOM 기반).
- **방어:** 데이터가 쓰이는 맥락(HTML 본문·속성·URL 등)에 맞는 **출력 인코딩**, 안전한 템플릿 엔진, `textContent` 등 안전한 DOM API 사용. HTML 입력을 허용해야 한다면 검증된 sanitizer 사용. CSP는 보조 방어 수단.
- **Spring 기준:** JSP, Thymeleaf 또는 JavaScript로 화면에 값을 출력할 때 **자동 이스케이프 적용 여부와 위험한 직접 삽입 위치**를 확인한다.

```javascript
// 일반 텍스트를 화면에 보여줄 때 권장
messageElement.textContent = userInput;

// 검증되지 않은 입력을 HTML로 해석하므로 위험
// messageElement.innerHTML = userInput;
```

**면접 한 줄:** "XSS는 사용자 입력이 브라우저에서 스크립트로 실행되는 문제이며, 입력 자체를 신뢰하지 않고 출력 위치에 맞게 인코딩하거나 안전한 DOM API를 사용해 방어합니다."

**주의할 오해:** 모든 입력에 같은 방식으로 HTML 이스케이프를 적용하면 충분하다는 설명은 부정확하다. HTML, HTML 속성, JavaScript, URL은 안전한 처리 맥락이 다르다.

## 4. CORS: 다른 출처에서 API 응답을 읽도록 허용하는 정책

- **뜻:** 웹페이지의 출처(Origin)와 API 서버의 출처가 다를 때, 브라우저가 JavaScript에 응답 접근을 허용할지 결정하는 HTTP 헤더 기반 메커니즘.
- **Origin:** **프로토콜(scheme) + 호스트 + 포트**의 조합. 포트만 달라도 출처가 다를 수 있다.
- **예시:** `https://front.example`의 화면에서 `https://api.example` API를 호출한다.
- **Preflight:** 일부 교차 출처 요청에서 브라우저가 먼저 `OPTIONS`를 보내 허용 메서드·헤더 등을 확인한다. **모든 CORS 요청에서 발생하는 것은 아니다.**
- **방어/설정:** 필요한 Origin·Method·Header만 허용한다. 자격 증명이 포함된 교차 출처 요청에서는 `Access-Control-Allow-Origin: *`를 사용할 수 없다.
- **Spring 기준:** `CorsConfigurationSource` 또는 Spring MVC CORS 설정과 `http.cors(...)`를 일관되게 구성한다. 사전 요청은 인증 쿠키가 없으므로 인증 필터가 거부하지 않도록 CORS 처리를 우선해야 한다.

**면접 한 줄:** "CORS는 다른 출처의 API 응답을 브라우저 JavaScript가 읽을 수 있는지를 제어하는 정책입니다. 서버가 허용 출처와 메서드를 응답 헤더로 알려주며, 인증과 인가를 대신하지 않습니다."

**주의할 오해:** "CORS에 걸리면 서버 요청 자체가 무조건 차단된다"는 설명은 틀리다. 사전 요청이 실패하면 후속 요청이 막힐 수 있지만, 사전 요청이 필요 없는 교차 출처 요청은 서버에 도달한 뒤 **브라우저가 응답 읽기를 차단**할 수 있다. 서버-서버 통신도 CORS로 보호되는 것은 아니다.

## 5. SSRF: 백엔드 서버가 임의 주소에 접근하도록 유도

- **뜻:** 사용자가 입력한 URL이나 네트워크 대상을 서버가 충분히 검증하지 않아, 공격자가 서버 권한으로 내부 또는 외부 대상에 요청을 보내도록 하는 공격.
- **예시:** 파일 다운로드·미리보기·Webhook 기능에서 받은 URL을 서버가 그대로 호출해 내부 서비스까지 접근할 위험이 생긴다.
- **방어:** 가능한 경우 **고정된 목적지 허용목록(allowlist)**을 사용한다. URL scheme·호스트·DNS 해석 결과·IP 대역을 검증하고 리다이렉트 후 주소도 재검증한다. 내부망·메타데이터 엔드포인트로 나가는 트래픽은 네트워크 차원에서도 제한한다.
- **Spring 기준:** `RestClient`, `WebClient`, `RestTemplate` 등이 외부 URL을 받아 호출하는 코드에서 URL 검증과 outbound 네트워크 정책을 검토한다.

**면접 한 줄:** "SSRF는 공격자 입력을 통해 서버가 의도하지 않은 주소에 직접 요청하도록 만드는 공격입니다. 목적지 허용목록과 실제 DNS/IP, 리다이렉트 검증, 네트워크 통제로 방어합니다."

**주의할 오해:** 단순히 URL 문자열에 `localhost`나 `127.0.0.1`이 포함됐는지만 검사하면 충분하지 않다. 여러 IP 표현, DNS 변화, 리다이렉트 등의 영향을 고려해야 한다.

## 6. 면접 단골 비교 질문

| 질문 | 20초 답변 핵심 |
| --- | --- |
| CSRF와 XSS는 어떤 차이가 있나요? | CSRF는 **요청 자체를 위조**, XSS는 **브라우저에서 악성 코드가 실행**되는 문제입니다. |
| CORS와 CSRF는 같은 보안 기능인가요? | 아닙니다. CORS는 **교차 출처 응답 읽기 정책**, CSRF는 **원치 않는 상태 변경 요청 방어** 문제입니다. |
| CSRF Token이 있으면 XSS도 안전한가요? | 아닙니다. XSS가 가능한 상황에서는 동일 출처의 코드가 토큰을 읽거나 요청을 실행할 수 있어 XSS도 별도로 막아야 합니다. |
| JWT 방식이면 CSRF를 꺼도 되나요? | 토큰이 **어떻게 전달되는지** 판단해야 합니다. 자동 전송되는 쿠키를 사용한다면 CSRF 위험이 남을 수 있습니다. |
| CORS 오류가 나면 서버가 API 호출을 못 받은 건가요? | 반드시 그렇지 않습니다. Preflight 실패는 실제 요청을 막을 수 있고, 단순 요청은 서버에 도달해도 브라우저가 응답 공유를 막을 수 있습니다. |
| SSRF와 CSRF는 무슨 차이인가요? | CSRF는 **사용자 브라우저가 인증된 요청**을 보내게 만들고, SSRF는 **서버가 네트워크 요청**을 보내게 만듭니다. |
| XSS 방어를 위해 입력만 필터링하면 충분한가요? | 아닙니다. **출력 맥락에 맞는 인코딩**이 기본이며 HTML 허용 시 sanitizer와 CSP 보조 통제를 검토해야 합니다. |
| Spring Security만 적용하면 SSRF도 막히나요? | 아닙니다. Spring Security의 인증·인가와 별도로, **외부 요청 대상 검증과 네트워크 egress 통제**가 필요합니다. |

## 7. 실무 검증 체크리스트

- [ ] **CSRF:** 인증 쿠키가 있는 상태 변경 요청에서 토큰 누락·불일치를 거부하는가?
- [ ] **XSS:** 사용자가 입력한 특수문자가 HTML·스크립트로 실행되지 않고 텍스트로 보이는가?
- [ ] **CORS:** 허용하지 않은 Origin의 브라우저 스크립트가 API 응답을 읽지 못하는가? 허용한 Origin은 정상 동작하는가?
- [ ] **SSRF:** URL 입력 기능이 허용하지 않은 내부·외부 목적지를 호출하지 않는가? 리다이렉트와 DNS/IP 변경도 고려했는가?
- [ ] **공통:** 실패를 로그 및 테스트로 확인했고, 브라우저 정책과 서버 측 인가를 혼동하지 않는가?

## 8. 하단 그림으로 네 가지 흐름 외우기

위에서 배운 내용을 **누가 요청을 시작하는지, 어디서 실행·판단하는지**로 구분하면 외우기 쉽다. 아래 도식은 실제 공격 구현 방법이 아니라 신뢰 경계를 설명하는 학습용 개념도다.

### 8-1. CSRF: 공격자 → 사용자 브라우저 → 정상 서비스

![CSRF: 공격자 사이트가 사용자를 유도하고 브라우저의 로그인 쿠키가 조건에 따라 첨부된 요청이 정상 서비스로 전달됨. CSRF 토큰 등 검증이 필요함.]({{ '/assets/diagrams/web-security-csrf-flow.svg' | relative_url }})

**기억할 지점:** 악용되는 것은 **사용자 브라우저의 로그인 상태**다.

### 8-2. XSS: 입력 문자열 → 취약한 화면 → 브라우저 실행

![XSS: 신뢰할 수 없는 입력이 화면에서 HTML 또는 DOM 코드로 해석되어 방문자 브라우저에서 스크립트가 실행될 수 있음.]({{ '/assets/diagrams/web-security-xss-flow.svg' | relative_url }})

**기억할 지점:** 데이터로 보여야 할 값이 **브라우저에서 코드로 실행**된다.

### 8-3. CORS: 다른 출처 화면 → 브라우저 정책 → API 응답

![CORS: 다른 출처의 웹페이지가 API를 요청할 때 브라우저가 출처 및 응답 헤더를 확인하여 스크립트의 응답 접근을 판단함.]({{ '/assets/diagrams/web-security-cors-flow.svg' | relative_url }})

**기억할 지점:** CORS는 **브라우저의 응답 읽기 허용 정책**이다. API의 실제 인가는 서버에서 별도로 검사해야 한다.

### 8-4. SSRF: 입력 URL → 백엔드 서버 → 내부 또는 제한된 대상

![SSRF: 공격자가 입력한 URL을 백엔드 서버가 검증 없이 요청하면 내부 및 제한된 서비스로 접근할 수 있으므로 목적지를 검증해야 함.]({{ '/assets/diagrams/web-security-ssrf-flow.svg' | relative_url }})

**기억할 지점:** 요청 주체는 브라우저가 아니라 **백엔드 서버**다.

## 9. 마지막 10초 요약

```text
CSRF: 로그인 상태를 이용한 요청 위조 → Token / Origin
XSS : 입력값이 스크립트로 실행      → Output Encoding / Safe DOM
CORS: 교차 출처 응답 읽기 정책     → Allowed Origin / Method
SSRF: 서버가 임의 주소에 요청      → Destination Allowlist / Egress
```

함께 읽기: [Spring Security 인증·인가와 면접 핵심 질문](https://github.com/son1004007/son1004007.github.io/blob/main/_posts/2026-10-09-spring-security-authentication-authorization-interview-guide.md)

## 공식 참고 문서

- [OWASP: CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP: XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN: Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [Spring Security: CORS](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)
- [OWASP: SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
