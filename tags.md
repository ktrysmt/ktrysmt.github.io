---
title: Tags
permalink: /tags/
layout: list
description: "記事をタグ別に分類した一覧ページです。"
---

{% assign sorted_tags = site.tags | sort %}
<div class="tag-index">
  <p class="tag-cloud">
    {% for tag in sorted_tags %}
      <a href="#{{ tag[0] }}" class="tag-link">{{ tag[0] }} ({{ tag[1].size }})</a>
    {% endfor %}
  </p>
</div>

{% for tag in sorted_tags %}
<h2 id="{{ tag[0] }}">{{ tag[0] }} <span class="tag-count">({{ tag[1].size }})</span></h2>
<ul class="tag-post-list">
  {% assign sorted_posts = tag[1] | sort: "date" | reverse %}
  {% for post in sorted_posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="dates">{{ post.date | date: "%b %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
{% endfor %}
