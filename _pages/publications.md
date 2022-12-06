---
layout: page
permalink: /publications/
title: publications
description: publications by categories in reversed chronological order.
years: [2022,2021,2020]
type: [Book Chapters,Conferences and Workshops,Journals]
nav: true
nav_order: 1
---
<!-- _pages/publications.md -->
<div class="publications">

{%- for y in page.type %}
  <h2 class="type">{{y}}</h2>
    {%- for yy in page.years %}
      {% bibliography -f papers -q @*[year={{yy}}]* %}
    {% endfor %}
      {% bibliography -f papers -q @*[type={{y}}]* %}
{% endfor %}
{% endfor %}
</div>
