---
layout: doc
title: Nimbus Tech Docs
subtitle: Guides and references from the Nimbus Tech team.
---

{% assign docs = site.pages | where: "layout", "doc" | sort: "title" %}
{% assign all_tags = docs | map: "tags" | compact | join: "," | split: "," | uniq | sort %}

<p class="tag-filter">
  <button type="button" class="tag is-active" data-tag="">All</button>
  {% for tag in all_tags %}<button type="button" class="tag" data-tag="{{ tag }}">{{ tag }}</button>{% endfor %}
</p>

<ul class="doc-list">
{% for doc in docs %}{% unless doc.url == page.url %}
  <li data-tags="{{ doc.tags | join: ' ' }}">
    <a href="{{ doc.url | relative_url }}">{{ doc.title }}</a>
    {% if doc.subtitle %}<p>{{ doc.subtitle }}</p>{% endif %}
    <p class="doc-list-meta">
      {% if doc.updated %}<span class="doc-list-date">Updated {{ doc.updated | date: "%B %-d, %Y" }}</span>{% endif %}
      {% for tag in doc.tags %}<span class="tag">{{ tag }}</span>{% endfor %}
    </p>
  </li>
{% endunless %}{% endfor %}
</ul>

<script>
  // Filter the list by tag. The chosen tag is kept in the URL (?tag=claude),
  // so tag links inside the docs open the home page already filtered.
  (function () {
    const buttons = document.querySelectorAll(".tag-filter button");
    const items = document.querySelectorAll(".doc-list li");
    function apply(tag) {
      buttons.forEach((b) => b.classList.toggle("is-active", b.dataset.tag === tag));
      items.forEach((li) => {
        li.hidden = tag !== "" && !li.dataset.tags.split(" ").includes(tag);
      });
    }
    buttons.forEach((b) =>
      b.addEventListener("click", () => {
        const url = new URL(location.href);
        if (b.dataset.tag) url.searchParams.set("tag", b.dataset.tag);
        else url.searchParams.delete("tag");
        history.replaceState(null, "", url);
        apply(b.dataset.tag);
      })
    );
    apply(new URLSearchParams(location.search).get("tag") || "");
  })();
</script>
