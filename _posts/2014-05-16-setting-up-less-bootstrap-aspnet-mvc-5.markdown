---
layout: post
title: "Setting up LESS, Bootstrap and ASP.NET MVC 5"
date: 2014-05-16
categories: [LESS]
---

Recently, I joined a project built on ASP.NET MVC 5, [Bootstrap](http://getbootstrap.com/), Web API and KnockoutJS. 
It was initially built using the default styles that comes out of the box with Bootstrap. While this looks great, we 
needed to override them with the brand colours. On top of that, we also needed to manage the styles of various third 
party plugins and our own custom styles for the application.

To make this easy, we decided to use the [LESS](http://lesscss.org/). For the uninitiated, LESS is a dynamic
stylesheet language that is a super-set of CSS. Any CSS code is valid LESS code. It introduces concepts such as 
variables, nesting, mix-ins (extensions for .NET developers) etc. This gives more expressiveness to writing and 
organising CSS. In the end, the LESS code is compiled down to standard CSS file.

After scouring the interwebs, I managed to get this integrated and working with the ASP.NET MVC 5 project. This is a 
guide for future references.

####Setting up with NuGet####

1) Start with installing LESS via our favourite package manager, NuGet.

<div class="centered">
    <img src="/images/less-nuget.png"  alt="LESS via NuGet" />
</div>

2) Uninstall the default Bootstrap NuGet package that comes with ASP.NET MVC 5.

I would uninstall this package and replace them with the Bootstrap LESS files, as per the next step.
This allows to do some *cool* LESS stuff with Bootstrap.

**WARNING: Any changes in these Bootstrap files will be lost when you uninstall the package!**

<div class="centered">
    <img src="/images/uninstall-default-bootstrap-pkg-nuget.png"  alt="Uninstall default Bootstrap NuGet package" />
</div>

3) Next, install Twitter.Bootstrap.Less

<div class="centered">
    <img src="/images/bootstrap-less-nuget.png"  alt="Bootstrap LESS files via NuGet" />
</div>

This gives you all of Bootstrap in LESS files. If you did not perform step 2, NuGet might warn you about overriding the
default Bootstrap files. Once again, if you customised them already, overriding them will cause you to loose those 
changes.

4) Then, install a bundle transformer

<div class="centered">
    <img src="/images/bundletransformer-less-nuget.png"  alt="LESS BundleTransformer via NuGet" />
</div>

This is going to process those LESS files and convert them into CSS via ASP.NET's 
[bundling and minifications](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification).

5) Finally, install JavascriptEngineSwitcher.V8

<div class="centered">
    <img src="/images/JavaScriptEngineSwitcher.V8-nuget.png"  alt="JavaScriptEngineSwitcher via NuGet" />
</div>

####Fix up Web.config####

Now, we need to tinker with Web.config file. There should be a section called ```<bundleTransformer>```, go to it.
Add the ```<less>``` child section in there as per below.

``` xml
<bundleTransformer xmlns="http://tempuri.org/BundleTransformer.Configuration.xsd">
  <less useNativeMinification="true" ieCompat="true" strictMath="false" strictUnits="false" dumpLineNumbers="None">
    <jsEngine name="V8JsEngine" />
  </less>
  <core>
    <css>
      <minifiers>
        <add name="NullMinifier" type="BundleTransformer.Core.Minifiers.NullMinifier, BundleTransformer.Core" />
      </minifiers>
      <translators>
        <add name="NullTranslator" type="BundleTransformer.Core.Translators.NullTranslator, BundleTransformer.Core" enabled="false" />
        <add name="LessTranslator" type="BundleTransformer.Less.Translators.LessTranslator, BundleTransformer.Less" />
      </translators>
    </css>
    <js>
      <minifiers>
        <add name="NullMinifier" type="BundleTransformer.Core.Minifiers.NullMinifier, BundleTransformer.Core" />
      </minifiers>
      <translators>
        <add name="NullTranslator" type="BundleTransformer.Core.Translators.NullTranslator, BundleTransformer.Core" enabled="false" />
      </translators>
    </js>
  </core>
</bundleTransformer>
```

####Let's configure the BundleConfig.cs####

BundleConfig.cs lives in the App_Start directory by default. This is where the bundling and minification is usually 
handled. We need to integrate the LESS bundling in there.

In a standard ASP.NET MVC 5 project, the Bundle.config comes with bundling setup for the default Bootstrap css included. 
We need to remove that bundling and add in the LESS bundling for Bootstrap, as show below (I've commented out the 
default bundling).

``` csharp
public static void RegisterBundles(BundleCollection bundles)
{
    bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                "~/Scripts/jquery-{version}.js"));

    bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
                "~/Scripts/jquery.validate*"));

    // Use the development version of Modernizr to develop with and learn from. Then, when you're
    // ready for production, use the build tool at http://modernizr.com to pick only the tests you need.
    bundles.Add(new ScriptBundle("~/bundles/modernizr").Include(
                "~/Scripts/modernizr-*"));

    bundles.Add(new ScriptBundle("~/bundles/bootstrap").Include(
              "~/Scripts/bootstrap.js",
              "~/Scripts/respond.js"));

    //Remove default bootstrap bundling in favour of LESS bundling
    //bundles.Add(new StyleBundle("~/Content/css").Include(
    //          "~/Content/bootstrap.css",
    //          "~/Content/site.css"));

    var commonStylesBundle = new CustomStyleBundle(@"~/bundles/bootstrapless");
    commonStylesBundle.Orderer = new NullOrderer();
    commonStylesBundle.Include("~/Content/bootstrap/bootstrap.less");
    bundles.Add(commonStylesBundle);
}
```

####Include the bundle in your views####

To use the bundle, we need to add some code to the view, specifically your ```_Layout.cshtml``` view which should be 
located in the Shared folder under views.

<div class="centered">
    <img src="/images/layout_view_location.png"  alt="Layout view" />
</div>

Open, the view and replace the following code:

``` csharp
@Styles.Render("~/Content/css")
```

with this:

``` csharp
@Styles.Render("~/Bundles/BootstrapLess")
``` 

####Time to test it: F5!####
You are now ready to test it out. Debug the application, and you should see the default ASP.NET MVC 5 web application 
with all the Bootstrap styles. 

**Note: have a look at the source of the website, and you should see something like the screenshot below.**

<div class="centered">
    <img src="/images/less-demo-website-source-code.png"  alt="LESS in the wild" />
</div>

If you click on the ```<link>``` with the LESS file as a href, you will see the CSS that was generated from. There is 
one more step we need to do, add [minification](http://en.wikipedia.org/wiki/Minification_(programming\)).

####Minify the LESS output####

To achieve this, we need to enable it in the BundleConfig.cs file. Open it and add the following line at the end of the 
```RegisterBundles``` function.

``` csharp
BundleTable.EnableOptimizations = true;
```

####Part 2: Organising LESS in an ASP.NET MVC project####

[Part 2 of this post](/posts/organising-less-in-an-aspnet-mvc-project/), will discuss how I chose to organise and manage 
the LESS files of Bootstrap, third party plugins (e.g. Toastr) and our site specific custom CSS styles in the project.
