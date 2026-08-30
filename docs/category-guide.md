# Category Guide

이 문서는 블로그 글 작성 시 사용할 카테고리 기준입니다.

## 기본 원칙

카테고리는 글의 큰 목적을 나타내고, 태그는 세부 기술과 키워드를 나타냅니다.

```text
Category = 큰 분류
Tag = 세부 기술, 도구, 상황, 자격증, 키워드
```

기술·업무 글의 카테고리는 늘리지 않고 아래 7개를 우선 사용합니다.

## Standard Categories

| Category | 사용 기준 | 예시 태그 |
|---|---|---|
| `backend` | 백엔드 개발, API, 인증/권한, Spring Boot, 웹 애플리케이션 구조 | Java, Spring Boot, JSP, MyBatis, REST API |
| `database` | DB 설계, SQL, Oracle, PostgreSQL, Tibero, 결과 테이블 설계 | PostgreSQL, Oracle, Tibero, SQL, Modeling |
| `infrastructure` | Linux, 웹서버, WAS, 컨테이너, 배포, 장애 대응 | Linux, Rocky Linux, Nginx, Apache, Tomcat, Podman |
| `data-systemization` | 분석 결과를 웹/API/DB/운영 시스템으로 연결하는 글 | Data Analysis, Batch, API, Requirements |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, 감사/감리, CISA, ISMS-P | Security, CISA, ISMS-P, Access Control, Audit |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 커뮤니케이션, 범위 조율 | PMP, Requirement, Report, Communication |
| `career` | 이직, 포트폴리오, 블로그 운영, AI 개발 워크플로, 경력 방향, 학습 전략 | Portfolio, Resume, Career, GitHub Pages, AI Workflow |

## Category Selection Rules

### 1. 운영 장애 대응 글

Linux, Apache, Nginx, Tomcat, Podman, Docker, SSL, 배포 관련 장애는 `infrastructure`를 사용합니다.

```yaml
categories: [infrastructure]
tags: [Linux, Apache, Troubleshooting]
```

### 2. 백엔드 애플리케이션 글

Spring Boot, Java, JSP, 인증, API, 서비스 구조는 `backend`를 사용합니다.

```yaml
categories: [backend]
tags: [Java, Spring Boot, REST API]
```

### 3. DB 중심 글

SQL, 테이블 설계, Oracle, PostgreSQL, Tibero, 성능, 데이터 모델링은 `database`를 사용합니다.

```yaml
categories: [database]
tags: [PostgreSQL, Oracle, SQL, Modeling]
```

### 4. 분석 결과 시스템화 글

분석 결과를 DB, API, 화면, 배치, 운영 시스템으로 연결하는 글은 `data-systemization`을 사용합니다.

```yaml
categories: [data-systemization]
tags: [Data Analysis, Systemization, Batch, API]
```

### 5. 보안/감사/내부통제 글

접근통제, 로그관리, 권한관리, CISA, ISMS-P, 내부통제는 `security-audit`을 사용합니다.

```yaml
categories: [security-audit]
tags: [CISA, ISMS-P, Access Control, Audit]
```

### 6. 프로젝트 관리/보고/요구사항 글

PMP, 일일보고, 업무일지, 고객 커뮤니케이션, 분석 범위와 개발 범위 조율은 `project-management`를 사용합니다.

```yaml
categories: [project-management]
tags: [PMP, Requirement, Report, Communication]
```

### 7. 경력/포트폴리오/블로그 운영 글

이직 준비, 경력 방향, 포트폴리오 구축, GitHub Pages 블로그 운영, ChatGPT/Codex 기반 개발 워크플로는 `career`를 사용합니다.

```yaml
categories: [career]
tags: [Portfolio, Resume, GitHub Pages, AI Workflow]
```

## Personal Notes Policy

주택, 재무·생활계획, 가족 생활, 개인 장기 의사결정처럼 기술·업무 포트폴리오의 중심 흐름과 다른 글은 메인 기술 글 목록과 분리합니다.

이런 글에는 `Personal` 태그를 반드시 추가합니다.

```yaml
---
layout: post
title: "개인 기록 제목"
description: "개인적으로 다시 확인할 가치가 있는 장기 의사결정 기록"
date: YYYY-MM-DD
categories: [career]
tags: [Personal, Decision Log]
---
```

운영 규칙:

- `Personal` 태그 글은 홈의 기술·업무 글 목록에서 제외합니다.
- `Blog`와 `Topics`의 기술 목록에서도 제외합니다.
- `/notes/`에서 별도 목록으로 노출합니다.
- 주택 관련 기존 글의 `Housing` 태그도 호환을 위해 Personal Notes로 취급합니다.
- 개인 기록을 기술 카테고리 체계 확장의 이유로 사용하지 않습니다.
- 공개하기에 지나치게 민감한 개인정보, 가족 세부정보, 자산 상세값 등은 넣지 않습니다.

## Do Not Use as Category

아래 값은 기술·업무 카테고리로 쓰지 않습니다. 필요하면 태그로 사용합니다.

- `study`
- `linux`
- `spring`
- `pmp`
- `cisa`
- `troubleshooting`
- `web`
- `devops`
- `personal`
- `housing`

예시:

```yaml
# Bad
categories: [study]
tags: [PMP]

# Good
categories: [project-management]
tags: [PMP, Study]
```

```yaml
# Bad
categories: [linux]
tags: [Rocky Linux]

# Good
categories: [infrastructure]
tags: [Linux, Rocky Linux]
```

## Existing Post Category Mapping

| 글 | Category |
|---|---|
| ChatGPT를 활용해 GitHub Pages 기술 블로그를 시작합니다 | `career` |
| Rocky Linux 디스크 마운트 실패 대응 절차 | `infrastructure` |
| Apache VirtualHost 404 장애 원인 분석 | `infrastructure` |
| Podman 기반 CloudBeaver 운영 검토 | `infrastructure` |
| 분석 결과를 웹 서비스로 시스템화할 때 고려할 점 | `data-systemization` |
