---
layout: default
title: Topics
permalink: /topics/
---

{%- for cat in site.categories -%}
{%- if site.allowedcats contains cat[0]  -%}

<h2>{{cat[0] | capitalize }}</h2>
<ul>
{%- for post in cat[1] -%}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{%- endfor -%}
</ul>
{% endif %}
{%- endfor -%}
