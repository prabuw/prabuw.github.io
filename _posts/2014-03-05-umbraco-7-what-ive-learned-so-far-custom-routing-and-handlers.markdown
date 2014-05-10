---
layout: post
title: 'Umbraco 7 : What I&#8217;ve learned so far - Custom routing and Handlers'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p>Another way to achieve custom routing with Umbraco is to create an <em><a href="http://our.umbraco.org/documentation/Reference/Events/application-startup" target="_blank">application event handlers</a></em>, which is essentially overriding the global.asax. To create a custom application event handler class, you need to first implement the <em>IApplicationEventHandler</em> interface, like below. </p>

```csharp
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

<p>To achieve the custom routing, simply add routes to the route table as seen above. It is important to remember that Umbraco places all it's views in the Views folder, and the admin portal will not show them if you were to place in any other location.</p>
<p>Personally, I would prefer to able organise the views into folders, but this comes at the cost of not being able to view them from the admin portal. I prefer using this method to controlling my routing, as it is simple yet quite powerful.</p>
