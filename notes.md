---
layout: page
title: Notes
description: 주택, 생활계획 등 기술·업무 글과 분리해 관리하는 개인 기록 목록입니다.
---

# Notes

기술·업무 글과 성격이 다른 개인 기록을 별도로 모아두는 공간입니다.

이 목록은 메인 기술 글 목록과 분리합니다. 주택, 생활계획, 장기 의사결정처럼 다시 확인할 가치가 있지만 기술 포트폴리오의 중심 흐름에는 포함하지 않을 내용을 정리합니다.

## 개인 기록 목록

<ul>
{% assign personal_count = 0 %}
{% for post in site.posts %}
  {% if post.tags contains 'Personal' or post.tags contains 'Housing' %}
    {% assign personal_count = personal_count | plus: 1 %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: '%Y-%m-%d' }}</small>
    </li>
  {% endif %}
{% endfor %}
</ul>

{% if personal_count == 0 %}
<p>아직 공개된 개인 기록이 없습니다.</p>
{% endif %}

## 분류 기준

앞으로 개인 기록은 `Personal` 태그를 사용합니다. 주택 관련 기존 기록은 `Housing` 태그도 개인 기록으로 취급합니다.

기술·업무 경험은 기존 7개 기술 카테고리를 사용하고, 개인 기록은 메인 홈과 Topics의 기술 목록에서 분리합니다.
