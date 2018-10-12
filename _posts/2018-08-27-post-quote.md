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

<a href=" " title="This is some text I want to display." style="background-color:#FFFFFF;color:#000000;text-decoration:none">This link has mouseover text.</a>

this is a <a class="thumbnail" href="#">hoverlink<span><img src="https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg" alt="Little Egret"><br> Little Egret</span></a>
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
