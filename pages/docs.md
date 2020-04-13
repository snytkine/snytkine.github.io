---
layout: page
title: Documentation
permalink: /docs/
---

# Documentation

Welcome to the {{ site.title }} Documentation pages! Here you can quickly jump to a 
particular page.

<div class="section-index">
    <hr class="panel-line">
    {% for post in site.docs  %}        
    <div class="entry">
    <h5><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></h5>
    <p>{{ post.description }}</p>
    <span>relative_path = {{ post.relative_path }}</span>
    <span>collection = {{ post.collection }}</span>
    <span>date = {{ post.date }}</span>
    <i>{{ post.nav_title}}</i>
    </div>
    {% endfor %}
</div>
