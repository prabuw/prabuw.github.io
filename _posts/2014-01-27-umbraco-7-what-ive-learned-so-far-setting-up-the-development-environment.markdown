---
layout: post
title: "Umbraco 7: What I&#8217;ve learned so far - Setting up the development environment"
date: 2014-01-27
categories: [Umbraco]
keywords: "Umbraco 7, CMS, Content Management System, Umbraco basics, Umbraco 7 Setup, NuGet"
description: "Setting up Umbraco 7 CMS via an empty ASP.NET MVC 4 project and NuGet."
---
Recently, I picked up a side project to build a CMS powered site for a small company. It has been a while since I last
worked with a CMS (6 years ago), which was [DotNetNuke](http://www.dnnsoftware.com "DotNetNuke") and that
was a painful experience for me. Anyways, I thought I would give Umbraco a go, at the advice of a colleague.

I've never used [Umbraco](http://umbraco.com/ "Umbraco") before, and starting with version 7 was mildly painful.
This was mostly due to the lack of documentation out there yet and also the concepts they use to build the site up
were foreign to me. This is my knowledge I have acquired so far.

**NOTE: I just started using Umbraco, so there might be a better or a different way to achieve the same things I am
doing. If you know these things, please get in touch so I can amend the post and also learn. Sharing is caring!**

#### Installing a fresh copy#### 

Follow these [instructions](http://our.umbraco.org/documentation/installation/install-umbraco-with-nuget "instructions") 
and you will be fine. My steps are as follows:

1. Create a new instance of an ASP.NET MVC 4 project and use the empty template.
2. Followed that up by running the following command in the NuGet package console: Install-Package UmbracoCms. It will
   download all the dependencies, and also prompt you to overwrite the web.config file in the console, make sure to
   reply yes.

    <div class="centered">
       <img src="/images/nugetprompt.jpg"  alt="NuGet prompt" style="width: 640px; height: 86px" />
    </div>

3. Debug the solution and go through steps in the wizard.
    a. I personally prefer to use a blank SQL Server as a database, so it is easier to back up and restore when
       deploying to where the website is hosted. Alternatively you can choose to the in-built SQL Compact (SQL CE 4)
       to get started, if you do not have a copy of SQL Server.
    b. You can pick a started kit, that will have configure the CMS with a pre-baked example site. This can be useful
       for you to get your head around some of the concepts in Umbraco, and how they relate to each other. However,
       when I built the final site, I chose not use one to have a clean install.

    <div class="centered">
        <img src="/images/wizard.jpg"  alt="Wizard" style="width: 640px; height: 379px" />
    </div>

4. Once the install is complete, it will direct you to the backend login page, where you can login and see the inside of
the CMS.
