---
title: Notes
layout: default
---

<div id="articles">
  <h1>Notes</h1>
  <ul class="posts noList">
    {% for post in site.posts %}
      {% if post.categories contains 'notes' %}
      <li>
      	<span class="date">{{ post.date | date_to_string }} &middot; <a href="{{ post.url }}">Note</a></span>
      	<p class="description">{{ post.content | strip_html | strip_newlines | truncate: 200 }}</p>
      </li>
      {% endif %}
    {% endfor %}
  </ul>
</div>
