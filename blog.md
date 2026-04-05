---
name: ktrysmt.log
title: ktrysmt.log
layout: list
permalink: /blog/
description: "SREとかセキュリティガバナンスとか"
---

{% assign category_order = "AI Engineering,Cloud Infrastructure,Developer Tools,Engineering Practices,Programming,Security Governance" | split: "," %}
<div class="category-filter">
  <a href="#" class="category-filter-link active" data-category="all">All ({{ site.posts.size }})</a>
  {% for cat_name in category_order %}
    {% if site.categories[cat_name] %}
      <a href="#" class="category-filter-link" data-category="{{ cat_name }}">{{ cat_name }} ({{ site.categories[cat_name].size }})</a>
    {% endif %}
  {% endfor %}
</div>

<ul id="post-list">
    {% for post in site.posts %}
        <li data-category="{{ post.categories | first }}">
            <a href="/{{ post.url | remove_first: '/' }}">{{ post.title }}</a>
            <div class="post-meta-row">
                <a href="#" class="post-category-link" data-category="{{ post.categories | first }}">{{ post.categories | first }}</a>
                <span class="dates">{{ post.date | date:"%b %d, %Y" }}</span>
            </div>
        </li>
    {% endfor %}
</ul>

<script>
document.addEventListener("DOMContentLoaded", function() {
  var filterLinks = document.querySelectorAll(".category-filter-link");
  var categoryLinks = document.querySelectorAll(".post-category-link");
  var items = document.querySelectorAll("#post-list li");

  function filterBy(category) {
    filterLinks.forEach(function(l) {
      l.classList.toggle("active", l.dataset.category === category);
    });
    var first = true;
    items.forEach(function(li) {
      var visible = category === "all" || li.dataset.category === category;
      li.style.display = visible ? "" : "none";
      li.classList.remove("first-visible");
      if (visible && first) {
        li.classList.add("first-visible");
        first = false;
      }
    });
    if (category !== "all") {
      history.replaceState(null, "", "#" + encodeURIComponent(category));
    } else {
      history.replaceState(null, "", window.location.pathname);
    }
  }

  filterLinks.forEach(function(link) {
    link.addEventListener("click", function(e) {
      e.preventDefault();
      filterBy(this.dataset.category);
    });
  });

  categoryLinks.forEach(function(link) {
    link.addEventListener("click", function(e) {
      e.preventDefault();
      filterBy(this.dataset.category);
      window.scrollTo({ top: 0, behavior: "smooth" });
    });
  });

  var hash = decodeURIComponent(window.location.hash.slice(1));
  filterBy(hash || "all");
});
</script>

