---
layout: post
title: "Spring Security 인증·인가 구조와 면접 핵심 질문 정리"
description: "Spring Security의 인증·인가 차이, 필터 체인, 세션, DB 로그인, RBAC, CSRF, SSO와 실무 면접 질문을 빠르게 복습할 수 있도록 정리합니다."
date: 2026-10-09
categories: [backend]
tags: [Java, Spring Boot, Spring Security, Authentication, Authorization, RBAC, Interview]
---

Spring Security는 사용자의 신원을 확인하는 **인증(Authentication)**과 사용자의 접근 권한을 판단하는 **인가(Authorization)**를 처리한다. 개발자는 요청을 처리하는 구성 요소와 각 구성 요소의 역할을 설명할 수 있어야 한다.

이 글은 **암기용 요약 → 구성 요소 → 처리 흐름 → 설정 예시 → 면접 질문 → 검증 항목** 순서로 정리했다. 설명과 예시는 Spring Security 6.5.x의 Servlet 기반 Spring MVC 애플리케이션을 기준으로 한다. 버전에 따라 설정 API와 기본 동작을 확인해야 한다.

## 1. 1분 암기 노트

| 용어 | 핵심 |
| --- | --- |
| Authentication (인증) | 사용자 신원 확인: 누구인가? |
| Authorization (인가) | 접근 허용 판단: 무엇을 할 수 있는가? |
| SecurityFilterChain | HTTP 요청에 적용할 보안 필터 구성 |
| AuthenticationManager | 인증 요청을 적절한 인증 처리자에게 위임 |
| AuthenticationProvider | 비밀번호·토큰 등 인증 수단을 검증 |
| UserDetailsService | 아이디 기반 사용자 정보 조회 (주로 DB 로그인에 사용) |
| PasswordEncoder | 비밀번호 해시 생성·검증 |
| Authentication | 인증된 주체 및 권한 정보를 담는 객체 |
| SecurityContext / SecurityContextHolder | 현재 요청에서 사용할 인증 정보 보관·조회 |
| SecurityContextRepository | 요청 간 인증 컨텍스트 저장·복원 (예: HTTP 세션) |
| GrantedAuthority / ROLE | 사용자에게 부여된 권한 / 역할 표현 |
| CSRF | 사용자의 기존 인증 상태를 악용하는 요청 위조 공격 |
| RBAC | 역할에 권한을 부여하고 역할로 접근을 통제하는 방식 |

**핵심 문장:** Spring Security는 Servlet Filter 체인에서 인증과 인가를 처리하고, 인증 결과를 SecurityContext에 보관하며, 요청별·메서드별 접근 정책으로 권한을 확인한다.

## 2. 요청과 인증의 전체 흐름

```text
HTTP 요청
  → Servlet Filter / FilterChainProxy
  → 일치하는 SecurityFilterChain 선택
  → SecurityContext 복원 + 보안 필터 적용
  → 인증 필터 (로그인 요청이거나 인증 정보가 있는 경우)
      → AuthenticationManager
      → AuthenticationProvider
      → UserDetailsService / PasswordEncoder (DB 비밀번호 인증의 예)
      → 인증된 Authentication 생성
  → AuthorizationFilter에서 접근 권한 판단
  → Controller → Service → Mapper
  → HTTP 응답
```

- **인증이 필요한 요청:** Spring Security는 인증 방식에 따라 로그인 페이지로 이동시키거나 HTTP 401 응답을 반환한다.
- **권한이 부족한 요청:** Spring Security는 일반적으로 HTTP 403 응답을 반환한다.
- **인증 성공:** Spring Security는 인증된 사용자의 Authentication을 SecurityContext에 보관한다. 세션 기반 서비스는 다음 요청에서 사용할 인증 컨텍스트를 세션 저장소에 기록한다.
- **로그인 상태 유지:** 세션 기반 서비스는 저장된 인증 컨텍스트를 다음 요청에서 복원한다. Bearer/JWT 기반 서비스는 요청에 첨부된 토큰을 검사해 사용자를 인증한다.
- **FilterChainProxy:** FilterChainProxy는 요청 경로에 처음 일치하는 SecurityFilterChain을 선택하고, 체인에 등록된 필터를 순서대로 실행한다.

