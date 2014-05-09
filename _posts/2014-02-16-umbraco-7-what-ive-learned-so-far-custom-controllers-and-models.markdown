---
layout: post
title: 'Umbraco 7: What I&#8217;ve learned so far - Custom Controllers and Models'
categories:
- CMS
- Umbraco
tags:
- RenderMvcController
- Umbraco 7
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p align="justify">Moving on from document types and templates, I wanted to build a model and have a view inherit it, as per the traditional MVC structure. All pages (that have a template associated with it), result in a physical file (with a .cshtml extension) being created by Umbraco. These pages are placed in the <em>Views</em> folder.</p>
<p>Theses pages by default inherit from the following code</p>

```csharp
@inherits Umbraco.Web.Mvc.UmbracoTemplatePage
```

<p>This (to my understanding) is the equivalent of the following:</p>

```csharp
Umbraco.Web.Mvc.UmbracoViewPage
```

<p align="justify">RenderModel is Umbraco’s representation of the page’s information&nbsp; and content stored in the database. In fact, the data the CMS user has entered for the page is stored in a field called <em>properties</em>. Umbraco takes this and renders the view following it’s convention of looking for view in the <em>Views</em> folder.</p>
<p align="justify">Access to a property’s value can be done as such (straight from the view):</p>

```csharp
@Model.Content.GetPropertyValue("PropertyName")
```

<p align="justify">Generally, I would have to check if the property exists first, before retrieving the value. This repetitive pattern of code can be replaced with an extension method.</p>

```csharp
public static T CheckPropertyExistsAndGetValue(this IPublishedContent content, string propertyName) where T : class
{
    if (content.HasProperty(propertyName) && content.GetProperty(propertyName).HasValue)
    {
        return content.GetPropertyValue(propertyName);
    }
	
	return default(T);
}
```

<p align="justify">Even with an extension method, I came across situations where more logic is required.&nbsp; Previous versions of Umbraco seem to recommend placing logic in the view, mixed amongst the HTML mark up. I am more in favour of building a model (placing the logic in the model or a&nbsp; controller), and placing limited logic in the view.</p>
<p align="justify">To achieve this, I had to two things, make the view inherit from a custom model and create a custom controller for Umbraco to execute.</p>
<p>Firstly create the custom model. Note that I am extending the <em>RenderModel</em>, which allows you to get access to page’s data.</p>

```csharp
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

<p>Then use the custom model in the view, place the code shown below:</p>

```csharp
@inherits UmbracoViewPage;
```

<p>Finally, we need to create a custom controller that will intercept the request. This is referred as <a href="http://our.umbraco.org/documentation/Reference/Mvc/custom-controllers" target="_blank">hijacking</a> by Umbraco.&nbsp; The controller file must be placed in the Controllers folder as per the MVC conventions. It must also inherit the <em>RenderMvcController </em>class as seen below.</p>

```csharp
public class HomeController : Umbraco.Web.Mvc.RenderMvcController
{
    public ActionResult Index(RenderModel model)
    {
	var customModel = new PersonModel(model);
        return View("Home", customModel);
    }
}
```

<p>The convention Umbraco requires that the controller be named after the document type, so in the example above, the document type is called<em> home</em>. The name of the action corresponds to the template. However, if no action matches then the <em>Index</em> action will be executed.</p>
