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


### The Scenario

Our theoretical company, let's call them "Snap Inc"[^coincidence], sells luxury sunglasses that have a tendency to *break all the goddamn time*[^broke]. Customers can email support@VeryCoolSunglasses<nolink>.com for help. Each pair of glasses comes with a unique ID the customer will need to provide.
  
[^coincidence]: Any resemblance to actual companies or products is purely coincidental.
[^broke]: ![20180725_215647.jpg]({{site.imgurl}}/20180725_215647.jpg)

  
<hr>

All together, a functional product will need to:
- [ ]  Create a new log row when a customer emails us.  
	- [ ]  Avoid creating duplicate tickets for the same customer. (Check if a ticket already exists)  
- [ ]  Update the status of the ticket when we respond.  
	- [ ]   Update the `Status` field to `Replied` when an email is sent back.  
	- [ ]   Update the `Status` field to `Completed` when we send a "ticket is completed" email.  
	- [ ]   Update the `Responder` field with the name of who responded. 

### Initial Setup

![Annotation 2019-03-12 100327.png]({{site.imgurl}}/Annotation 2019-03-12 100327.png)
Let's begin by 

