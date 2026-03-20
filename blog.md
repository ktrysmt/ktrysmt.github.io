---
permalink: /blog/
profile: false
layout: list
description: "SREとかセキュリティガバナンスとか"
---

<ul id="post-list">
    {% for post in site.posts %}
        <li>
            <a href="/{{ post.url | remove_first: '/' }}"><aside class="dates">{{ post.date | date:"%b %d, %Y" }}</aside></a>
            <a href="/{{ post.url | remove_first: '/' }}">{{ post.title }}</a>
            {% if post.description %}<p class="excerpt">{{ post.description | truncate: 120 }}</p>{% endif %}
        </li>
    {% endfor %}
</ul>

