---
color: secondary
categories: ''
title: How to connect PowerBI to a Tabular API (and why you shouldn't)
description: You can get tabular REST APIs working through liberal use of custom PowerQuery
  code, and I'll show you how.
style: ''
image: assets/images/pbidesktop_q3ikg6bxqs.png
tags:
- APIs
- PowerQuery
- Hacks
- PowerBI
toc: true

---
{:toc}

Before we start, I'll be basing this on the [Clickup API](https://clickup.com/api "Clickup API") (their API documentation is actually pretty good, you should give it a check!).  
Clickup is a project management tool, similar to Monday.com and Azure DevOps.  
{% include elements/highlight.html text="Nothing in this document is all that specific to Clickup"%}, so the contents should _mostly_ hold true for whatever other REST API you're looking at.  
The data we're looking at will contain things like tasks, assigned people, tags, due-dates, etc.

***

I say "why you shouldn't" because, as we'll learn, doing a direct connection like this is a series of problem-solving activities based around overcoming the inherent limitations of PowerBI by creating PowerQuery functions ourselves.

## The basics: getting data

Let's open up our query editor (Home->Transform Data) and set up a few helper parameters (variables).  
![](/assets/images/screenshot-2022-01-29-230305.jpg)

Here we can create variables just like in many other programming languages.  
I'm going to create two parameters:

`ClickUpTeamID` - This is the ID that represents your entire company/workspace. _Everything_ lives in here (people, tasks, permissions, etc). You can see it whenever you use clickup, your url will start with something like `https://app.clickup.com/12651678/...`

`ClickUpAPIKey` - If you've used API's before, you know what this is. It's both a username and password, one single value that authenticates whoever uses it. As such, the actual value is hidden pretty religiously by administrators, as simply seeing it is enough for anyone to copy-paste and start using it. You can get your key from [https://app.clickup.com/10631688/settings/apps](https://app.clickup.com/10631688/settings/apps "https://app.clickup.com/10631688/settings/apps").

![](/assets/images/ifq2qyzgfa.png)

Now that we have our two parameters, let's build a quick query to grab the open tasks.

![](/assets/images/9dlbhqdqb2.png)

Looking at the documentation, I also included the `include_closed` and `subtasks` parameters to get those tasks as well.

        let
            Source = Json.Document(Web.Contents("https://api.clickup.com/api/v2",
                [RelativePath="team/" & ClickUpTeamID & "/task",
                Query=[include_closed="TRUE", 
                       subtasks="TRUE"],
                Headers=[Authorization=ClickUpAuthHeader]]))
        in 
            Source

If we run this, we get back 100 tasks. Which is great, but not enough. If we're building this to do analysis on our team's work habits/capacity, we need to see everything.

According to the API specification, tasks is a paginated GET. We need to include the `page` parameter to specify which page to go to beyond the first.

So an example query that would get the _next_ 100 results might be

     let
     	Source = Json.Document(Web.Contents("https://api.clickup.com/api/v2",
        	[RelativePath="team/" & ClickUpTeamID & "/task",
            Query=[include_closed="TRUE", 
            	subtasks="TRUE",
                page=2],
    		Headers=[Authorization=ClickUpAuthHeader]]))
    in 
    	Source

### Issue #1: PowerBI Doesn't Support Pagination

In order to get all the results we're looking for, we're can't just do a standard query. We need to do one query per page, passing in a different `page` value, keep going until the result is blank (no more tasks), and then combine all the values.

PowerBI doesn't have a built-in way to do this. Ideally, the UI would give us an option to treat this as a paginated API and treat it accordingly, but no such luck. We'll have to code something ourselves.

#### Creating Functions

the `let` and `in` operators define a query in PowerQuery. `let` defines the steps, and `in` sets the final object to be returned (99% of the time, `in` is just the name of your final step).

If we add an input line like `(PageNo as number) =>` to the top of the code, above `let`, we can turn our query into a function.

I'm also going to edit our code to use that variable as the page to go get from the REST API.

    (PageNo as number) =>
    let
    	Source = Json.Document(Web.Contents("https://api.clickup.com/api/v2",
        	[RelativePath="team/" & ClickUpTeamID & "/task",
            Query=[include_closed="TRUE", 
            	   subtasks="TRUE", 
                   page=Number.ToText(PageNo)],
    		Headers=[Authorization=ClickUpAuthHeader]]))
    in 
    	Source

This changes the query into a UI that accepts a a parameter.

![](/assets/images/pbidesktop_avblzzi4oq.png)

When we now use the UI to provide a value for `PageNo` and invoke it, a new query will be created that invoked the function, e.g.

    let
        Source = ClickupTasks(3)
    in
        Source

which will get the third page of results.

With this in mind, I'm going to rename our function-ized query to `GetClickupTaskPage`, and create a new query that looks like this.

    let
        Records = List.Generate(()=> 
            [Source = ClickupGetTaskPages(0), Page=0],
            each List.Count([Source][tasks]) > 0,
            each [Source = ClickupGetTaskPages([Page]+1), Page=[Page] +1],
            each [Source])
     in
     	Records

This code will generate a list with an accompanying `Page` variable, and while the `Source` query-object isn't blank (still returning tasks), it will increment the variable and call `Source` again with the next page, before finally giving us all the `Source`'s together as a big list.

![](/assets/images/pbidesktop_q3ikg6bxqs.png)Each of these rows is a query we made, containing up to 100 rows each.

From there, I run the following steps to convert this list into a table, and expand the `Record` objects down to the row-level so that they're not summarized objects anymore. (I did this through the UI, but here's the code it generates)

    #"Converted to Table" = Table.FromList(Records, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"tasks"}, {"Column1.tasks"}),
    #"Expanded Column1.tasks" = Table.ExpandListColumn(#"Expanded Column1", "Column1.tasks"),
    #"Expanded Column1.tasks1" = Table.ExpandRecordColumn(#"Expanded Column1.tasks", "Column1.tasks", {"id", "custom_id", "name", "text_content", "description", "status", "orderindex", "date_created", "date_updated", "date_closed", "archived", "creator", "assignees", "watchers", "checklists", "tags", "parent", "priority", "due_date", "start_date", "points", "time_estimate", "custom_fields", "dependencies", "linked_tasks", "team_id", "url", "permission_level", "list", "project", "folder", "space"}, {"id", "custom_id", "name", "text_content", "description", "status", "orderindex", "date_created", "date_updated", "date_closed", "archived", "creator", "assignees", "watchers", "checklists", "tags", "parent", "priority", "due_date", "start_date", "points", "time_estimate", "custom_fields", "dependencies", "linked_tasks", "team_id", "url", "permission_level", "list", "project", "folder", "space"}),

