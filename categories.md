---
layout: page
title: Categories
description: 기술·업무 글을 백엔드, 데이터베이스, 인프라, 데이터 시스템화, 보안·감사, 프로젝트 관리, 커리어로 분류합니다.
---

# Categories

기술·업무 글은 아래 7개 카테고리를 기준으로 정리합니다. 주택이나 생활계획 같은 개인 기록은 <a href="{{ '/notes/' | relative_url }}">Notes</a>에서 별도로 관리합니다.

## Category Policy

| Category | 설명 |
|---|---|
| `backend` | Java, Spring Boot, API, 인증/권한, 웹 애플리케이션 구조 |
| `database` | SQL, DB 설계, Oracle, PostgreSQL, Tibero, 데이터 모델링 |
| `infrastructure` | Linux, Nginx, Apache, Tomcat, Podman, 배포, 장애 대응 |
| `data-systemization` | 분석 결과를 DB, API, 화면, 운영 시스템으로 연결하는 작업 |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, CISA, ISMS-P |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 고객 커뮤니케이션 |
| `career` | 이직, 포트폴리오, 기술 블로그 운영, AI 개발 워크플로, 경력 방향 |

## Posts by Category

### Backend
{% assign backend_posts = site.categories.backend %}
<ul>
{% for post in backend_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>

### Database
{% assign database_posts = site.categories.database %}
<ul>
{% for post in database_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>

### Infrastructure
{% assign infrastructure_posts = site.categories.infrastructure %}
<ul>
{% for post in infrastructure_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>

### Data Systemization
{% assign data_systemization_posts = site.categories['data-systemization'] %}
<ul>
{% for post in data_systemization_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>

### Security & Audit
{% assign security_audit_posts = site.categories['security-audit'] %}
<ul>
{% for post in security_audit_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>

### Project Management
{% assign project_management_posts = site.categories['project-management'] %}
<ul>
{% for post in project_management_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>

### Career / AI Workflow
{% assign career_posts = site.categories.career %}
<ul>
{% for post in career_posts %}
  {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
</ul>
