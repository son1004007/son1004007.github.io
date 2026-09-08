---
layout: post
title: "커리어 채용공고 분석 V2: 연봉 추정을 버리고 기업의 경력 인정 수준을 본다"
date: 2026-09-08
categories: [career-data-analysis]
tags: [Career, JobKorea, Data Analysis, Backend, AI, Identity, IT Audit, Employer Recognition]
---

앞선 채용공고 데이터 분석에서는 JobKorea의 native filter를 이용해 서울·경력·정규직·우량 기업형태의 관련 IT 공고 182건을 수집하고, 182건 전체의 상세 JD를 확보했다.

그 분석에서 한 가지 기준을 다시 수정했다.

> **한국 경력직 채용에서는 지원 전에 실제 제시 연봉을 알기 어렵기 때문에, 회사 평균연봉이나 인터넷 연봉 추정치를 후보 순위의 핵심 근거로 사용하면 안 된다.**

회사 평균연봉은 해당 회사의 전체 구성원 평균일 뿐이고, 특정 직무·직급에서 내가 받을 오퍼와 동일하지 않다. 실제 처우는 최종합격 이후 처우협의에서 확인되는 경우가 많다.

그래서 V2에서는 연봉 proxy를 순위와 탈락 조건에서 제거했다.

---

# V2에서 무엇을 보나

지원 전에는 다음 순서로 본다.

```text
회사 실명 확인
→ 자체 제품/핵심사업 여부
→ 재무·사업 안정성
→ 역할의 업무구조
→ 현재 직접경력 인정 가능성
→ 직함/레벨 reset 위험
→ 실제 채용 과정의 경력 인정 신호
→ 최종 오퍼 후 실제 보상 비교
```

지원 전의 compensation은 의도적으로 `UNKNOWN`으로 둔다.

실제 오퍼나 회사가 공식적으로 제시한 range가 나오기 전에는 예상 연봉 숫자를 만들지 않는다.

---

# 더 중요한 데이터: 기업이 내 경력을 어떤 레벨로 보는가

연봉을 모르더라도 면접 과정에서는 꽤 중요한 정보를 얻을 수 있다.

예를 들어 기업이 나를:

- 현재보다 낮은 레벨로 보는지
- 일부 경력만 인정하는지
- 현재 경력을 온전히 인정하는지
- Senior/Manager급 자산으로 인정하는지

를 볼 수 있다.

앞으로 실제 지원에서는 다음 값을 기록한다.

```text
recognition_level_signal
years_credited_if_explicit
title_or_level_signal
scope_signal
```

예를 들어 두 직무에 모두 합격했다고 해보자.

```text
Backend
→ Senior Engineer scope

IT Audit
→ Junior / entry audit scope
```

이 경우 연봉을 아직 몰라도 Backend 시장이 내 현재 경력을 더 높은 수준으로 인정한다고 볼 수 있다.

반대로:

```text
Backend
→ Mid-level 5년차

Technical Audit
→ Manager-level technical auditor
```

라면 전체 IT 경력이 Audit 시장에서 더 높은 seniority로 환산될 가능성이 있다.

즉 커리어 선택에서 중요한 질문이 바뀐다.

> **어디가 평균연봉이 높은가?**

보다

> **어느 시장이 지금까지 쌓은 경력을 더 높은 레벨로 인정하는가?**

를 먼저 확인한다.

---

# 연봉 proxy를 제거한 뒤에도 1순위는 바뀌지 않았다

V2에서도 현재 가장 강한 시장 테스트는 **좋은 Product Backend / AI / Identity Backend**다.

이유는 연봉 추정치가 아니다.

- 현재 Backend/응용SW 직접경력을 가장 적게 리셋한다.
- Java/Spring, Python, SQL/RDBMS, Linux/Docker, LLM/RAG/Agent 경험을 직접 활용한다.
- 자체 제품을 만드는 우량회사에서 실제 공고가 존재한다.
- 새로운 도메인의 직접경력 수년을 먼저 요구하지 않는 공고가 있다.

대표적인 사례는 다음과 같다.

## 당근 Backend - Identity Service

이 역할은 Backend 3년 이상을 기본 feeder로 보고, OAuth 2.0/OIDC 경험은 우대한다.

따라서 Identity를 별도의 IAM 커리어로 완전히 갈아타는 역할이라기보다:

```text
Backend
+ 인증/인가
+ Identity domain
```

