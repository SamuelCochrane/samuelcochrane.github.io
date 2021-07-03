---
title: Quotes I like
permalink: quotes
style: border #fill, border
color: success #primary / secondary / success / danger / warning / info / light / dark (choose one only)
toc: false
share: false
author: false
header:
  teaser: 'https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg'
categories:
  - Uncategorized
tags: null
published: false
---
This page serves as a log of quotes I've found that left some lasting impression.

{% for item in site.data.quotes %}
> {{item.quote}}
<br>
<small><cite>
> {{item.from}}
<br>
  Added {{item.dateAdded}}
</cite></small>
{% endfor %}
