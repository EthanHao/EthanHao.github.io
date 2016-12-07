---
layout: default
---

{% for project in site.portfolio limit:3 %}
      <li>
        <img src="{{ project.thumbnail-path }}" alt="{{ project.title }}"/>
        <a href="{{ project.url }}">{{ project.title }}</a>
        <p>{{ project.short-description }}</p>
      </li>
{% endfor %}


 