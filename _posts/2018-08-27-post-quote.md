---
title: "Quotes I like"
permalink: quotes #unique word that will be used as url basesite.com/[word]
toc: false
share: false
author: false
header:
  #image: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
  teaser: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
categories:
#  - Guides
#  -
  - Uncategorized
tags:
#  - edge case


---
This page serves as a log of quotes I've found that left some lasting impression.


this is a <a class="thumbnail">hoverlink<span><img src="https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg" alt="Little Egret"><br> Little Egret</span></a>
and this is a [normal link](https://google.com)


{% for item in site.data.quotes %}
> {{item.quote}}
<br>
<small><cite>
> {{item.from}}
<br>
  Added {{item.dateAdded}}
</cite></small>
{% endfor %}
