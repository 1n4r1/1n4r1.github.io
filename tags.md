---
layout: page
permalink: /tags/
title: Tags
---

<div id="tags">
{% for category in site.categories reversed %}
  <div class="tag-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>
    
    <h3 class="tag-head">{{ category_name }}</h3>
    <a name="{{ category_name | slugize }}"></a>
    {% for post in site.categories[category_name] %}
    <article class="archive-item">
      <h4>{{ post.date | date_to_string }}: <a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></h4>
    </article>
    {% endfor %}
  </div>
  <br>
{% endfor %}
</div>
