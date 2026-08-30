---
layout: page
title: Notes
description: 주택, 가족, 생활계획, 재무, 장기 의사결정처럼 다시 확인할 가치가 있는 개인 기록 목록입니다.
---

# Life / Notes

이 공간은 주택, 가족 생활, 재무, 생활 개선, 장기 계획과 개인적인 의사결정을 기록합니다.

기술·업무 글과 목록을 분리하는 이유는 중요도가 낮아서가 아니라 **찾는 목적과 맥락이 다르기 때문**입니다. IT와 직접 관련이 없어도 실제로 경험하고 판단한 내용이며 나중에 다시 볼 가치가 있다면 남깁니다.

## 기록 목록

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
<p>아직 공개된 Life / Notes 기록이 없습니다.</p>
{% endif %}

## 기록 기준

앞으로 생활·개인 기록은 `Personal` 태그를 사용합니다. 주택 관련 기존 기록은 `Housing` 태그도 같은 영역으로 취급합니다.

남길지 여부는 IT 관련성이 아니라 다음을 기준으로 판단합니다.

- 실제 경험이나 고민이 담겨 있는가
- 중요한 선택과 판단 근거가 있는가
- 나중에 다시 확인할 가치가 있는가
- 장기간 이어지는 생활·가족·주거·재무 계획과 연결되는가

테마 데모, placeholder, 중복 콘텐츠처럼 본인의 기록이라고 보기 어려운 글만 정리 대상이 됩니다.
