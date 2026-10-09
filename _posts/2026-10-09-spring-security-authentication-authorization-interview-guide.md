---
layout: post
title: "Spring Security 인증·인가 구조와 면접 핵심 질문 정리"
description: "Spring Security의 인증·인가 차이, 필터 체인, 세션, DB 로그인, RBAC, CSRF, SSO와 실무 면접 질문을 빠르게 복습할 수 있도록 정리합니다."
date: 2026-10-09
categories: [backend]
tags: [Java, Spring Boot, Spring Security, Authentication, Authorization, RBAC, Interview]
---

Spring Security를 사용해 인증·인가를 구현했다고 설명하려면 **누가 로그인했는지 확인(인증)**하고, **그 사용자가 어떤 기능에 접근할 수 있는지 판단(인가)**하는 과정을 코드와 요청 흐름으로 설명할 수 있어야 한다.

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

- **인증되지 않은 사용자:** 로그인 요구 또는 401 응답 등 인증 방식에 따른 처리.
- **인증되었지만 권한 부족:** 일반적으로 403 응답.
- **인증 성공:** SecurityContext를 현재 요청에서 사용. 세션 방식에서는 인증 컨텍스트가 다음 요청에도 복원되도록 저장해야 한다.
- **주의:** 모든 요청마다 ID/PW를 다시 검사하는 것은 아니다. 세션 방식에서는 저장된 인증 정보를 복원하고, Bearer/JWT 방식은 유효한 토큰을 요청마다 검사하는 구조가 일반적이다.
- **FilterChainProxy:** 여러 SecurityFilterChain 중 첫 번째로 일치하는 체인을 적용한다. 필터 순서가 중요하다.

## 3. 실제로 구현하는 기능과 책임

| 기능 | 핵심 구현 위치·구성 요소 | 확인할 사항 |
| --- | --- | --- |
| 접근 경로 분리 | `SecurityFilterChain`, `requestMatchers` | 공개·인증 필요·관리자 전용 경로 |
| ID/PW 로그인 | `AuthenticationManager`, `DaoAuthenticationProvider`, `UserDetailsService` | 계정 존재·비활성·비밀번호 검증 |
| 비밀번호 관리 | `PasswordEncoder` (예: BCrypt) | 평문 저장 금지, `matches`로 검증 |
| 인가 | `hasRole`, `hasAuthority`, `@PreAuthorize` | URL뿐 아니라 중요한 Service 동작도 보호 |
| 세션 관리 | `SecurityContextRepository`, `HttpSession` | 인증 컨텍스트 저장, 세션 고정 공격 방어 |
| 로그아웃 | logout 처리, 세션 무효화 | 재요청 시 인증이 남지 않는지 |
| CSRF 방어 | `CsrfFilter`, CSRF token | 세션 쿠키 기반 상태 변경 요청 |
| 실패 응답 | `AuthenticationEntryPoint`, `AccessDeniedHandler` | API의 401/403, 브라우저 로그인 리다이렉트 |
| SSO 연동 | 검증된 SSO/OIDC/SAML 연동 구성 | 신원 검증·계정 매핑·서비스 역할 적용 |

**중요:** `UserDetailsService`는 사용자 정보를 조회하는 역할이지 비밀번호 자체를 검증하는 구성 요소가 아니다. DB 비밀번호 인증은 일반적으로 `DaoAuthenticationProvider`가 `PasswordEncoder`를 사용해 검증한다.

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
- `hasRole("ADMIN")`: 기본적으로 `ROLE_ADMIN` 권한 검사. `hasAuthority("ADMIN")`과 단순히 같은 표현이 아니다.
- `anyRequest().authenticated()`: 앞의 규칙에 없는 경로의 기본 접근 정책.
- `@PreAuthorize("hasRole('ADMIN')")`: Service 메서드 수준에서도 권한 검사 가능. 메서드 보안 활성화와 프록시 적용 조건에 주의한다.
- 이 예시는 **인증·인가 설정 일부**이며, 실제 DB 조회 코드나 계정 모델 전체를 구현한 완성 예제는 아니다.

