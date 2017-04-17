---
layout: post
title: "Umbraco 7 : What I&#8217;ve learned so far - Custom routing and Handlers"
date: 2014-03-05
categories: [Umbraco]
keywords: "Umbraco 7, CMS, Content Management System, Custom Routing, Umbraco Handlers"
description: "Setting up custom routing and handlers in an Umbraco 7 CMS."
comments: true
---
Another way to achieve custom routing with Umbraco is to create an [application event handlers](http://our.umbraco.org/documentation/Reference/Events/application-startup "application event handlers"),
which is essentially overriding the global.asax. To create a custom application event handler class, you need to first
implement the **IApplicationEventHandler** interface, like below.

``` csharp
public class MyCustomStartupHandler : IApplicationEventHandler
{
    public void OnApplicationInitialized(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
    }

    public void OnApplicationStarting(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
    }

    public void OnApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
        RouteTable.Routes.MapRoute("Root", "", new
        {
            controller = "Home",
            action = "Index";
        });
    }
}
```

To achieve the custom routing, simply add routes to the route table as seen above. It is important to remember that
Umbraco places all it's views in the Views folder, and the admin portal will not show them if you were to place in
any other location.

Personally, I would prefer to able organise the views into folders, but this comes at the cost of not being able to view
them from the admin portal. I prefer using this method to controlling my routing, as it is simple yet quite powerful.
