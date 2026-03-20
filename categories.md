---
title: Categories
permalink: /categories/
layout: list
description: ""
---

{% assign category_order = "Security Governance,AI Engineering,Cloud Infrastructure,Developer Tools,Engineering Practices,Programming" | split: "," %}

<div class="tag-index">
  <p class="tag-cloud">
    {% for cat_name in category_order %}
      {% if site.categories[cat_name] %}
        <a href="#{{ cat_name }}" class="tag-link">{{ cat_name }} ({{ site.categories[cat_name].size }})</a>
      {% endif %}
    {% endfor %}
  </p>
</div>

{% for cat_name in category_order %}
  {% unless site.categories[cat_name] %}{% continue %}{% endunless %}
  {% assign cat_posts = site.categories[cat_name] | sort: "date" | reverse %}
<h2 id="{{ cat_name }}">{{ cat_name }} <span class="tag-count">({{ cat_posts.size }})</span></h2>
<ul class="tag-post-list">
  {% for post in cat_posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="dates">{{ post.date | date: "%b %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
{% endfor %}