## 5. 면접 핵심 질문 15개와 짧은 답변

### 인증과 인가

**Q1. 인증과 인가의 차이는?**  
인증은 사용자 신원 확인이고, 인가는 인증된 주체가 리소스나 기능에 접근할 권한이 있는지 판단하는 과정이다.

**Q2. Spring Security는 언제 동작하나요?**  
Controller 전에 Servlet Filter 체인에서 요청을 처리한다. `FilterChainProxy`가 적절한 `SecurityFilterChain`을 선택하고 보안 필터가 실행된다.

**Q3. 로그인 인증 흐름을 설명해 보세요.**  
폼 로그인의 예에서는 인증 필터가 자격 증명을 받아 `AuthenticationManager`에 전달하고, `AuthenticationProvider`가 사용자 정보·비밀번호를 검증한다. 성공 시 인증된 `Authentication`을 `SecurityContext`에 저장한다.

**Q4. UserDetailsService와 PasswordEncoder의 역할은?**  
전자는 사용자 정보를 조회하고, 후자는 비밀번호를 안전하게 해시하고 입력 비밀번호가 저장된 해시와 일치하는지 검증한다.

**Q5. SecurityContext는 무엇인가요?**  
현재 처리 중인 요청의 인증 정보를 담는 컨텍스트다. `SecurityContextHolder`로 접근하며, 여러 요청에 걸쳐 유지하려면 저장소(예: 세션)를 사용한다.

### 권한과 세션

**Q6. 권한별 접근 제어는 어떻게 하나요?**  
`requestMatchers`와 `hasRole`/`hasAuthority`로 URL 접근을 제어한다. 중요한 비즈니스 작업은 `@PreAuthorize`나 Service 검증 등으로 추가로 보호한다.

**Q7. hasRole과 hasAuthority의 차이는?**  
기본 설정에서 `hasRole("ADMIN")`은 `ROLE_ADMIN`을 검사한다. `hasAuthority("ADMIN")`은 `ADMIN`이라는 권한 문자열 자체를 검사한다.

**Q8. 401과 403은 어떻게 다르나요?**  
API에서 401은 인증이 없거나 유효하지 않은 경우, 403은 인증은 되었지만 권한이 부족한 경우가 대표적이다. 폼 로그인은 401 대신 로그인 페이지로 리다이렉트할 수도 있고, CSRF 실패도 403이 될 수 있다.

**Q9. 세션 방식과 JWT 방식의 차이는?**  
세션 방식은 서버 측 세션을 통해 인증 상태를 유지한다. JWT는 서명 등 검증을 통해 토큰에 포함된 주체·권한 주장을 확인한다. JWT 자체로 로그아웃·폐기·권한 변경 문제가 해결되는 것은 아니다.

**Q10. 세션 고정 공격은 어떻게 막나요?**  
로그인 전후 동일한 세션 ID를 악용하지 못하도록 인증 성공 시 세션 ID를 변경하는 방식을 사용한다. HTTPS와 쿠키의 Secure, HttpOnly, SameSite 설정도 함께 검토한다.

### 웹 보안과 SSO

**Q11. CSRF란 무엇이며 언제 방어해야 하나요?**  
브라우저가 자동으로 보내는 인증 정보(예: 세션 쿠키)를 공격자가 악용해 원치 않는 요청을 발생시키는 공격이다. 세션 쿠키 기반 상태 변경 요청에는 CSRF token 등 방어가 필요하다. **REST API나 JWT 사용 여부만으로 무조건 CSRF를 꺼도 된다고 단정하지 않는다.**

**Q12. CORS와 CSRF는 어떻게 다르나요?**  
CORS는 브라우저의 교차 출처 응답 접근을 통제하는 정책이고, CSRF는 사용자의 인증 상태를 악용해 요청을 실행시키는 공격이다. CORS가 CSRF 방어를 대신하지 않는다.

