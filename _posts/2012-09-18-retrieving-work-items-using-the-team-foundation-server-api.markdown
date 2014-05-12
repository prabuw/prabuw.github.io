---
layout: post
title: "Retrieving work items using the Team Foundation Server API"
date: 2012-09-18
categories: [TFS]
---
A while ago I worked on an internal application, a [Kanban](http://en.wikipedia.org/wiki/Kanban, "Kanban") board web
application. The purpose of the application was to provide a view for the stakeholders of pieces of work as they
travelled through the pipeline, from analysis to deployment. It allowed them to take more of an interest and at the
same time reduce the rather annoying questions of *"Where is it up to now? What's taking so long!?"*.

A piece of work and it's information was stored as a [Work Item](http://msdn.microsoft.com/en-US/library/ms244668(v=vs.90).aspx "Work Item")
within Team Foundation Server (TFS). Usually each work item will have several tasks linked to represent smaller
components such as analysis, design, development and testing. It was a fun application to work on and I have documented
my experience with working with the [Team Foundation Server (TFS) API](http://msdn.microsoft.com/en-us/library/bb130146\(v=vs.80\).aspx "Team Foundation Server \(TFS\) API")
below. The application is built on ASP.Net MVC 3, jQuery and [Twitter Bootstrap](http://twitter.github.com/bootstrap "Twitter Bootstrap")
for styling the UI.

####Connecting to TFS####
Firstly to gain access to a collection, we must connect to TFS. MSDN states that the [WorkItemStore](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.workitemstore\(v=vs.110\).aspx "WorkItemStore")
class represents a work item tracking client connection to a server that is running TFS.

```csharp
using System;
using Microsoft.TeamFoundation.Client;
using Microsoft.TeamFoundation.WorkItemTracking.Client;

namespace Learning.TFS
{
    public class TFSManager
    {
        private WorkItemStore _workItemStore;

        public void ConnectToTFS()
        {
            var collectionUri = new Uri("http://TFS:8080/TFS/MyCollection");
            var projectCollection = TfsTeamProjectCollectionFactory.GetTeamProjectCollection(collectionUri);
            _workItemStore = projectCollection.GetService<WorkItemStore>();
        }
    }
}
```
####Querying for work items####
Once a connection is acquired, I wanted to retrieve some work items. To achieve this, the TFS API provides a SQL like
language called [Work Item Query Language (WIQL)](http://msdn.microsoft.com/en-us/library/bb130155\(v=vs.80\).aspx "Work Item Query Language (WIQL)")

An example of a query would be like such:

```sql
SELECT [System.Id], [System.WorkItemType], [System.Title], [System.AssignedTo]
FROM WorkItems 
WHERE [System.TeamProject] = 'TFSTestProject'
ORDER by [System.Id] desc
```

Note above the qualifying where clause limits the query to retrieve work items from a project called *TFSTestProject*.
These types of queries are also known as **flat queries**.

After defining the query, execute it&nbsp; and iterate through the results:

```csharp
public List<WorkItemViewModel> GetWorkItems(string query)
{            
    WorkItemCollection workItemCollection = _workItemStore.Query(query);

    var workItemList = new List<WorkItemViewModel>();

    foreach (WorkItem workItem in workItemCollection)
    {

        var model = new WorkItemViewModel()
        {
            Id = workItem.Id,
            RequestNo = workItem.Fields["MyCompany.RequestNumber"].Value.ToString(),
            Title = workItem.Title,
            WorkItemType = workItem.Fields["MyCompany.WorkItemType"].Value.ToString(),
            Priority = workItem.Fields["MyCompany.Priority"].Value.ToString()
        };
    }
            
    return workItemList;
}
```
If you are wondering where *"MyCompany.RequestNumber"* comes from, these are custom fields defined in the TFS process
template used inside the TFS server. To find instructions on how to view or customise a process template, go
[here](http://msdn.microsoft.com/en-us/library/bb668982.aspx "here").

####Retrieving a single work item####
Certain fields are exempted when work items are returned as a result of a WIQL query. To retrieve the full work item,
use the *GetWorkItem* function of the TFS API, which takes in a work item id.

```csharp
public WorkItemViewModel GetWorkItem(int id)
{
    WorkItemViewModel model = null;

    var workItem = _workItemStore.GetWorkItem(id);

    if (workItem != null)
    {
        model = new WorkItemViewModel()
        {
            Id = workItem.Id,
            RequestNo = workItem.Fields["MyCompany.RequestNumber"].Value.ToString(),
            Title = workItem.Title,
            WorkItemType = workItem.Fields["MyCompany.WorkItemType"].Value.ToString(),
            Priority = workItem.Fields["MyCompany.Priority"].Value.ToString()
            Description = workItem.Description,
        };
    }
            
    return model;
}
```
####Tree query: Retrieving work items and their children####
One of the requirements of the application was to not only show the work item information, but also the linked work
items, such as tasks, bugs and issues. Retrieving a bunch of work items and then setting off back to the server to get
the linked work items per parent work item would be slow and painful!

The concept to tackling my problem:

1. Find all the work item ids of the parents meeting your criteria and also the ids of their children.
2. Retrieve the work items for the work items ids returned in step 1.

Tree query to the rescue! Below is an example of a tree query:

```sql
SELECT [System.Id], [System.State], [System.WorkItemType] 
FROM WorkItemLinks 
WHERE 
(
    Source.[System.TeamProject] = 'TFSTestProject' 
    AND Source.[System.WorkItemType] = 'Requirement' 
    AND Source.[System.State] NOT IN ('Proposed', 'Completed')
) 
AND [System.Links.LinkType] = 'System.LinkTypes.Hierarchy-Forward' 
AND Target.[System.WorkItemType] <> '' 
ORDER BY [System.Id] mode(Recursive)
```

Important components of a tree query:

+ The first thing to note is in the tree query above is that we are querying against **WorkItemLinks** instead of
  **WorkItems**.
+ Another fact to remember is that when writing your query, you have a reference to the source item and the target item,
  with the keywords **Source** and **Target** respectively. This is handy for expressing conditions in the query's
  where clause.

A tree structure by its nature means that a child element can be a parent of another. To allow the query to traverse the
tree, set the **mode** to **recursive**.

```csharp
private List<WorkItemViewModel> GetWorkItemTree(string query)
{
    var treeQuery = new Query(_workItemStore, query);
    var links = treeQuery.RunLinkQuery();

    var workItemIds = links.Select(l => l.TargetId).ToArray();

    query = "SELECT * FROM WorkItems";
    var flatQuery = new Query(_workItemStore, query, workItemIds);
    var workItemCollection = flatQuery.RunQuery();

    var workItemList = new List<WorkItemViewModel>();

    for (int i = 0; i < workItemCollection.Count; i++)
    {
        var workItem = workItemCollection[i];

        if (workItem.Type.Name == "Requirement")
        {                   
            var model = new WorkItemViewModel()
            {
                Id = workItem.Id,
                RequestNo = workItem.Fields["MyCompany.RequestNumber"].Value.ToString(),
                Title = workItem.Title,
                WorkItemType = workItem.Fields["MyCompany.WorkItemType"].Value.ToString(),
                Priority = workItem.Fields["MyCompany.Priority"].Value.ToString()
            };

            workItemList.Add(model);
        }
        else
        {
            switch (workItem.Type.Name)
            {
                case "Task":
                    var task = new TFSTask()
                    {
                        Name = workItem.Title,
                        Activity = workItem.Fields["MyCompany.Activity"].Value.ToString(),
                        Start = (DateTime?)workItem.Fields["MyCompany.ActivityStart"].Value,
                        Due = (DateTime?)workItem.Fields["MyCompany.ActivityFinish"].Value,
                        Status = workItem.State
                    };

                    workItemList.Last().Tasks.Add(task);
                    break;
                case "Issue":
                    var issue = new TFSIssue()
                    {
                        Name = workItem.Title,
                        Created = workItem.CreatedDate,
                        Due = (DateTime?)workItem.Fields["MyCompany.ActivityDue"].Value,
                        Status = workItem.State
                    };
                    workItemList.Last().Issues.Add(issue);
                    break;
                default:
                    break;
            }
        }
    }

    return workItemList;
}
```

To execute the tree query, you must use the **RunLinkQuery** method on the *[Query](http://msdn.microsoft.com/en-us/library/bb179746.aspx "Query")*
class. This returns an array of *[WorkItemLinkInfo](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.workitemlinkinfo.aspx "WorkItemLinkInfo")*
objects, which contains the following fields: **SourceId, TargetId, LinkTypeId** and **IsLocked**. An example of the
results returned:

| SourceId | TargetId | LinkTypeId | IsLocked |
|----------|----------|------------|----------|
| 0        | 3641     | 0          | FALSE    |
| 3641     | 3646     | 2          | FALSE    |
| 3641     | 3647     | 2          | FALSE    |
| 0        | 3648     | 0          | FALSE    |
| 3648     | 3978     | 2          | FALSE    |


The above results represent the following two separate trees (when the source id is zero, this denotes the root of the
tree):

<div class="centered">
    <img src="/images/tree.jpg"  alt="Tree" />
</div>

From here using some simple <abbr title="Language-Integrated Query">LINQ</abbr>, we can extract the work item ids into a
list from the array of **WorkItemLinkInfo** objects. Now we can retrieve all the work items using a single standard flat
query.

Using the tree query gave me a much nicer code base, and not to mention the significantly lower amount of trips to TFS
to get the information I needed. In retrospect, the TFS SDK was quite easy and fun to work with.
