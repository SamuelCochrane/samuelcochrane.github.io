---
title: "Quotes I like"
permalink: quotes #unique word that will be used as url basesite.com/[word]
toc: false
share: false
author: false
header:
  overlay_image: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
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


> A river cuts through rock, not because of its power but because of its persistence.
><br> <cite>
- Jim Watkins
<br><small>Added 08/27/18</small>
</cite>

>For all of the most important things, the timing always sucks. Waiting for a good time to quit your job? The stars will never align and the traffic lights of life will never all be green at the same time. The universe doesn't conspire against you, but it doesn't go out of its way to line up the pins either. Conditions are never perfect. "Someday" is a disease that will take your dreams to the grave with you. Pro and con lists are just as bad. If it's important to you and you want to do it "eventually," just do it and correct course along the way.
><br> <cite>
- Timothy Ferriss, The 4-Hour Workweek
<br><small>Added 08/27/18</small>
</cite>


<ul>
{% for quote in site.data.quotes %}
  <li>
    > {{quote.quote}}
    <br> <cite>
    - {{quote.name}}
    <br><small>{{quote.dateAdded}}</small>
    </cite>
  </li>
{% endfor %}
</ul>