## 3. 실제로 구현하는 기능과 책임

| 기능 | 핵심 구현 위치·구성 요소 | 확인할 사항 |
| --- | --- | --- |
| 접근 경로 분리 | `SecurityFilterChain`, `requestMatchers` | 공개·인증 필요·관리자 전용 경로 |
| ID/PW 로그인 | `AuthenticationManager`, `DaoAuthenticationProvider`, `UserDetailsService` | 계정 존재·비활성·비밀번호 검증 |
| 비밀번호 관리 | `PasswordEncoder` (예: BCrypt) | 비밀번호 해시 저장, `matches`로 검증 |
| 인가 | `hasRole`, `hasAuthority`, `@PreAuthorize` | Spring Security가 URL과 Service 메서드의 권한을 검사 |
| 세션 관리 | `SecurityContextRepository`, `HttpSession` | 인증 컨텍스트 저장, 세션 고정 공격 방어 |
| 로그아웃 | logout 처리, 세션 무효화 | 기존 세션의 재사용 차단 |
| CSRF 방어 | `CsrfFilter`, CSRF token | 세션 쿠키 기반 상태 변경 요청 |
| 실패 응답 | `AuthenticationEntryPoint`, `AccessDeniedHandler` | API의 401/403, 브라우저 로그인 리다이렉트 |
| SSO 연동 | 검증된 SSO/OIDC/SAML 연동 구성 | 신원 검증·계정 매핑·서비스 역할 적용 |

**역할 구분:** `UserDetailsService`는 사용자 정보를 조회한다. `DaoAuthenticationProvider`는 일반적인 ID/PW 인증에서 `PasswordEncoder`를 사용해 비밀번호를 검증한다.

## 4. SecurityFilterChain 설정 예시

다음은 **세션 기반 폼 로그인**과 역할 기반 URL 인가의 학습용 설정이다. DB 사용자 조회는 별도 `UserDetailsService`와 `PasswordEncoder` Bean 구성이 필요하다. `@EnableMethodSecurity`는 메서드 권한 검사를 사용할 때 활성화한다.

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/css/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .logout(Customizer.withDefaults());
        // 세션 쿠키 기반 폼 로그인에서는 CSRF 보호를 유지한다.
        return http.build();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

- `permitAll()`: 로그인 페이지 등 익명 접근 허용.
- `authenticated()`: 로그인한 사용자만 접근 허용.
- `hasRole("ADMIN")`: 기본 설정에서 `ROLE_ADMIN` 권한을 검사한다. `hasAuthority("ADMIN")`은 `ADMIN` 권한 문자열을 그대로 검사한다.
- `anyRequest().authenticated()`: Spring Security가 나머지 모든 경로에 사용자 인증을 요구한다.
- `@PreAuthorize("hasRole('ADMIN')")`: Service 메서드 수준에서도 권한 검사 가능. 메서드 보안 활성화와 프록시 적용 조건에 주의한다.
- **예시의 범위:** 이 설정은 HTTP 경로별 접근 정책과 폼 로그인·로그아웃을 보여준다. 실제 서비스는 사용자 조회용 UserDetailsService와 계정 저장소를 함께 구성한다.

## 5. 면접 핵심 질문 15개와 짧은 답변

### 인증과 인가

**Q1. 인증과 인가의 차이는?**  
인증은 사용자 신원 확인이고, 인가는 인증된 주체가 리소스나 기능에 접근할 권한이 있는지 판단하는 과정이다.

**Q2. Spring Security는 언제 동작하나요?**  
Spring Security는 Controller 호출 전에 Servlet Filter 체인에서 HTTP 요청을 처리합니다. `FilterChainProxy`는 일치하는 `SecurityFilterChain`을 선택하고 필터를 순서대로 실행합니다.

**Q3. 로그인 인증 흐름을 설명해 보세요.**  
폼 로그인에서 인증 필터는 ID/PW를 `AuthenticationManager`에 전달합니다. `AuthenticationProvider`는 사용자와 비밀번호를 검증합니다. Spring Security는 인증된 `Authentication`을 `SecurityContext`에 보관합니다.

