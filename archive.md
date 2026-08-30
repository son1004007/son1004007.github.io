---
layout: default
title: Blog
description: 백엔드 개발, Linux 운영, 데이터 시스템화, 보안·감사, AI 개발 워크플로 관련 기술·업무 글 목록입니다.
---

# Blog

실무 문제 해결, 서버 운영, 백엔드 개발, 데이터 분석 시스템화, 보안/감사 전환 과정을 정리한 기술·업무 글입니다.

개인적인 생활·의사결정 기록은 <a href="{{ '/notes/' | relative_url }}">Notes</a>에서 별도로 확인할 수 있습니다.

## 전체 기술·업무 글

{% assign postsByYearMonth = site.posts | group_by_exp: "post", "post.date | date: '%Y-%m'" %}
{% for yearMonth in postsByYearMonth %}
  {% assign visible_count = 0 %}
  {% for post in yearMonth.items %}
    {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
      {% if post.categories contains 'backend' or post.categories contains 'database' or post.categories contains 'infrastructure' or post.categories contains 'data-systemization' or post.categories contains 'security-audit' or post.categories contains 'project-management' or post.categories contains 'career' %}
        {% assign visible_count = visible_count | plus: 1 %}
      {% endif %}
    {% endunless %}
  {% endfor %}

  {% if visible_count > 0 %}
  <h2>{{ yearMonth.name }}</h2>
  <ul>
    {% for post in yearMonth.items %}
      {% unless post.tags contains 'Personal' or post.tags contains 'Housing' %}
        {% if post.categories contains 'backend' or post.categories contains 'database' or post.categories contains 'infrastructure' or post.categories contains 'data-systemization' or post.categories contains 'security-audit' or post.categories contains 'project-management' or post.categories contains 'career' %}
        <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
        {% endif %}
      {% endunless %}
    {% endfor %}
  </ul>
  {% endif %}
{% endfor %}