And just like that, we've overcome our first hurdle. We got our tasks.

![](/assets/images/pbidesktop_pcoxlrtkq9.png)

### Issue #2: PowerBI Doesn't Support UNIX Timestamps

Checking the data returned from the api, I immediately noticed something odd. All of the time columns (start date, due date, ...) have an integer value (like 1641782153786).

![](/assets/images/pbidesktop_aqve92pybg.png)

This is called a UNIX Timestamp, and represents the number of seconds since January 1st, 1970 (when time started existing).

There are some [great tools](https://www.unixtimestamp.com/) for converting this to a human-readable time.

PowerBI is not one of them.

If you try to use the built-in data type conversion tools to turn this into a timestamp, it doesn't work.

![](/assets/images/pbidesktop_rxfubybdbl.png)![](/assets/images/pbidesktop_zu8dnxijry.png)

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

![](/assets/images/pbidesktop_i2q0gyvz2m.png)

This function takes in an integer (or text, and converts it to an integer), and tries to convert it to a DateTimeZone value by adding the unix timestamp (as a duration of seconds) to a 1970 timestamp. If the value is null, it returns null instead so that we don't get errors.

The good news is that this works! The bad news is that this is more complex than just changing the field type. For each column we want to fix, we'll have to add a query step that adds a _new_ column that invokes this function on the original column, and then delete the original column.

    = Table.AddColumn(#"..Previous Step..", "start_date", each getDateTimeZone([original_start_date]))
    = Table.RemoveColumns(#"Invoked Custom Function1",{"original_start_date"})

![](/assets/images/pbidesktop_80aptxpxbo.png)

### Creating context-specific queries

Once I started adding other queries to the api, I noticed a problem. For a few of the things I wanted to query (like tags, folders, and lists) there was no way to query the entire environment for them; I could only see them within the context of a larger bucket like `space`

! In Clickup terms, a `space` is a bucket inside the environment that you can have any number of lists/folders in. If you had a business, you might have a space for accounting, sales, development, etc. that you have many folders and lists inside.

So, I can't just "get all the folders". First, I need to get all the spaces (a space just being a top-level bucket), and then run a query like this for each.

    let 
    	Source = Json.Document(Web.Contents("https://api.clickup.com/api/v2",
            [RelativePath="space/" & ~space id~ & "/folder",
            Query=[],
            Headers=[Authorization=ClickUpAuthHeader]]))
    in
    	Source

We can combine our previous two concepts to do exactly this.

First, I'll take the code above and convert it into a function.

    (spaceid as text) as table =>
    let
        Source = Json.Document(Web.Contents("https://api.clickup.com/api/v2",
            [RelativePath="space/" & spaceid & "/folder",
            Query=[
            ],
            Headers=[Authorization=ClickUpAuthHeader]]))
    in
    	Source

Then, I'll run a query to get my list of spaces, and add a step to invoke this function (just like I did with the timestamps) on _every row_. This returns a table in each row with a list of folders in that row's space. I can expand the values (like we did for tasks) to get my folders out and visible in a row-level.

    let
        Source = Json.Document(Web.Contents(ClickUpBaseURL,
            [RelativePath="team/" & ClickUpTeamID & "/space",
            Query=[archived="FALSE"
            ],
            Headers=[Authorization=ClickUpAuthHeader]])),
        spaces = Source[spaces],
        #"Converted to Table" = Table.FromList(spaces, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
        #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"id", "name", "private", "statuses", "multiple_assignees", "features", "archived"}, {"space_id", "name", "private", "statuses", "multiple_assignees", "features", "archived"}),
    	#"Added Custom" = Table.AddColumn(#"Expanded Column1", "Custom", each ClickupGetFolders([space_id])),
        #"Removed Errors" = Table.RemoveRowsWithErrors(#"Added Custom", {"Custom"}),
        #"Expanded Custom" = Table.ExpandTableColumn(#"Removed Errors", "Custom", {"Folder.id", "Folder.name", "Folder.orderindex", "Folder.override_statuses", "Folder.hidden", "Folder.space", "Folder.task_count", "Folder.archived", "Folder.statuses", "Folder.lists", "Folder.permission_level"}, {"Folder.id", "Folder.name", "Folder.orderindex", "Folder.override_statuses", "Folder.hidden", "Folder.space", "Folder.task_count", "Folder.archived", "Folder.statuses", "Folder.lists", "Folder.permission_level"}),
    in
        #"Expanded Custom"

![](/assets/images/pbidesktop_vgi4u39b9v.png)

This dynamic query-path allows us to get everything we need.

however...

### Issue #3: PowerBI.com Doesn't Support Refreshing Non-Static Query Paths

Remember how we had to use parameters and row-executed functions to overcome those pagination issues? And how we had to dynamically change to query path to get the right space to look at?

Well, the online service doesn't like that very much, and it won't let us refresh the data online once we publish this dashboard.

![](/assets/images/brave_xbetnzvjln.png)

What's incredible to me is how this is such a random limitation.

You can connect to your data on desktop, _manually_ refresh on desktop just fine, publish to the service and see the data online, but you can't _schedule nor trigger_ a refresh on the service.

This means people can access the report from the website, but in order for that data to be up to date, a workspace admin would need to open the desktop file, hit Refresh, and re-publish the report. Even in a once-a-day cadence, this is an absurdly manual process.

After much googling, I was able to find someone else dealing with this problem, and a good blog post by Chris Webb: [Web.Contents(), M Functions And Dataset Refresh Errors In Power BI](https://blog.crossjoin.co.uk/2016/08/23/web-contents-m-functions-and-dataset-refresh-errors-in-power-bi/ "Web.Contents(), M Functions And Dataset Refresh Errors In Power BI")

> This query refreshes with no problems in Power BI Desktop. However, when you publish a report that uses this code to PowerBI.com and try to refresh the dataset, you’ll see that refresh fails and returns a rather unhelpful error message:  
> _Unable to refresh the model because it references an unsupported data source._

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

You could hardcode the list of spaces/folders to query as a set of static URL strings.  
The report itself can now be refreshed on the service, but you will still have to occasionally go in and manually update those strings yourself. If people add new folders/spaces/etc, they won't automatically get queried by the report until you edit the file to do so.

Alternatively, since you have all the tasks, you could get your lists of users/folders/lists/tags by creating summary tables that reference your Tasks table and grab the distinct values out of it. The downside of this one is that, as far as PowerBI is now concerned, anything that isn't being used by a task doesn't exist. You would not see any empty lists, inactive users, etc. You could imagine this might make your dashboard especially volatile if you were only pulling open tasks, as entire folders and people blink out of existence after closing their tasks.

### Maybe use an ETL server instead.

Because of all these issues, I decided to use an online ETL server to first stage the data from the API. 

Setting up a proper data pipeline offers a variety of benefits, not least of which is that any transformations you make can be accessible as a single 'source of truth', instead of being silo'd in the report. All of the workarounds we had to do to get the API working can be dropped in favor of pulling in standardized tables from our server.  
  
I went with [https://www.keboola.com/](https://www.keboola.com/ "https://www.keboola.com/") as their free tier was generous enough that I was able to accomplish this without cost.

For posterity, here' the code I used to pull the `Tasks` as an example of their configuration structure.

    {
     "parameters": {
      "api": {
       "baseUrl": "https://api.clickup.com/api/v2/team/[redacted]/",
       "http": {
        "headers": {
         "Authorization": "pk_[redacted]",
         "Content-Type": "application/json;charset=UTF-8"
        },
        "defaultOptions": {
         "params": {
          "include_closed": true,
          "subtasks": true,
          "assignees%5B%5D": "Sam Cochrane"
         }
        }
       },
       "pagination": {
        "method": "pagenum",
        "firstPage": 0
       }
      },
      "config": {
       "debug": true,
       "outputBucket": "bk_clickup",
       "jobs": [
        {
         "endpoint": "task",
         "dataField": "tasks",
         "responseFilter": [
          "custom_fields"
         ]
        }
       ]
      }
     }
    }