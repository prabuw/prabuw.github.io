---
layout: post
title: "Umbraco 7: What I&#8217;ve learned so far - Custom Controllers and Models"
date: 2014-02-16
categories: [Umbraco]
keywords: "Umbraco 7, CMS, Content Management System, Umbraco Controllers"
description: "Customising Controllers and Models in an Umbraco 7 CMS."
comments: true
---
Moving on from document types and templates, I wanted to build a model and have a view inherit it, as per the
traditional MVC structure. All pages (that have a template associated with it), result in a physical file (with a
.cshtml extension) being created by Umbraco. These pages are placed in the **Views** folder.

Theses pages by default inherit from the following code:

``` csharp
@inherits Umbraco.Web.Mvc.UmbracoTemplatePage
```

This (to my understanding) is the equivalent of the following:

``` csharp
Umbraco.Web.Mvc.UmbracoViewPage
```

RenderModel is Umbraco's representation of the page's information and content stored in the database. In fact, the
data the CMS user has entered for the page is stored in a field called **properties**. Umbraco takes this and renders
the view following it's convention of looking for view in the **Views** folder.

Access to a property's value can be done as such (straight from the view):

``` csharp
@Model.Content.GetPropertyValue("PropertyName")
```

Generally, I would have to check if the property exists first, before retrieving the value. This repetitive pattern of
code can be replaced with an extension method.

``` csharp
public static T CheckPropertyExistsAndGetValue(this IPublishedContent content, string propertyName)where T : class
{
    if (content.HasProperty(propertyName) && content.GetProperty(propertyName).HasValue)
    {
        return content.GetPropertyValue(propertyName);
    }
   
    return default(T);
}
```

Even with an extension method, I came across situations where more logic is required. Previous versions of Umbraco seem
to recommend placing logic in the view, mixed amongst the HTML mark up. I am more in favour of building a model
(placing the logic in the model or a controller), and placing limited logic in the view.

To achieve this, I had to two things, make the view inherit from a custom model and create a custom controller for
Umbraco to execute.

Firstly create the custom model. Note that I am extending the **RenderModel**, which allows you to get access to page's
data.

``` csharp
public class PersonModel : RenderModel
{
	public string Name { get; set; }
	public string Occupation { get; set; }</p>
	public PersonModel(RenderModel model) : base(model.Content, model.CurrentCulture)
	{
	    // Code to extract the data from the render
	    // model and place it in the properties above.
	}
}
```

Then use the custom model in the view, place the code shown below:

``` csharp
@inherits UmbracoViewPage;
```

Finally, we need to create a custom controller that will intercept the request. This is referred as
[hijacking](http://our.umbraco.org/documentation/Reference/Mvc/custom-controllers "hijacking") by Umbraco. The
controller file must be placed in the Controllers folder as per the MVC conventions. It must also inherit the
**RenderMvcController** class as seen below.

``` csharp
public class HomeController : Umbraco.Web.Mvc.RenderMvcController
{
    public ActionResult Index(RenderModel model)
    {
        var customModel = new PersonModel(model);
        return View("Home", customModel);
    }
}
```

The convention Umbraco requires that the controller be named after the document type, so in the example above, the
document type is called **home**. The name of the action corresponds to the template. However, if no action matches
then the **Index** action will be executed.