---
layout: page
title: dazz.github.io
tagline: "the code log"
---
{% include JB/setup %}

<ul class="posts unstyled">
  {% for post in site.posts %}
    <li>
        <h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }} <small>{{ post.date | date_to_long_string }}</small></a></h3>
        {% if post.excerpt %}
            &raquo; {{ post.excerpt }} ... <span class="label label-important"><a href="{{ BASE_PATH }}{{ post.url }}">read more</a></span>
        {% endif %}
    </li>
  {% endfor %}
</ul>

