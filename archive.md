---
layout: default
title: Blog
---

# Blog

실무 문제 해결, 서버 운영, 백엔드 개발, 데이터 분석 시스템화, 보안/감사 전환 과정을 정리한 글입니다.

처음 방문했다면 아래 글부터 확인하면 됩니다.

## 먼저 볼 글

<ul>
  <li><a href="{{ '/career/2026/06/21/start-github-pages-blog-with-chatgpt/' | relative_url }}">ChatGPT를 활용해 GitHub Pages 기술 블로그를 시작합니다</a></li>
  <li><a href="{{ '/infrastructure/2026/06/22/rocky-linux-disk-mount-failure/' | relative_url }}">Rocky Linux 디스크 마운트 실패 대응 절차</a></li>
  <li><a href="{{ '/infrastructure/2026/06/22/apache-virtualhost-404-troubleshooting/' | relative_url }}">Apache VirtualHost 404 장애 원인 분석</a></li>
  <li><a href="{{ '/infrastructure/2026/06/22/cloudbeaver-with-podman/' | relative_url }}">Podman 기반 CloudBeaver 운영 검토</a></li>
  <li><a href="{{ '/data-systemization/2026/06/22/data-analysis-systemization/' | relative_url }}">분석 결과를 웹 서비스로 시스템화할 때 고려할 점</a></li>
</ul>

## 전체 글

{% assign postsByYearMonth = site.posts | group_by_exp: "post", "post.date | date: '%Y-%m'" %}
{% for yearMonth in postsByYearMonth %}
  {% assign year = yearMonth.name | slice: 0, 4 %}
  {% if year == "2026" %}
  <h2>{{ yearMonth.name }}</h2>
  <ul>
    {% for post in yearMonth.items %}
      <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
  {% endif %}
{% endfor %}
