---
layout: post
title: Retrieving work items using the Team Foundation Server API
categories:
- TFS
tags:
- Team Foundation Server
status: publish
type: post
published: true
meta:
  twitter_cards_summary_img_size: a:7:{i:0;i:227;i:1;i:160;i:2;i:2;i:3;s:24:"width="227"
    height="160"";s:4:"bits";i:8;s:8:"channels";i:3;s:4:"mime";s:10:"image/jpeg";}
author: 
---
<p align="justify">A while ago I worked on an internal application, a <a href="http://en.wikipedia.org/wiki/Kanban">Kanban</a> board web application. The purpose of the application was to provide a view for the stakeholders of pieces of work as they travelled through the pipeline, from analysis to deployment. It allowed them to take more of an interest and at the same time reduce the rather annoying questions of <em>“Where is it up to now? What's taking so long!?”</em>.</p>
<p align="justify">A piece of work and it's information was stored as a <a href="http://msdn.microsoft.com/en-US/library/ms244668(v=vs.90).aspx">Work Item</a> within Team Foundation Server (TFS). Usually each work item will have several tasks linked to represent smaller components such as analysis, design, development and testing.</p>
<p align="justify">It was a fun application to work on and I have documented my experience with working with the <a href="http://msdn.microsoft.com/en-us/library/bb130146(v=vs.80).aspx">Team Foundation Server (TFS) API</a> below. The application is built on ASP.Net MVC 3, jQuery and <a href="http://twitter.github.com/bootstrap/">Twitter Bootstrap</a> for styling the UI.</p>
<p align="justify"><strong><u>Connecting to TFS</u></strong></p>
<p align="justify">Firstly to gain access to a collection, we must connect to TFS. MSDN states that the <a href="http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.workitemstore(v=vs.110).aspx">WorkItemStore</a> class represents a work item tracking client connection to a server that is running TFS. </p>

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
<p><strong><u>Querying for work items</u></strong></p>
<p align="justify">Once a connection is acquired, I wanted to retrieve some work items. To achieve this, the TFS API provides a SQL like language called <a href="http://msdn.microsoft.com/en-us/library/bb130155(v=vs.80).aspx">Work Item Query Language (WIQL)</a>.</p>
<p align="justify">An example of a query would be like such:</p>

```sql
SELECT [System.Id], [System.WorkItemType], [System.Title], [System.AssignedTo]
FROM WorkItems 
WHERE [System.TeamProject] = 'TFSTestProject'
ORDER by [System.Id] desc
```

<p align="justify">Note above the qualifying where clause limits the query to retrieve work items from a project called <em>TFSTestProject</em>. These types of queries are also known as <em>flat queries</em>.</p>
<p align="justify">After defining the query, execute it&nbsp; and iterate through the results:</p>

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
<p align="justify">If you are wondering where <em>“MyCompany.RequestNumber”</em> comes from, these are custom fields defined in the TFS process template used inside the TFS server. To find instructions on how to view or customise a process template, go <a href="http://msdn.microsoft.com/en-us/library/bb668982.aspx">here</a>.</p>
<p><strong><u>Retrieving a single work item</u></strong></p>
<p>Certain fields are exempted when work items are returned as a result of a WIQL query. To retrieve the full work item, use the <em>GetWorkItem</em> function of the TFS API, which takes in a work item id.</p>

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
<p><strong><u>Tree query: Retrieving work items and their children</u></strong></p>
<p align="justify">One of the requirements of the application was to not only show the work item information, but also the linked work items, such as tasks, bugs and issues. Retrieving a bunch of work items and then setting off back to the server to get the linked work items per parent work item would be slow and painful!</p>
<p align="justify">The concept to tackling my problem:</p>
<ol>
<li>
<div align="justify">Find all the work item ids of the parents meeting your criteria and also the ids of their children. </div>
<li>
<div align="justify">Retrieve the work items for the work items ids returned in step 1. </div>
</li>
</ol>
<p align="justify">Tree query to the rescue! Below is an example of a tree query:</p>

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

<p align="justify">Important components of a tree query:</p>
<ul>
<li>
<div align="justify">The first thing to note is in the tree query above is that we are querying against <em>WorkItemLinks</em> instead of <em>WorkItems</em>. </div>
<li>
<div align="justify">Another fact to remember is that when writing your query, you have a reference to the source item and the target item, with the keywords <em>Source</em> and <em>Target</em> respectively.&nbsp; This is handy for expressing conditions in the query's where clause.</div>
<li>
<div align="justify">A tree structure by its nature means that a child element can be a parent of another. To allow the query to traverse the tree, set the <em>mode</em> to <em>recursive</em>. </div>
</li>
</ul>

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
<p align="justify">To execute the tree query, you must use the <em>RunLinkQuery</em> method on the <em><a href="http://msdn.microsoft.com/en-us/library/bb179746.aspx">Query</a></em> class. This returns an array of <em><a href="http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.workitemlinkinfo.aspx">WorkItemLinkInfo</a></em> objects, which contains the following fields: <em>SourceId, TargetId, LinkTypeId</em> and <em>IsLocked</em>. An example of the results returned:</p>
<table border="0" cellspacing="0" cellpadding="2" width="312">
<tbody>
<tr>
<td valign="top" width="100">SourceId</td>
<td valign="top" width="100">TargetId</td>
<td valign="top" width="100">LinkTypeId</td>
<td valign="top" width="10">IsLocked</td>
</tr>
<tr>
<td valign="top" width="100">0</td>
<td valign="top" width="100">3641</td>
<td valign="top" width="100">0</td>
<td valign="top" width="10">FALSE</td>
</tr>
<tr>
<td valign="top" width="100">3641</td>
<td valign="top" width="100">3646</td>
<td valign="top" width="100">2</td>
<td valign="top" width="10">FALSE</td>
</tr>
<tr>
<td valign="top" width="100">3641</td>
<td valign="top" width="100">3647</td>
<td valign="top" width="100">2</td>
<td valign="top" width="10">FALSE</td>
</tr>
<tr>
<td valign="top" width="100">0</td>
<td valign="top" width="100">3648</td>
<td valign="top" width="100">0</td>
<td valign="top" width="10">FALSE</td>
</tr>
<tr>
<td valign="top" width="100">3648</td>
<td valign="top" width="100">3978</td>
<td valign="top" width="100">2</td>
<td valign="top" width="10">FALSE</td>
</tr>
</tbody>
</table>
<p align="justify"><font face="Georgia">The above results represent the following two separate trees (when the source id is zero, this denotes the root of the tree):</font></p>
<p align="justify"><font face="Georgia"></font>&nbsp;</p>
<p><a href="http://pwee167.files.wordpress.com/2012/10/tree.jpg"><img style="background-image:none;padding-left:0;padding-right:0;display:block;float:none;margin-left:auto;margin-right:auto;padding-top:0;border-width:0;" title="tree" border="0" alt="tree" src="http://pwee167.files.wordpress.com/2012/10/tree_thumb.jpg" width="227" height="160"></a></p>
<p align="justify">&nbsp;</p>
<p align="justify">From here using some simple <abbr title="Language-Integrated Query">LINQ</abbr>, we can extract the work item ids into a list from the array of <em>WorkItemLinkInfo</em> objects. Now we can retrieve all the work items using a single standard flat query. </p>
<p align="justify">Using the tree query gave me a much nicer code base, and not to mention the significantly lower amount of trips to TFS to get the information I needed. In retrospect, the TFS SDK was quite easy and fun to work with.</p>