으로 전문화하는 경로에 가깝다.

현재 SSO/RBAC/Backend/보안 경험을 모두 사용할 수 있으면서도 direct IAM 경력 N년을 먼저 요구하지 않는다는 점이 중요하다.

## 무신사 Core AI Backend

개발 5년 이상 또는 그에 준하는 역량을 요구하고 Java/Spring, Python, PostgreSQL, AI Agent 경험이 현재 업무와 직접 겹친다.

이 역할의 시장 테스트에서 확인할 것은 예상연봉이 아니라:

> 현재 개발경력을 실제로 5년급 Product AI Backend로 인정하는가?

이다.

## 오늘의집 / 당근페이 / 토스 계열 Product Backend

이 역할들도 현재 Backend 경력을 유지하면서 제품회사로 이동할 수 있는 시장 테스트 대상이다.

다만 금융·결제·대규모 서비스는 장애대응, 고가용성, on-call responsibility를 실제 면접에서 확인해야 한다.

---

# Technical IT Audit은 2순위 선택적 테스트

Technical IT Audit도 완전히 제외하지 않는다.

일부 공고는 개발·운영·인프라·보안을 포함한 일반 IT 경력을 Audit feeder로 인정한다.

하지만 V2에서는 평균연봉 기대치를 제거했기 때문에 훨씬 엄격하게 본다.

다음 조건이 필요하다.

```text
기존 IT경력을 feeder로 명시
+ direct audit N년이 절대 필수가 아님
+ 정규직
+ 회사 안정성 양호
+ 직급/레벨 reset이 과도하지 않음
```

Technical Audit에서 확인할 핵심 데이터는:

> **11년 전체 IT경력을 회사가 어떤 Audit level/title/scope로 인정하는가?**

이다.

단순히 CISA가 있거나 Audit이라는 직함이 좋아 보여서 이동하지 않는다.

---

# AI Product Security는 작은 실험으로 유지

일반 AppSec 공고는 Secure SDLC, SAST/DAST/SCA, Threat Modeling, 취약점 분석 같은 직접경력을 요구하는 경우가 많다.

그래서 Product Security 전체를 1순위로 올리지 않는다.

대신 당근 AI Security처럼 LLM/RAG/Agent 구현경험과 개인 연구·CTF·블로그 같은 실제 증거를 인정하는 역할은 작은 비용으로 테스트할 수 있다.

기존 Agent/RAG 서비스에 대해:

- Prompt Injection
- Tool Abuse
- RAG Poisoning

같은 공격 시나리오를 재현한 짧은 보고서를 만든 뒤 실제 시장 반응을 확인하는 방식이다.

---

# 최종 V2 순서

## 1순위

**Good Product Backend / AI / Identity Backend**

현재 직접경력을 가장 적게 리셋하면서 자체 제품 도메인으로 이동할 수 있다.

## 2순위

**Broad-feeder Technical IT Audit / Assurance**

기존 IT경력을 seniority에 실제 반영하는 회사만 선택적으로 테스트한다.

## 3순위

**AI Product Security / development-friendly AppSec**

직접 AppSec 경력 N년보다 개발·AI·보안 실전 evidence를 인정하는 역할만 본다.

## 후순위

- direct Cloud Security
- IT SOX
- Privacy Governance
- 순수 Security Policy
- 고객사 상주 SI
- Forward Deployed / Support-heavy
- 구조적 24x7/on-call 역할

---

# 실제 지원 데이터가 다음 분석의 핵심이다

앞으로는 지원 결과마다 다음을 기록한다.

```text
회사
직무
지원일
서류 결과
면접 결과
기업이 인정한 경력 level
명시적으로 인정한 경력연수
직함/레벨
업무 scope
실제 on-call/WLB 정보
최종 오퍼 여부
실제 제시 보상
```

보상은 마지막에 기록한다.

그 전에는 알 수 없는 숫자를 예측하지 않는다.

이번 V2의 핵심 결론은 다음과 같다.

> **좋은 회사에서 얼마를 받을지 미리 맞히는 것보다, 지금까지 쌓은 경력을 어느 시장이 더 높은 레벨로 인정하는지 먼저 검증한다. Product Backend/AI/Identity를 1차로 테스트하고, Technical Audit과 AI Product Security는 비교군으로 병렬 테스트한다. 실제 보상은 오퍼가 나온 뒤 최종 선택에 사용한다.**
