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
gallery:
  - image_path: /assets/images/Annotation 2019-03-12 100327.png
    alt: Create a flow
    title: ''
  - image_path: /assets/images/Annotation 2019-03-12 100438.png
    alt: Set to trigger when we get an email
    title: ''
---

Love it or hate it, a lot of work is still done through email. Keeping a log of emails and responses can give a massive edge.
Lets look at how we can automate the logging of all activity done on a theoretical support email alias.


## The Scenario

Our theoretical company, let's call them "Snap Inc"[^coincidence], sells luxury sunglasses that have a tendency to *break all the goddamn time*[^broke]. 

[^coincidence]: Any resemblance to actual companies or products is purely coincidental.
[^broke]: ![20180725_215647.jpg]({{site.imgurl}}/20180725_215647.jpg)

Customers can complete a form on the website which will generate an email to support@spectacles<nolink>.com for help. This will generate a random ticket number as well. Tickets are assigned by the last digit of the ticket number: Tim takes tickets ending in 0-3, Jayson in 4-6, and Wendy in 7-9.

All of the requestors can send email via the support alias.
When they respond, they use templated responses and include their name for the customer. A response might look something like this.

![Annotation 2019-03-13 120058.png]({{site.imgurl}}/Annotation 2019-03-13 120058.png)

Since they're sending from an alias, they respond with a Reply All thus bringing their sent response back to the inbox. This is so the other processors can check their work for auditing purposes.
  
They all periodically check the inbox and when they find a ticket ending in their digits, manually create a new log line for it. When they respond to the tickets, they'll update the Status accordingly. 
The basic log is a shared excel file, and currently looks like this.
  
![Annotation 2019-03-13 120030.png]({{site.imgurl}}/Annotation 2019-03-13 120030.png)

As you can imagine, there's a lot of room here for human error. One of the processors could miss an email and no log line would be created for it. They could misstype a log line. Finally, this just kinda feels like a waste of time doesn't it? It would be much better if the log tracked everything itself and just acted as a passive tracking solution.

---

Let's automate this log.
All together, a functional product will need to:

```
□ Create a new log row when a customer emails us.  
	□ Avoid creating duplicate tickets for the same customer. (Check if a ticket already exists)
  	□ Assign the ticket to the correct processor automatically.
  	□ Add the Email Date & TicketID info automatically.
□  Update the status of the ticket when we respond.  
	□ Update the Status field to Replied when an email is sent back.  
	□ Update the Status field to Completed when we send a "ticket is completed" email.  
	□ Update the Responder field with the name of who responded.
```


## Initial Setup

Let's begin by creating a new flow set to trigger whenever an email is received by our support inbox.

{% include gallery id="gallery" caption="" %}

You might notice all of our actions will boil down to either creating a new log line or updating an existing one. So, first we should check if the Case# in the subject line maps to an existing ticket.

First, let's create a few new fields in our log.
  
![Annotation 2019-03-13 122653.png]({{site.imgurl}}/Annotation 2019-03-13 122653.png)

The TicketID field will grab the ID from the subject line, making it easier to track things.


Now, let's build the logic for the TicketID field.

![Annotation 2019-03-13 123357.png]({{site.imgurl}}/Annotation 2019-03-13 123357.png)

This will create a new variable we can use throughout the flow. We'll grab the number from the subject using the following expression:
  
  ```
  substring(triggerBody()?['Subject'], indexOf(triggerBody()?['Subject'], '#'), 8)
  ```
  
 We know that the subject of every email will look something like `Update to Spectacles Support Case #27307612`, so this will find the character `#` and grab the 8 digit string that follows it.
  
  Next, let's check if we have a log line for this ticket already.
  We'll create a `Get a Row` request using the `Ticket Number` variable as a key. If there's no ticket yet, this will fail.
  
![Annotation 2019-03-13 124633.png]({{site.imgurl}}/Annotation 2019-03-13 124633.png)

 Now we can add two parallel steps, one to create a new row, and one to update an existing row.
  
![Annotation 2019-03-13 124911.png]({{site.imgurl}}/Annotation 2019-03-13 124911.png)

  And we'll set the `Add a row` to only run if `Get a Row` failed.
  
![Annotation 2019-03-13 125144.png]({{site.imgurl}}/Annotation 2019-03-13 125144.png)
![Annotation 2019-03-13 125208.png]({{site.imgurl}}/Annotation 2019-03-13 125208.png)


Nice! Now that we have the logic to create and update log lines, we'll need some logic to actually figure out what to put there.
  
![Annotation 2019-03-13 130759.png]({{site.imgurl}}/Annotation 2019-03-13 130759.png)
  
  
  ----
 
  Remember that when a processor responds to a ticket, they'll do so via Reply All thus sending their response back to the support Inbox, this way the other requestors can see how they're responding.
