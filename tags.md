---
title: Tags
permalink: /tags/
layout: list
description: "記事をタグ別に分類した一覧ページです。"
---

{% comment %}タグを最新投稿日順にソート{% endcomment %}
{% assign tag_entries = "" %}
{% for tag in site.tags %}
  {% assign latest_post = tag[1] | sort: "date" | last %}
  {% assign tag_date = latest_post.date | date: "%Y%m%d" %}
  {% assign tag_entries = tag_entries | append: tag_date | append: ":" | append: tag[0] | append: "," %}
{% endfor %}
{% assign sorted_entries = tag_entries | split: "," | sort | reverse %}

<div class="tag-index">
  <p class="tag-cloud">
    {% for entry in sorted_entries %}
      {% if entry == "" %}{% continue %}{% endif %}
      {% assign tag_name = entry | split: ":" | slice: 1, 99 | join: ":" %}
      <a href="#{{ tag_name }}" class="tag-link">{{ tag_name }} ({{ site.tags[tag_name].size }})</a>
    {% endfor %}
  </p>
</div>

{% for entry in sorted_entries %}
  {% if entry == "" %}{% continue %}{% endif %}
  {% assign tag_name = entry | split: ":" | slice: 1, 99 | join: ":" %}
  {% assign tag_posts = site.tags[tag_name] | sort: "date" | reverse %}
<h2 id="{{ tag_name }}">{{ tag_name }} <span class="tag-count">({{ tag_posts.size }})</span></h2>
<ul class="tag-post-list">
  {% for post in tag_posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="dates">{{ post.date | date: "%b %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
{% endfor %}
