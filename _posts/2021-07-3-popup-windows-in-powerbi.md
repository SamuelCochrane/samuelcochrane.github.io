---
title: How to build a toggle-able window in PowerBI (even though that's not a feature)
# toc: false
style: border #fill, border
color: info #primary / secondary / success / danger / warning / info / light / dark (choose one only)
image: https://remnote-user-data.s3.amazonaws.com/x4aWaP4M69AxYfnmxF0EqJvbd2S-ZLwct9l6DkN5x-QO5pwQW3LhUbLsXO7b5WVtrB4Z4G2nnMp03JW1HzwqFFiwZMp7DT1nP8HQrMcSpWohVhjSTzO3E3De7ulKOubv.gif
categories:
  - Portfolio
tags: 
  - PowerBI
  - Hacks
  #  - Machine learning
  #  - SciKit
  #  - Python
  #  - Deep learning
  #  - Jupyter
published: true
---


One common UX issue I've had when building PowerBI reports is that stakeholders often want  __a lot__  of slicers.
It makes sense, having a large amount of control over their data is a reasonable request. The only problem is that it tends to visually overload the report.
![](https://remnote-user-data.s3.amazonaws.com/z67Usp8yAQNq2LzqVJmvL9AgJew82lgJ46rg8Hwc1pg5wg-UH9d0ODPSVut0q3SLdqu7KNfNL9VQz5VXoKsfzQbHwbIFltRTgQ1NDX-wTUdmNLxBrslrKBhvgA1eph7h.png)
It starts to look visually heavy very fast. In the worst-case scenario, slicers could end up taking up nearly half of your screen real-estate.
Usually, you'd hide all these options behind a collapsible/toggleable window.

![](https://d1r27dnp1fh4g5.cloudfront.net/i/tutorial/sidebar2-animated.gif)
Unfortunately, PBI doesn't have such functionality. There's no built-in way to to create expandable menu's or modal windows.

But I found a workaround. Here's how to hack it.

### Creating our groups

First, go to View->Selection to open the Selection window. This window shows you all the elements on your page, and in what order they're shown in (things lower on the list will be shown behind those higher on the list).

![](https://remnote-user-data.s3.amazonaws.com/5BEPGxGIGusUsL2x43UUUAdS1NytqBCpXuG1xxUOurLhDFYGNzCMhDcXiszNkfKa_1G6Z-eESqSAesStXPLME3wfSipJFANEFo5aY0ZOo-l42PBzI-fBqIvuYYkW0SHC.png)
In this window, select your slicers, right click, and create a group.

With the new group, you'll see that you can toggle the visibility of all of them at once by changing the visibility of the group.
At this step, you'll also want to add a box and place it behind the slicers. This will be the "window".

![](https://remnote-user-data.s3.amazonaws.com/6VLYqe7Bq1LdTxQtwLhQVl-c7zqc8OfAB4K_l8pHdptJ2ZeLaXnXQB6_kQNvQ8QU-rRUT9p66m2hyNdgrTd0KyxBrODznaEqIZLDJRRRfFetf5Y0mFy_cvIV7g8b9CcG.png)


While we're here, let's also add two buttons. A "hide" button and a "show" button. These can stay outside our group.
![](https://remnote-user-data.s3.amazonaws.com/OJPFVuZvnSC0OMuY-w5GKzh3WYqCqT1W8ifWx3fmraYtWjk_ESY7eZHgVd1J9FCKpuBe7phJe3XH3o_meS0-qgSubHEtzPL88GbXH1JMRHO_lTjfwVCMaUmMxIRCpIwY.png)
Our Selection window should now look like this:
![](https://remnote-user-data.s3.amazonaws.com/dYiGvING-kXaGxb7AD0-4-PUzhVoJEkYQiOdpozPnUJhNa7Z3dNHKPykiTEtDqtYk4UcnGld4Je_gLOlY9FzPju9ksXwqVtV6oFsqbU03RLXnJ8ozmILNM-CdVuQIc6U.png)

Our hierarchy is:
- Hide button
- Group
    - All the slicers
    - our window background (a square shape)
- Show button

### Creating our bookmarks

Next, we'll configure the functionality of the two buttons. Each one should show or hide this group respectively.
In the top ribbon, go to the View->Bookmarks tab and create two bookmarks.


{% include elements/highlight.html text='In PowerBI, a Bookmark is a shortcut to a certain "state" the report is in. 
This includes selections and visibility. <br>
You can use it to quickly link users to specific things you want them to see. E.g., you could have a <i>"bicycle sales for the last fiscal month in Arkansas"</i> bookmark that sets the values of the product, date, and geolocation slicers.' %}


The Show Slicers bookmark should:
  - show the group
  - show the Hide button
  - hide the Show button

The Hide Slicers bookmark should do the inverse:
  - hide the group
  - hide the Hide button
  - show the Show button


For both bookmarks, right click and ensure that it's only bookmarking the Display and page. Data should be unchecked.
![](https://remnote-user-data.s3.amazonaws.com/jOYGLDjWPN8CCv4UJ6crztkwAFL3Imd3PYMvRB4Zw0cEKNf4HaBi92ZkvZNUiUJDkZDT-y-qP3RFQKS51Bc0KIIvRwfMwOWVxTsfYg97tc4c7JUP-6-Nv7MZ4-YOkdt3.png)

### Creating the functionality
Finally, link the buttons to the bookmarks. When we click the button, it will send us to this Bookmark, which controls the visibility of the slicer window (and hides itself and shows the other button).

![](https://remnote-user-data.s3.amazonaws.com/Lud6iQYwmW2egzJySgMMJzWKh8P1eH1FJFzQ0oOLjxKOSZoO07EszXed1kUKewvA5LUIxTaOlA-97Sj-uMPvNYgTGlSS8WJCLTonpgUKyI9yrfmNgGpIINepDWu6iGTN.png)

And here's our end result: a "window" that we can show and hide to save valuable screen space and keep our reports looking clean.

![](https://remnote-user-data.s3.amazonaws.com/x4aWaP4M69AxYfnmxF0EqJvbd2S-ZLwct9l6DkN5x-QO5pwQW3LhUbLsXO7b5WVtrB4Z4G2nnMp03JW1HzwqFFiwZMp7DT1nP8HQrMcSpWohVhjSTzO3E3De7ulKOubv.gif)
