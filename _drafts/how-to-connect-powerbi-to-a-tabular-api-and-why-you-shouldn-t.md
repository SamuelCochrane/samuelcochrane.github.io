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
Before we start, I'll be basing this on the [Clickup API](https://clickup.com/api "Clickup API") (their API documentation is actually pretty good, you should give it a check!).  
Clickup is a project management tool, similar to Monday.com and Azure DevOps.  
{% include elements/highlight.html text="Nothing in this document is all that specific to Clickup"%}, so the contents should _mostly_ hold true for whatever other REST API you're looking at.  
The data we're looking at will contain things like tasks, assigned people, tags, due-dates, etc.

***

I say "why you shouldn't" because, as we'll learn, doing a direct connection like this is a series of problem-solving activities based around overcoming the inherent limitations of PowerBI.

## The basics: getting data

Let's open up our query editor and set up a view helper parameters (variables).  
![](assets/images/screenshot-2022-01-29-230305.jpg)

Here we can create variables just like in many other programming languages.  
I'm going to create two parameters:

\`

one for storing the team ID and one for storing the API Key (both of which are private)

Let's create a blank query in PowerBI

        let
            Source = Json.Document(Web.Contents("https://api.clickup.com/api/v2",
                [RelativePath="team/" & "10631688" & "/task",
                Query=[include_closed="TRUE", 
                       subtasks="TRUE", 
                       page=Number.ToText(PageNo)],
                Headers=[Authorization=ClickUpAuthHeader]]))
        in 
            Source

### PowerBI Doesn't Support Pagination

### PowerBI Doesn't Support UNIX Timestamps

Checking the data returned from the api, I immediately noticed something odd. All of the time (start date, due date, ...) was being given as an integer value (like 1641782153786).

PZ image

This is called a UNIX Timestamp, and represents the number of seconds since January 1st, 1970 (when time started existing).

There are some \[great tools\]([https://www.unixtimestamp.com/](https://www.unixtimestamp.com/ "https://www.unixtimestamp.com/")) for converting this to a human-readable time. 

PowerBI is not one of them.

If you try to use the built-in data type conversion tools to turn this into a timestamp, it doesn't work. 

PZ image

Here's my workaround. 

    let
        getDateTimeZone = (optional UNIX_Timestamp) =>
    	let
        	Source = if Value.Is(UNIX_Timestamp, type text) then Number.FromText(UNIX_Timestamp , "en-US") else UNIX_Timestamp,
        	step2 = try DateTimeZone.ToLocal(DateTime.AddZone(#datetime(1970, 1, 1, 0, 0, 0) + #duration(0, 0, 0, Source/1000),0)) otherwise null
    	in
        	step2
    in 
        getDateTimeZone

Adding the following code to a new blank query will create a function.

PZ image

This function takes in an integer (or text - and converts it to an integer), and tries to convert it to a DateTimeZone value by adding the unix timestamp (as a duration of seconds) to a 1970 timestamp. If the value is null, it returns null.

The good news is that this works! The bad news is that this is more complex than just changing the field type. For each column we want to fix, we'll have to add a query step that adds a new column that invokes that function on the original column, and then delete the original column. 

    = Table.AddColumn(#"..Previous Step..", "start_date", each getDateTimeZone([original_start_date]))
    = Table.RemoveColumns(#"Invoked Custom Function1",{"original_start_date"})

PZ image of updated columns as times

### PowerBI.com Doesn't Support Refreshing Non-Static Queries

Remember how we had to use parameters and row-executed functions to overcome those pagination issues? Well, the online service doesn't like that very much.

This is, for most people, a complete non-starter.

What's incredible to me is how this is such a random limitation. You can connect to it on desktop, publish to the service, _manually_ refresh on desktop, but you can't _schedule nor trigger_ a refresh on the service.

This means people can access the report from the website, but in order for that data to be up to date, a workspace admin would need to open the desktop file, hit Refresh, and re-publish the report. Even in a once-a-day cadence, this is an absurdly manual process.

After much googling, I was able to find someone else dealing with this problem, and a good blog post by Chris Webb: [Web.Contents(), M Functions And Dataset Refresh Errors In Power BI](https://blog.crossjoin.co.uk/2016/08/23/web-contents-m-functions-and-dataset-refresh-errors-in-power-bi/ "Web.Contents(), M Functions And Dataset Refresh Errors In Power BI")

> This query refreshes with no problems in Power BI Desktop. However, when you publish a report that uses this code to PowerBI.com and try to refresh the dataset, you’ll see that refresh fails and returns a rather unhelpful error message:  
> _Unable to refresh the model because it references an unsupported da_
>
> _ta source._

So here's the problem, as stated by Chris Webb

> When a published dataset is refreshed, Power BI does some static analysis on the code to determine what the data sources for the dataset are and whether the supplied credentials are correct. Unfortunately in some cases, such as **when the definition of a data source depends on the parameters from a custom M function, that static analysis fails and therefore the dataset does not refresh.**

Chris Webb is able to solve this for himself by setting a static query URL and adding parameters to the end as a data payload.

    //before
    Web.Contents(
    	"https://data.gov.uk/api/3/action/package_search?q="&Term
    )
                
    //after
    Web.Contents(
    	"https://data.gov.uk/api/3/action/package_search", 
    	[Query=[q=Term]]
    )

But this is what we were _already_ doing for things like "subtasks=TRUE". We weren't concatenating that into the URL, we were passing it as a parameter.

What we _can't_ set as a static part of the the relative path. So, earlier, when we needed to iterate through and run a query for each space to get a list of folders inside each one? 

    Source = Json.Document(
    	Web.Contents("https://api.clickup.com/api/v2", 
        [RelativePath="space/" & spaceid & "/list",
        ...

That space id isn't a parameter, it's part of the actual path we're connecting to. 

As far as I've found, there's no workaround for this.

If you're still determined at this point, here's what I suggest:   
you could hardcode the list of spaces/folders to query as a set of static URL strings.   
The report itself can now be refreshed on the service, but you will still have to occasionally go in and manually update those strings yourself. If people add new folders/spaces/etc, they won't automatically get queried by the report until you edit the file to do so.