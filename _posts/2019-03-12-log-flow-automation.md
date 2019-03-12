---
title: Automating an email log with MS Flow
permalink: logflow
header:
  overlay_image: assets/images/golf.PNG
  overlay_filter: 0.3
  show_overlay_excerpt: false
  teaser: assets/images/golf.PNG
categories:
  - Portfolio
tags:
  - Automation
  - MS Flow
  - Outlook
published: true
---

Love it or hate it, a lot of work is still done through email. Keeping a log of emails and responses can give a massive edge.
Lets look at how we can automate the logging of all activity done on a theoretical support email alias.

###The Scenario
Our theoretical company, let's call them "Snap Inc" [^1], sells luxury sunglasses that have a tendency to *break all the goddamn time* [^2]. Customers can email support@VeryCoolSunglasses<nolink>.com for help. Each pair of glasses comes with a unique ID the customer will need to provide.
[^1]:  Any resemblance to actual companies or products is purely coincidental.
[^2]: ![PowerBI Viz](/assets/images/golf.PNG)

<hr>

All together, a functional product will need to:
* [ ] Create a new log row when a customer emails us.
  * [ ] Avoid creating duplicate tickets for the same customer. (Check if a ticket already exists)
* [ ] Update the status of the ticket when we respond.
  * [ ] Update the `Status` field to `Replied` when an email is sent back.
  * [ ] Update the `Status` field to `Completed` when we send a "ticket is completed" email.
  * [ ] Update the `Responder` field with the name of who responded.



### Initial Setup

<small>Designed & built an interactive dashboard for a golf tourney. Allows golfers to see their stats & compare against the other golfers.<br>Built using PowerBI for the MSIT Give Golf Tournament.</small>
<hr>

Launch Consulting would be sponsoring one of the holes for the annual MSIT Give Golf Tournament, an event Microsoft hosts for charity, and Launch wanted to do something unique. They partnered with <a href="https://trackmangolf.com/" target="_blank">Trackman</a>, a company that designs hardware that can track golf swing data.
<br>These devices are usually used by pro golfers to do complex analysis on their swings, but here every golfer that came by the hole 16 would be able to see their swing data for free.

![PowerBI Viz]({{site.url}}{{site.baseurl}}/assets/images/golf.PNG)

This dashboard was designed to let the golfers view their swing data (ball speed, distance, etc) after the tournament, and compare how they & their team did to others.
<br> This was a big hit at the tournament, and many couldn't wait to get online after it ended to start bragging to their coworkers about how they _"hit the ball 20MPH faster than James in accounting"_ (or at least something along those lines).


<hr>
The dashboard is available at [http://launchcg.com/golf](https://launchcg.com/golf/), and can be viewed along with a well-produced video & some other jump-out statistics produced by other members of the team.
