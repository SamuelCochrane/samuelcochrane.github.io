---
color: warning
categories: ''
title: How to connect PowerBI to a Tabular API (and why you shouldn't)
description: ''
style: ''
image: ''
tags: []
toc: false

---

***

Before we start, I'll be basing this on the [Clickup API](https://clickup.com/api "Clickup API") (their API documentation is actually pretty good, you should give it a check!).  
Clickup is a project management tool, similar to Monday.com and Azure DevOps.   
Nothing in this document is all that specific to Clickup however, so the contents should _mostly_ hold true for whatever other REST API you're looking at.  
   
The basics: getting data

Let's create a blank query in PowerBI

        let
            Source = Json.Document(Web.Contents(ClickUpBaseURL,
                [RelativePath="team/" & "10631688" & "/task",
                Query=[include_closed="TRUE", 
                       subtasks="TRUE", 
                       page=Number.ToText(PageNo)],
                Headers=[Authorization=ClickUpAuthHeader]]))
        in 
            Source

test