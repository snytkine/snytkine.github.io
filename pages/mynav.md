---
layout: page
title: Auto Nav
permalink: /nav/
---

# Documentation

Welcome to the {{ site.title }} Documentation pages! Here you can quickly jump to a 
particular page.

<div class="section-index">
    <hr class="panel-line">
    {% assign colls = site.collections | sort: "order" %}
    {% for coll in colls %} 
      {% if coll.nav_title %}       
        <div class="entry">
         <h5>~1~</h5>
         <p>label={{ coll.label }}</p>
         <p>nav_title={{ coll.nav_title }}</p>
         <p>order={{ coll.order }}</p>
         <div> 
         <p>ITEMS </p>
         {%  for doc in coll.docs %}
         <p>doc_title={{ doc.title }}</p>
         {%  endfor %}
         </div>
        </div>
      {% endif %}
    {% endfor %}
</div>
