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
      	<span class="date">{{ post.date | date_to_string }} &middot; <a href="{{ post.url }}">{% if post.note_type == 'link' %}Link{% elsif post.note_type == 'youtube' %}Video{% else %}Note{% endif %}</a></span>
      	{% if post.link_url %}{% if post.description %}<p class="description">{{ post.description }}</p>{% endif %}{% else %}<p class="description">{% if post.description %}{{ post.description }}{% else %}{{ post.content | strip_html | strip_newlines | truncate: 200 }}{% endif %}</p>{% endif %}
        {% if post.link_url %}
        <a href="{{ post.link_url }}" class="note-link-card" target="_blank" rel="noopener">
          {% if post.link_image %}<img src="{{ post.link_image }}" alt="" loading="lazy" onerror="this.remove()">{% endif %}
          <span class="note-link-info">
            <strong>{{ post.link_title }}</strong>
            {% if post.link_description %}<span class="note-link-desc">{{ post.link_description }}</span>{% endif %}
            <span class="note-link-domain">{{ post.link_domain }}</span>
          </span>
        </a>
        {% endif %}
      </li>
      {% endif %}
    {% endfor %}
  </ul>
</div>
