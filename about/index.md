---
layout: layouts/post.njk
title: About Me
tags:
  - nav
navtitle: About
templateClass: tmpl-post
---

OUTPAW stands for One Useless Thing Per a Week.

{% for section in sections %}
## {{ section.title }}
{% for entry in section.entries %}
{% if entry.title -%}
{{ entry.title }}
{% if entry.description %}: _{{ entry.description }}_{% if entry.period %} &horbar; {{ entry.period }}{% endif %}{% endif -%}
{% if entry.score %}: {% for i in range(5) %}{% if i < entry.score %}&starf;{% else %}&star;{% endif %}{% endfor %}{% endif %}
{%- for item in entry.items %}
  * {{ item }}
{%- endfor %}
{% else -%}
* {{ entry | safe }}
{% endif %}
{% endfor %}
{% endfor %}
