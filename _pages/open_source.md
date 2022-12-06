---
layout: page
permalink: /opensource/
title: open source
description: Datasets, repositories, and other contributions to open source projects.
nav: true
nav_order: 3
---

### RoboMal Dataset
A unique dataset of robot malware and good software. 

<div class="repo p-2 text-center">
  <a href="https://purr.purdue.edu/publications/3860/about?v=1">
    <img class="repo-img-light w-100" alt="{{ include.repository }}" src="https://purr.purdue.edu/publications/3860/about?v=1">
    <img class="repo-img-dark w-100" alt="{{ include.repository }}" src="https://github-readme-stats.vercel.app/api/pin/?username={{ repo_url.first }}&repo={{ repo_url.last }}&theme={{ site.repo_theme_dark }}&show_owner={{ show_owner }}">
  </a>
</div>

### Bovine FEM and Physiological Model Dataset
{% assign repo_url =  "https://github.com/SparcLab/Bovine-FEM-Model" %}
<div class="repo p-2 text-center">
  <a href="https://github.com/SparcLab/Bovine-FEM-Model">
    <img class="repo-img-light w-100" src="https://github-readme-stats.vercel.app/api/pin/?username={{SparcLab}}&repo={{ Bovine-FEM-Model }}&theme={{ site.repo_theme_light }}">
    <img class="repo-img-dark w-100" src="https://github-readme-stats.vercel.app/api/pin/?username={{ repo_url.first }}&repo={{ repo_url.last }}&theme={{ site.repo_theme_dark }}">
  </a>
</div>

### DESK Dataset
<div class="repo p-2 text-center">
  <a href="https://github.com/{{ include.repository }}">
    <img class="repo-img-light w-100" src="https://github-readme-stats.vercel.app/api/pin/?username={{ repo_url.first }}&repo={{ repo_url.last }}&theme={{ site.repo_theme_light }}">
    <img class="repo-img-dark w-100" src="https://github-readme-stats.vercel.app/api/pin/?username={{ repo_url.first }}&repo={{ repo_url.last }}&theme={{ site.repo_theme_dark }}">
  </a>
</div>

