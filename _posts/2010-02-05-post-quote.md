---
title: "Quotes I like"
permalink: quotes #unique word that will be used as url basesite.com/[word]
toc: false
share: false
author: false
header:
  #overlay_image: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
  teaser: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
categories:
#  - Guides
#  -
#  - Uncategorized
tags:
  - edge case
  - featured image
  - image
  - layout

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
