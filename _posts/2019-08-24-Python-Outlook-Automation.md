---
title: "Automating Email Reports with Python"
#permalink: PythonOutlookAutomation #unique word that will be used as url basesite.com/[word]
header:
  teaser: https://farm5.staticflickr.com/4140/4939863887_84705982fd_b.jpg
  #overlay_image: assets/images/[thing]
  overlay_filter: 0.3
  show_overlay_excerpt: false
categories:
#  - Guides#  - Guides

#  - Portfolio
#  - Uncategorized
tags:
  - Python
  - Automation
  - VBA
  - Outlook
  - Excel
  # - Visualization
#toc: true
published: false
---

One common task I've seen as part of an analysts responsibilities is to send out a daily status email.
The email usually looks something like this.


PZ image of outlook email

Many execs find these emails useful as a quick way to stay up to date on the financial numbers of a company.
That said, sending an email on a daily cadence is a task primed for automation, let's see how to do it.

We can break this task down into a few component parts.

```
□ open an excel report
□ refresh the numbers in the excel Report
□ take screenshots of the relevant numbers
□ create a new email from a template (maybe by opening & editing yesterday's report)
□ populate the email with relevant contextual information
  □ today's date
  □ specific numbers from the report
```

As we're interfacing with two Office products (Excel and Outlook), VBA would be a good language to use to script these interactions.
That said, VBA has some limitations. It has to live as an extension to one of the products instead of a standalone file, it can't be set to automatically run on a cadence, and overall it's just a very limited language.

Therefore, I've decided to leverage Python to tackle this task by using a library called win32com that let's us write VBA-esque code in Python.


## Opening & Refreshing Excel

```
import win32com.client as win32
xl = win32.Dispatch("Excel.Application")
```
This code gives us a direct reference to an excel file we have open.[^reference]

[^reference]: If you have multiple excel reports open at once, this code will reference whichever report you've interacted with most recently.

Now, we can directly mess with things in Excel via code. Try it out!
```
xl.Columns.ColumnWidth = 10             #set every column to width 10
xl.Cells(2,1).Value                     #grab the value of cell A2 (row 2 column 1) in the current sheet
xl.Cells(5,3).Interior.ColorIndex = 8   #set the color of cell C5 (row 5 column 3) to cyan.
```

PZ how to refresh a pivot table in a report

## Saving pictures

Now we need to get pictures of the updated data for our email.

```
xl.Range("A1:H5").Copy()                             #copy range onto clipboard as a picture
ImageGrab.grabclipboard().save('paste1.png', 'PNG')  #save that picture in the current working directory
```
PZ image of the image saved in a file

## Populating the email

Now that we have everything we need, let's generate that email.

```
outlook = win32.Dispatch('outlook.application')   #get a reference to Outlook
mail = outlook.CreateItem(0)                      #create a new mail item
mail.To = 'executives@bigcompany.com'
mail.Subject = 'Finance Status Report '+datetime.today().strftime('%m/%d')   #put today's date in subject line

```

As for the body of the report, we will pass in HTML code using `mail.HTMLBody =` with the following.

```
<p>Hi Team,</p>

<p>This email is to provide a status of the our current sales numbers</p>


<img src='C:\\Users\\sam\\Desktop\\EmailAuto\\paste1.png'>


<img src='C:\\Users\\sam\\Desktop\\EmailAuto\\paste2.png'>


<p>Thanks and have a great day!</p>
```


finally, we can make this mail item visible with a `mail.Display()`.
(We could also just send it with a `mail.Send()`, but I've found people usually prefer to give the email a once-over before sending it out).

PZ image of the generated email

And that's it! With a script like this I've generally been able to save ~15 minutes of time per email, not to mention the reduced chances for human error.

<small>desc.</small>

<hr>


this is a <a class="thumbnail">hoverlink<span><img src="{{site.url}}{{site.baseurl}}/assets/reactionimages/mindblown.gif"><br></span></a>

and this is a [normal link](https://google.com)


and this is an image
![Image Name]({{site.url}}{{site.baseurl}}/assets/images/picfix_welcome.png)
![database ERD]({{site.url}}{{site.baseurl}}/assets/images/445_fullErd.PNG)

> This is a quote
<br>
<small><cite>
> from a guy
</cite></small>

## title

### sub title



```
code block
```

and `inline` code

Two image figure

<figure class="half">

<img src="../assets/images/Annotation 2019-03-12 100327.png">
<img src="../assets/images/Annotation 2019-03-12 100438.png">
<figcaption>caption </figcaption>
</figure>


Colons can be used to align columns.

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

There must be at least 3 dashes separating each header cell.
The outer pipes (|) are optional, and you don't need to make the
raw Markdown line up prettily. You can also use inline Markdown.

Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3
