---
layout: page
title: Study
permalink: /study/
---

{% assign study_posts = site.posts | where_exp: "post", "post.categories contains 'study'" %}
{% if study_posts.size == 0 %}
No study posts yet.
{% else %}
{% for post in study_posts %}
- [{{ post.title }}]({{ post.url }}) ({{ post.date | date: "%Y-%m-%d" }})
{% endfor %}
{% endif %}
