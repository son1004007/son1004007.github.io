# AGENTS.md

이 파일은 ChatGPT, Codex, Claude Code 등 AI 도구가 이 저장소를 수정할 때 먼저 읽어야 하는 작업 기준입니다.

## Repository Purpose

`son1004007/son1004007.github.io`는 GitHub Pages 기반 기술 블로그입니다.

목적은 실무 경험을 이력서와 포트폴리오에서 사용할 수 있는 공개 기술 기록으로 정리하는 것입니다.

## Read Order

작업을 시작하기 전에 아래 순서로 확인합니다.

1. `README.md`
2. `AGENTS.md`
3. `docs/category-guide.md`
4. `docs/blog-writing-guide.md`
5. `docs/post-template.md`
6. `_config.yml`
7. 최근 `_posts/` 글

## Core Writing Principle

기본 글 구조는 다음을 우선합니다.

```text
문제점 -> 원인 -> 해결 -> 실행 방법 -> 재발 방지 / 개선 방향 -> 포트폴리오 관점의 의미
```

글은 기술 블로그이지만 이력서/포트폴리오 보조 자료로 사용될 수 있어야 합니다.

## Target Identity

작성자는 다음 정체성을 기준으로 포지셔닝합니다.

- 백엔드/풀스택 개발자
- Java, Spring Boot, PostgreSQL, Python, FastAPI 실무 경험 보유
- Linux 서버 운영과 웹 서비스 배포 경험 보유
- 데이터 분석 결과를 시스템화하는 업무 경험 보유
- 보안/감사/내부통제 직무 전환 가능성을 준비 중

## Category Policy

카테고리는 글의 큰 목적을 나타내고, 태그는 세부 기술과 키워드를 나타냅니다.

카테고리는 아래 7개로 고정합니다.

| Category | 사용 기준 |
|---|---|
| `backend` | Java, Spring Boot, API, 인증/권한, 웹 애플리케이션 구조 |
| `database` | SQL, DB 설계, Oracle, PostgreSQL, Tibero, 데이터 모델링 |
| `infrastructure` | Linux, Nginx, Apache, Tomcat, Podman, 배포, 장애 대응 |
| `data-systemization` | 분석 결과를 DB, API, 화면, 운영 시스템으로 연결하는 작업 |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, CISA, ISMS-P |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 고객 커뮤니케이션 |
| `career` | 이직, 포트폴리오, 기술 블로그 운영, 경력 방향 |

다음 값은 카테고리로 쓰지 않습니다. 필요하면 태그로 사용합니다.

- `study`
- `linux`
- `spring`
- `pmp`
- `cisa`
- `troubleshooting`
- `web`
- `devops`

자세한 기준은 `docs/category-guide.md`를 따릅니다.

## Post Frontmatter

Jekyll 포스트는 아래 형식을 사용합니다.

```yaml
---
layout: post
title: "글 제목"
date: YYYY-MM-DD
categories: [infrastructure]
tags: [Linux, Troubleshooting]
---
```

파일명은 아래 형식을 따릅니다.

```text
_posts/YYYY-MM-DD-kebab-case-title.md
```

## Category Selection Examples

```yaml
# Spring Boot 장애 대응
categories: [backend]
tags: [Java, Spring Boot, Troubleshooting]
```

```yaml
# Linux / Apache / Podman 운영 장애
categories: [infrastructure]
tags: [Linux, Apache, Podman]
```

```yaml
# 분석 결과를 화면/API로 연결
categories: [data-systemization]
tags: [Data Analysis, API, Requirements]
```

```yaml
# PMP 학습 또는 요구사항 정리
categories: [project-management]
tags: [PMP, Requirement, Communication]
```

```yaml
# CISA / ISMS-P / 내부통제
categories: [security-audit]
tags: [CISA, ISMS-P, Internal Control]
```

## Public Safety Rules

절대 공개하지 않습니다.

- 고객사 내부 IP
- 서버 계정, 비밀번호, 토큰, 키
- 고객사 비공개 회의록 원문
- 계약상 비공개 자료
- 내부 시스템의 정확한 경로와 접속 정보
- 고객사 실명과 장애 내용을 직접 연결하는 표현
- 개인정보

업무 경험은 일반화합니다.

```text
Bad: A 고객사 192.168.x.x 서버에서 발생한 장애
Good: 공공기관 Linux 서버에서 발생한 마운트 실패 대응 절차
```

## Tone and Style

- 한국어 존댓말보다 문서체를 우선합니다.
- 불필요한 감상문을 줄입니다.
- 문제, 판단 기준, 실행 방법을 분리합니다.
- 명령어와 설정 예시는 재현 가능하게 작성합니다.
- 글 제목은 검색과 이력서 링크에 적합하게 구체적으로 작성합니다.

## Recommended Topics

우선 작성할 가치가 높은 글입니다.

1. Rocky Linux 디스크 마운트 실패 대응
2. Apache VirtualHost 404 장애 분석
3. Nginx + Tomcat + SSL 운영 구조
4. Podman 기반 CloudBeaver 운영 검토
5. Spring Boot + JSP 운영 환경 장애 대응
6. 분석 결과를 웹 서비스로 시스템화할 때 고려할 점
7. PMP, CISA, ISMS-P 학습 내용을 실무 관점으로 정리하는 방법

## Do Not Do

- 테마 원본 README로 되돌리지 않습니다.
- 공개 글에 고객사 식별 정보를 넣지 않습니다.
- 단순 개념 요약만 작성하지 않습니다.
- ChatGPT 대화 원문을 그대로 붙여 넣지 않습니다.
- 실무 경험을 과장하지 않습니다.
- 표준 카테고리 외 값을 임의로 추가하지 않습니다.

## Done Criteria

글 하나를 추가할 때 완료 기준은 다음과 같습니다.

1. Jekyll frontmatter가 있다.
2. 표준 카테고리 7개 중 하나를 사용했다.
3. 태그는 세부 기술과 키워드 중심으로 지정했다.
4. 문제점/원인/해결/실행 방법 구조가 명확하다.
5. 민감정보가 제거되어 있다.
6. 이력서/포트폴리오에서 링크해도 부끄럽지 않은 수준이다.
7. 제목만 봐도 어떤 역량을 보여주는 글인지 알 수 있다.
