---
layout: page
permalink: /archive/
title: Archive
---

<div id="archives">
{% assign postsByYearMonth = site.posts | group_by_exp:"post", "post.date | date: '%Y %b'"  %}
{% for yearMonth in postsByYearMonth %}
  <h3>{{ yearMonth.name }}</h3>
    <article class="archive-item">
      {% for post in yearMonth.items %}
        <h4>{{ post.date | date_to_string }}: <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h4>
      {% endfor %}
    </article>
    <br>
{% endfor %}
</div>