**Q4. UserDetailsService와 PasswordEncoder의 역할은?**  
`UserDetailsService`는 사용자 정보를 조회합니다. `PasswordEncoder`는 비밀번호 해시를 생성하고 입력 비밀번호와 저장된 해시의 일치 여부를 확인합니다.

**Q5. SecurityContext는 무엇인가요?**  
`SecurityContext`는 현재 요청의 인증 정보를 보관합니다. 애플리케이션은 `SecurityContextHolder`로 인증 정보를 조회합니다. 세션 기반 서비스는 인증 컨텍스트를 세션에 저장해 다음 요청에서도 사용합니다.

### 권한과 세션

**Q6. 권한별 접근 제어는 어떻게 하나요?**  
Spring Security는 `requestMatchers`와 `hasRole`/`hasAuthority`로 URL 접근을 제어합니다. Service 계층은 `@PreAuthorize`와 업무 규칙 검증으로 중요한 기능의 권한을 확인합니다.

**Q7. hasRole과 hasAuthority의 차이는?**  
기본 설정에서 `hasRole("ADMIN")`은 `ROLE_ADMIN`을 검사한다. `hasAuthority("ADMIN")`은 `ADMIN`이라는 권한 문자열 자체를 검사한다.

**Q8. 401과 403은 어떻게 다르나요?**  
API는 인증이 필요한 경우 주로 HTTP 401을 반환하고, 인증된 사용자의 권한이 부족한 경우 주로 HTTP 403을 반환합니다. 폼 로그인 설정은 로그인 페이지로 이동시키기도 하며, CSRF 토큰 검증 실패에도 403 응답을 사용할 수 있습니다.

**Q9. 세션 방식과 JWT 방식의 차이는?**  
세션 기반 서비스는 서버 세션에 인증 상태를 보관합니다. JWT 기반 서비스는 토큰의 서명과 유효성, 주체·권한 정보를 검사합니다. JWT 기반 서비스는 로그아웃, 토큰 폐기와 권한 갱신 정책도 함께 설계합니다.

**Q10. 세션 고정 공격은 어떻게 막나요?**  
Spring Security는 인증 성공 시 세션 ID를 변경해 세션 고정 공격을 방어합니다. 서버는 HTTPS를 적용하고, 브라우저는 Secure·HttpOnly·SameSite 쿠키 설정에 따라 세션 쿠키를 보호합니다.

### 웹 보안과 SSO

**Q11. CSRF란 무엇이며 언제 방어해야 하나요?**  
CSRF 공격자는 사용자의 브라우저가 인증 쿠키와 함께 위조 요청을 전송하도록 유도합니다. 서버는 CSRF 토큰으로 상태 변경 요청을 검증합니다. 쿠키로 JWT를 자동 전송하는 서비스도 CSRF 방어를 적용합니다.

**Q12. CORS와 CSRF는 어떻게 다르나요?**  
CORS는 브라우저가 교차 출처 API 응답에 대한 JavaScript 접근을 제어하는 정책입니다. CSRF는 공격자가 사용자의 로그인 상태를 이용해 서버에 위조 요청을 보내는 공격입니다. 서버는 각 목적에 맞게 CORS 설정과 CSRF 보호 기능을 구성합니다.

**Q13. Filter와 Interceptor는 무엇이 다른가요?**  
Servlet Filter는 DispatcherServlet을 포함한 Servlet 처리 과정에서 동작합니다. Spring MVC Interceptor는 Handler 실행 전후에 동작합니다. Spring Security는 Servlet Filter를 통해 웹 보안을 적용합니다.

**Q14. SSO 인증을 내부 권한과 연결하려면?**  
SSO 연동 모듈은 외부에서 검증된 사용자 신원을 서비스 내부 계정에 연결합니다. 서비스는 내부 계정 상태와 역할을 기준으로 접근 권한을 판단합니다.

**Q15. 보안 설정을 어떻게 테스트하나요?**  
테스트는 익명·일반·관리자 계정으로 접근 허용과 차단 결과를 확인합니다. 테스트는 비밀번호 오류, 비활성 계정, CSRF 토큰 누락, 로그아웃 후 세션 재사용, SSO 서명·대상 오류와 재사용 요청에 대한 차단 결과도 검증합니다.

