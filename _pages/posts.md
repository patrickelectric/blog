---
layout: archive
title: "Posts"
permalink: /
author_profile: true
---
{% include base_path %}

{% capture written_year %}'None'{% endcapture %}

{% for post in site.posts %}
  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
  {% if year != written_year %}
<h2 id="{{ year | slugify }}" class="archive_subtitle">{{ year }}</h2>
    {% capture written_year %}{{ year }}{% endcapture %}
  {% endif %}
  {% include archive-i-single.html %}
{% endfor %}
