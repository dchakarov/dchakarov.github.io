---
title: Apps
layout: default
permalink: /apps/
---

<div id="apps-index">
  <h1 class="pageTitle">Apps</h1>
  <p class="intro">Things I've built and shipped.</p>

  <ul class="app-grid noList">
    {% assign apps = site.apps | sort: 'order' %}
    {% for app in apps %}
      {% unless app.hidden %}
      <li>
        <a href="{{ app.url | prepend: site.baseurl }}" class="app-card">
          {% if app.icon %}<img src="{{ app.icon | prepend: site.baseurl }}" alt="" loading="lazy" onerror="this.remove()">{% endif %}
          <span class="app-card-info">
            <strong>{{ app.name }}</strong>
            {% if app.tagline %}<span class="app-card-tagline">{{ app.tagline }}</span>{% endif %}
            <span class="app-card-meta">
              {% if app.status == 'coming_soon' %}
                <span class="status-badge coming-soon">Coming Soon</span>
              {% elsif app.status == 'beta' %}
                <span class="status-badge beta">Beta</span>
              {% elsif app.status == 'released' %}
                <span class="status-badge released">Available</span>
              {% endif %}
              {% if app.platforms %}
                {% for platform in app.platforms %}<span class="platform-pill">{{ platform }}</span>{% endfor %}
              {% endif %}
            </span>
          </span>
        </a>
      </li>
      {% endunless %}
    {% endfor %}
  </ul>
</div>
