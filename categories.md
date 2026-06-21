---
layout: page
title: Categories
---

# Categories

이 블로그의 글은 아래 7개 카테고리를 기준으로 정리합니다.

카테고리는 큰 분류를 나타내고, 태그는 세부 기술과 키워드를 나타냅니다.

## Category Policy

| Category | 설명 |
|---|---|
| `backend` | Java, Spring Boot, API, 인증/권한, 웹 애플리케이션 구조 |
| `database` | SQL, DB 설계, Oracle, PostgreSQL, Tibero, 데이터 모델링 |
| `infrastructure` | Linux, Nginx, Apache, Tomcat, Podman, 배포, 장애 대응 |
| `data-systemization` | 분석 결과를 DB, API, 화면, 운영 시스템으로 연결하는 작업 |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, CISA, ISMS-P |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 고객 커뮤니케이션 |
| `career` | 이직, 포트폴리오, 기술 블로그 운영, 경력 방향 |

## Posts by Category

### Backend

{% assign backend_posts = site.categories.backend %}
{% if backend_posts %}
<ul>
  {% for post in backend_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}

### Database

{% assign database_posts = site.categories.database %}
{% if database_posts %}
<ul>
  {% for post in database_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}

### Infrastructure

{% assign infrastructure_posts = site.categories.infrastructure %}
{% if infrastructure_posts %}
<ul>
  {% for post in infrastructure_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}

### Data Systemization

{% assign data_systemization_posts = site.categories['data-systemization'] %}
{% if data_systemization_posts %}
<ul>
  {% for post in data_systemization_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}

### Security & Audit

{% assign security_audit_posts = site.categories['security-audit'] %}
{% if security_audit_posts %}
<ul>
  {% for post in security_audit_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}

### Project Management

{% assign project_management_posts = site.categories['project-management'] %}
{% if project_management_posts %}
<ul>
  {% for post in project_management_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}

### Career

{% assign career_posts = site.categories.career %}
{% if career_posts %}
<ul>
  {% for post in career_posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% else %}
<p>작성 예정입니다.</p>
{% endif %}