## 6. DB 로그인 + SSO 경험을 설명할 때

면접에서는 기능 이름 나열보다 **신원 확인 → 내부 계정 연결 → 권한 판단 → 세션 유지 → 거부 처리** 순서가 명료하다.

> Spring Security 기반으로 DB 로그인과 역할별 접근 제어를 구성하고, SSO 연동 시 외부에서 확인된 사용자 신원을 내부 계정과 연결해 서비스 권한으로 처리하는 방식을 다뤘습니다. 로그인 상태는 세션으로 유지하며 인증 실패와 접근 권한 부족을 구분했습니다. 구체적인 책임과 검증 범위는 당시 구현한 부분을 기준으로 설명드리겠습니다.

**면접 적용 기준:** 지원자는 자신이 직접 구현하거나 검증한 인증·인가 기능과 책임 범위를 구체적인 코드 흐름으로 설명한다. 공개 독립 재현 샘플의 테스트 결과는 해당 샘플의 검증 근거로 구분한다.

관련 독립 재현 사례: [Spring Security 인증 브리지](https://son1004007.github.io/engineering-career-portfolio/cases/spring-security-auth-bridge/). 이 샘플은 합성 계정과 서명된 SSO assertion을 사용해 인증·인가 흐름을 테스트한 독립 재현 사례다.

## 7. 실무 검증 체크리스트

- [ ] 테스트는 익명 사용자의 보호 API 접근 차단과 화면 로그인 리다이렉트를 구분해 확인한다.
- [ ] 테스트는 USER 역할의 관리자 API 접근 차단과 ADMIN 역할의 접근 허용을 확인한다.
- [ ] 테스트는 DB 비밀번호 해시 저장과 PasswordEncoder 검증 결과를 확인한다.
- [ ] 테스트는 비활성·삭제 계정의 접근 차단을 확인한다.
- [ ] 테스트는 로그인 이후 SecurityContext와 세션 유지 결과를 확인한다.
- [ ] 테스트는 로그아웃 처리 후 이전 세션의 접근 차단을 확인한다.
- [ ] 테스트는 CSRF 토큰이 누락된 상태 변경 요청의 차단을 확인한다.
- [ ] 테스트는 인증 성공 시 세션 ID 변경과 세션 고정 방어를 확인한다.
- [ ] 테스트는 SSO issuer·audience·서명·시간·재사용 검사 및 내부 계정 매핑을 확인한다.
- [ ] 테스트는 인증 실패와 접근 권한 부족에 대한 응답을 구분해 확인한다.

## 8. 암기용 최종 정리

```text
인증 = 누구인가?             Authentication
인가 = 무엇을 할 수 있나?   Authorization
입구 = SecurityFilterChain
검증 = AuthenticationManager → AuthenticationProvider
사용자 조회 = UserDetailsService
비밀번호 = PasswordEncoder
인증 결과 = Authentication → SecurityContext
접근 제어 = URL / 메서드 권한 검사, RBAC
로그인 유지 = 세션 또는 요청별 토큰 검증
웹 보호 = CSRF, 세션 관리, 보안 쿠키
실패 = 401/로그인 유도 vs 403/접근 거부
```

관련 학습: [CSRF·XSS·CORS·SSRF 차이와 면접 암기 노트]({{ '/web-security-csrf-xss-cors-ssrf/' | relative_url }})

## 참고 문서

- [Spring Security 6.5 - Architecture](https://docs.spring.io/spring-security/reference/6.5/servlet/architecture.html)
- [Spring Security 6.5 - Username/Password Authentication](https://docs.spring.io/spring-security/reference/6.5/servlet/authentication/passwords/index.html)
- [Spring Security 6.5 - Request Authorization](https://docs.spring.io/spring-security/reference/6.5/servlet/authorization/authorize-http-requests.html)
- [Spring Security 6.5 - Session Management](https://docs.spring.io/spring-security/reference/6.5/servlet/authentication/session-management.html)
- [Spring Security 6.5 - CSRF](https://docs.spring.io/spring-security/reference/6.5/servlet/exploits/csrf.html)
