---
title: Portfolio
permalink: /portfolio/
layout: collection
collection: portfolio
entries_layout: cards
include_date: false
---

If you are looking at this page, you might also be interested in [my CV]({{ site.baseurl }}/assets/docs/CV_Delort_Tristan.pdf)

<div class="cards">
    {% for project in site.portfolio %}
    <div class="page">
        <a href="{{ project.url | relative_url }}">
            <figure>
                <img src="{{ project.header.teaser }}" alt="{{ project.title }} teaser image" />
            </figure>
            <span class="page-name">{{ project.title }}</span>
            <br/>
            <span class="page-desc">> {{ project.excerpt }}</span>
        </a>
    </div>
    {% endfor %}
</div>