**Q13. Filter와 Interceptor는 무엇이 다른가요?**  
Servlet Filter는 DispatcherServlet 이전을 포함한 Servlet 처리 경계에서 동작하며, Spring MVC Interceptor는 Handler 호출 전후에 동작한다. Spring Security의 웹 보안은 필터 기반이다.

**Q14. SSO 인증을 내부 권한과 연결하려면?**  
외부 신원 검증이 성공하면 내부 사용자 계정에 안전하게 매핑하고, 내부 계정 상태와 역할을 기준으로 인가한다. 외부 토큰의 역할 값을 그대로 관리자 권한으로 신뢰하지 않는다.

**Q15. 보안 설정을 어떻게 테스트하나요?**  
익명·일반·관리자 계정으로 정상 접근과 거부를 확인한다. 잘못된 비밀번호, 비활성 사용자, CSRF 누락, 로그아웃 이후 접근, 세션 고정, 잘못된 SSO 서명·대상·재사용 등을 실패 시나리오로 검증한다.

## 6. DB 로그인 + SSO 경험을 설명할 때

면접에서는 기능 이름 나열보다 **신원 확인 → 내부 계정 연결 → 권한 판단 → 세션 유지 → 거부 처리** 순서가 명료하다.

> Spring Security 기반으로 DB 로그인과 역할별 접근 제어를 구성하고, SSO 연동 시 외부에서 확인된 사용자 신원을 내부 계정과 연결해 서비스 권한으로 처리하는 방식을 다뤘습니다. 로그인 상태는 세션으로 유지하며 인증 실패와 접근 권한 부족을 구분했습니다. 구체적인 책임과 검증 범위는 당시 구현한 부분을 기준으로 설명드리겠습니다.

이 문장은 **설명 구조를 연습하기 위한 예시**다. 실제 면접에서는 직접 구현한 항목만 말하고, 타인이 구현한 영역이나 독립 재현 샘플의 검증 결과를 실제 고객 시스템 전체의 성과로 주장하지 않는다.

관련 독립 재현 사례: [Spring Security 인증 브리지](https://son1004007.github.io/engineering-career-portfolio/cases/spring-security-auth-bridge/). 이 샘플은 합성 계정과 SSO assertion을 사용하며, 실제 외부 IdP 운영 검증은 포함하지 않는다.

## 7. 실무 검증 체크리스트

- [ ] 비로그인 사용자의 보호 API 접근 차단 (API 응답 / 화면 리다이렉트 구분)
- [ ] USER 계정의 관리자 API 접근 차단, ADMIN 계정은 허용
- [ ] DB 비밀번호가 해시로 저장되고 입력값과 검증되는지
- [ ] 비활성·삭제된 내부 계정의 접근 차단
- [ ] 로그인 이후 SecurityContext와 세션이 유지되는지
- [ ] 로그아웃 후 기존 세션으로 재접근할 수 없는지
- [ ] CSRF token 없이 세션 기반 상태 변경 요청을 실행할 수 없는지
- [ ] 인증 성공 시 세션 고정 방어가 적용되는지
- [ ] SSO issuer, audience, 서명, 시간, 재사용, 계정 매핑이 적절히 검증되는지
- [ ] 인증 실패와 권한 부족을 코드 및 테스트에서 구분하는지

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

## 참고 문서

- [Spring Security 6.5 - Architecture](https://docs.spring.io/spring-security/reference/6.5/servlet/architecture.html)
- [Spring Security 6.5 - Username/Password Authentication](https://docs.spring.io/spring-security/reference/6.5/servlet/authentication/passwords/index.html)
- [Spring Security 6.5 - Request Authorization](https://docs.spring.io/spring-security/reference/6.5/servlet/authorization/authorize-http-requests.html)
- [Spring Security 6.5 - Session Management](https://docs.spring.io/spring-security/reference/6.5/servlet/authentication/session-management.html)
- [Spring Security 6.5 - CSRF](https://docs.spring.io/spring-security/reference/6.5/servlet/exploits/csrf.html)
