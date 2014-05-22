---
layout: post
title: "Organising LESS in an ASP.NET MVC project"
date: 2014-05-22
categories: [LESS]
---

If you haven't got LESS set up in your project, check out the [previous post](/posts/setting-up-less-bootstrap-aspnet-mvc-5/).

LESS allows you to split the styles into multiple files and then bring it back together, which is precisely what 
Bootstrap has done. This is done using the ```@import``` command.

<div class="centered">
    <img src="/images/less-import-command.png"  alt="LESS @import command" />
</div>

The file that combines this is called **bootstrap.less**, and is located as seen in the screen shot
below.

<div class="centered">
    <img src="/images/bootstrap-master-less.png"  alt="Bootstrap.less" />
</div>

I prefer to leave the default less files of Bootstrap or any other NuGet package (e.g. Toastr) untouched and rather 
choose to extend the default styles in another file. To achieve this, I organise the folder structure in the following 
format:

<div class="centered">
    <img src="/images/organising-less-folder-structure.png"  alt="Folder structure" />
</div>

To combine all the different LESS files, I have a LESS file called **app.less**.

<div class="centered">
    <img src="/images/app-less.png"  alt="App.less" />
</div>

Now, alter the ```RegisterBundles``` function in the BundleConfig.cs file like below:

``` csharp
var commonStylesBundle = new CustomStyleBundle(@"~/bundles/less");
commonStylesBundle.Orderer = new NullOrderer();
commonStylesBundle.Include("~/Content/app.less");
bundles.Add(commonStylesBundle);
```

As you can see it will transform the app.less file, and in turn transform all the other included LESS files.
Finally, make sure to update the front-end and to test it out.

``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - My ASP.NET Application</title>
    @Styles.Render("~/Bundles/less")
    @Scripts.Render("~/bundles/modernizr")

</head>
```

The outcome of this will be a single CSS file downloaded by the browser using the transformational powers of LESS. I 
find this allows me to add some order to the madness that is managing CSS styles.