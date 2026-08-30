# Category Guide

이 문서는 블로그 글 작성 시 사용할 카테고리 기준입니다.

## 기본 원칙

카테고리는 글의 큰 목적을 나타내고, 태그는 세부 기술과 키워드를 나타냅니다.

```text
Category = 큰 분류
Tag = 세부 기술, 도구, 상황, 자격증, 키워드
```

블로그는 크게 두 축으로 운영합니다.

- **Work / Tech / Study**: 실무, 기술, 학습, 경력 기록
- **Life / Notes**: 주택, 가족, 재무, 생활, 장기 의사결정 기록

두 축은 중요도의 차이가 아니라 **목록과 탐색 목적의 차이**입니다.

글을 유지할지 여부는 IT 관련성으로 판단하지 않습니다. 실제 경험·학습·판단을 다시 확인할 가치가 있으면 유지합니다.

## Standard Technical Categories

기술·업무 글은 아래 7개를 우선 사용합니다.

| Category | 사용 기준 | 예시 태그 |
|---|---|---|
| `backend` | 백엔드 개발, API, 인증/권한, Spring Boot, 웹 애플리케이션 구조 | Java, Spring Boot, JSP, MyBatis, REST API |
| `database` | DB 설계, SQL, Oracle, PostgreSQL, Tibero, 결과 테이블 설계 | PostgreSQL, Oracle, Tibero, SQL, Modeling |
| `infrastructure` | Linux, 웹서버, WAS, 컨테이너, 배포, 장애 대응 | Linux, Rocky Linux, Nginx, Apache, Tomcat, Podman |
| `data-systemization` | 분석 결과를 웹/API/DB/운영 시스템으로 연결하는 글 | Data Analysis, Batch, API, Requirements |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, 감사/감리, CISA, ISMS-P | Security, CISA, ISMS-P, Access Control, Audit |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 커뮤니케이션, 범위 조율 | PMP, Requirement, Report, Communication |
| `career` | 이직, 포트폴리오, 블로그 운영, AI 개발 워크플로, 경력 방향, 학습 전략 | Portfolio, Resume, Career, GitHub Pages, AI Workflow |

기존 `study` 카테고리 글은 과거 학습 기록으로 유지할 수 있습니다. 새 글은 내용에 맞는 표준 카테고리를 우선합니다.

## Personal Notes Policy

주택, 재무·생활계획, 가족 생활, 건강 습관, 여행, 장기 의사결정 등은 `Life / Notes` 영역에서 관리합니다.

이런 글에는 `Personal` 태그를 추가합니다.

```yaml
---
layout: post
title: "개인 기록 제목"
description: "나중에 다시 확인할 가치가 있는 생활·의사결정 기록"
date: YYYY-MM-DD
categories: [career]
tags: [Personal, Decision Log]
---
```

운영 규칙:

- `Personal` 태그 글은 홈의 기술·업무 목록과 분리합니다.
- `/notes/`에서 별도 목록으로 노출합니다.
- 주택 관련 기존 `Housing` 태그도 Personal Notes로 취급합니다.
- Life / Notes는 포트폴리오와 무관하다는 이유로 삭제하지 않습니다.
- 생활 기록도 필요하면 검색 노출되도록 제목과 description을 작성합니다.
- 공개하기에 지나치게 민감한 개인정보, 가족 세부정보, 정확한 자산·주소 등은 최소화합니다.

## 삭제/정리 기준

다음은 정리 후보입니다.

- Jekyll/Poole 등 테마에 기본 포함된 데모 포스트
- 단순 placeholder나 테스트 글
- 작성자 자신의 경험·학습·판단이 들어 있지 않은 템플릿 원문
- 중복 글
- 민감정보 때문에 공개 유지가 부적절한 글

**비IT 글이라는 이유는 삭제 근거가 아닙니다.**

## Existing Post Examples

| 글 | 분류 |
|---|---|
| ChatGPT를 활용해 GitHub Pages 기술 블로그를 시작합니다 | Work / Career |
| Rocky Linux 디스크 마운트 실패 대응 절차 | Work / Infrastructure |
| git | Study (기존 기록 유지) |
| 2030년까지 서울 내 집 마련을 어떻게 시도할 것인가 | Life / Notes |
