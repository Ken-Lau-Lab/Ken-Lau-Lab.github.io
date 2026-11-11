---
layout: page
title: About # the navbar label is taken from this title; the visible heading is written in the body below
permalink: /
# The `page` layout is used instead of `about` so that latest posts can sit beside the profile image
# (the `about` layout wraps the body in a clearfix, which forces everything below the floated image).
# Posts AND any page that has a `date:` in its front matter (e.g. a new SOP) are listed under "latest posts".
_styles: >
  .home-header .post-description { font-size: 1.5rem; font-weight: 700; }
  .post-header { display: none; }
  .home-grid { display: flex; gap: 2rem; align-items: flex-start; }
  .home-main { flex: 1 1 auto; min-width: 0; }
  .home-profile { flex: 0 0 30%; }
  .home-profile img { width: 100%; height: auto; }
  .home-profile .more-info { margin-top: 5px; font-family: monospace; }
  .home-profile .more-info p { margin: 0; }
  @media (max-width: 575px) { .home-grid { flex-direction: column-reverse; } .home-profile { flex-basis: auto; width: 100%; } }
---

{% assign dated_pages = site.pages | where_exp: "p", "p.date" %}
{% assign recent_items = site.posts | concat: dated_pages | sort: "date" | reverse %}

<div class="home-header">
<h1 class="post-title"><strong>Ken Lau Lab<strong></h1>
<p class="post-description"><strong>Softwares, Tutorials, and SOPs from the Lau Lab</strong></p>
</div>
<div class="home-grid">
<div class="home-main">
<h2><a href="{{ '/blog/' | relative_url }}" style="color: inherit">Latest Posts</a></h2>
<div class="news"><div class="table-responsive"><table class="table table-sm table-borderless">
{% for item in recent_items limit: 3 %}
<tr><th scope="row" style="width: 20%">{{ item.date | date: '%b %d, %Y' }}</th><td><a class="news-title" href="{{ item.url | relative_url }}">{{ item.title }}</a></td></tr>
{% endfor %}
</table></div></div>
</div>
<aside class="home-profile">
<img class="img-fluid z-depth-1 rounded" src="{{ '/assets/img/laulab.png' | relative_url }}" alt="Lau Lab">
<div class="more-info">
<p>MRBIII 9271</p>
<p>465 21st Ave S</p>
<p>Nashville, TN 37232</p>
<p>Lab Website: <a href="https://kenlaulab.org/">kenlaulab.org</a></p>
</div>
</aside>
</div>
