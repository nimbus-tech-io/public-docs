---
layout: doc
title: Changelog
subtitle: What changed in the docs, with the important updates marked.
updated: 2026-10-13
author: Milos
tags: [meta]
---

{% assign repo = "https://github.com/nimbus-tech-io/public-docs/commit/" -%}
{% comment %}Newest date first. Entries with the same date keep their order from the data file.{% endcomment -%}
{% assign days = site.data.changelog | group_by: "date" | sort: "name" | reverse -%}
{% for day in days -%}
{% for e in day.items -%}
<section class="entry{% if e.level == 'important' %} entry-important{% endif %}">
<p class="entry-head"><span class="entry-level">{% if e.level == 'important' %}Important{% else %}Update{% endif %}</span> <span class="entry-date">{{ e.date | date: "%B %-d, %Y" }}</span></p>
<h2>{{ e.title }}</h2>
{{ e.summary | markdownify }}
{%- if e.diff %}
<pre class="diff"><code>{% assign lines = e.diff | split: "
" %}{% for l in lines %}{% if l == "" %}{% continue %}{% endif %}{% assign c = l | slice: 0 %}<span class="{% if c == '+' %}diff-add{% elsif c == '-' %}diff-del{% endif %}">{{ l | escape }}</span>
{% endfor %}</code></pre>
{%- endif %}
{%- if e.docs or e.commit %}
<p class="entry-links">
{%- for d in e.docs -%}
{%- assign p = site.pages | where: "name", d | first -%}
{%- if p %}<a href="{{ p.url | relative_url }}">{{ p.title }}</a>{% else %}<span>{{ d }}</span>{% endif %}
{%- endfor -%}
{%- if e.commit -%}
{%- assign cs = e.commit | slice: 0, 4 -%}
<a href="{% if cs == 'http' %}{{ e.commit }}{% else %}{{ repo }}{{ e.commit }}{% endif %}">Commit{% unless cs == 'http' %} {{ e.commit }}{% endunless %}</a>
{%- endif -%}
</p>
{%- endif %}
</section>
{% endfor %}
{% endfor %}
