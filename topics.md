---
layout: default
title: Topics
permalink: /topics/
---

{%- for cat in site.categories -%}

<h2>{{cat[0] | capitalize }}</h2>
<ul>
	{%- for post in cat[1] -%}
		<li><a href="{{ post.url }}">{{ post.title }}</a></li>
	{%- endfor -%}
</ul>
{%- endfor -%}
