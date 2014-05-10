---
layout: post
title: 'Umbraco 7: What I&#8217;ve learned so far - Setting up the development environment'
categories:
- CMS
- Umbraco
tags:
- Umbraco 7
- Umbraco 7 CMS Installation
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p>Recently, I picked up a side project to build a CMS powered site for a small company. It has been a while since I last worked with a CMS (6 years ago), which was <a href="http://www.dnnsoftware.com/" target="_blank">DotNetNuke</a> and that was a painful experience for me. Anyways, I thought I would give Umbraco a go, at the advice of a colleague.</p>
<p>I've never used <a href="http://umbraco.com/" target="_blank">Umbraco</a> before, and starting with version 7 was mildly painful. This was mostly due to the lack of documentation out there yet and also the concepts they use to build the site up were foreign to me. This is my knowledge I have acquired so far.</p>
<p><strong>NOTE: I just started using Umbraco, so there might be a better or a different way to achieve the same things I am doing. If you know these things, please get in touch so I can amend the post and also learn. Sharing is caring!</strong></p>
<h3><span style="font-weight:bold;"><span style="text-decoration:underline;">Installing a fresh copy</span></span></h3>
<p>Follow these <a href="http://our.umbraco.org/documentation/installation/install-umbraco-with-nuget" target="_blank">instructions</a> and you will be fine. My steps are as follows:</p>
<ol>
<li>Create a new instance of an ASP.NET MVC 4 project and use the empty template.</li>
<li>Followed that up by running the following command in the NuGet package console: <span style="background-color:#cccccc;"><em>Install-Package UmbracoCms.</em></span><span style="background-color:#ffffff;"><em> </em>It will download all the dependencies, and also prompt you to overwrite the web.config file in the console, make sure to reply yes.</span><a href="http://pwee167.files.wordpress.com/2014/01/nugetprompt.jpg"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="nugetprompt" alt="nugetprompt" src="http://pwee167.files.wordpress.com/2014/01/nugetprompt_thumb.jpg" width="600" height="86" border="0" /></a></li>
<li>Debug the solution and go through steps in the wizard.
<ol>
<li>I personally prefer to use a blank SQL Server as a database, so it is easier to back up and restore when deploying to where the website is hosted. Alternatively you can choose to the in-built SQL Compact (SQL CE 4) to get started, if you do not have a copy of SQL Server.</li>
<li>You can pick a started kit, that will have configure the CMS with a pre-baked example site. This can be useful for you to get your head around some of the concepts in Umbraco, and how they relate to each other. However, when I built the final site, I chose not use one to have a clean install.</li>
</ol>
<div><a href="http://pwee167.files.wordpress.com/2014/01/wizard.jpg"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="wizard" alt="wizard" src="http://pwee167.files.wordpress.com/2014/01/wizard_thumb.jpg" width="600" height="379" border="0" /></a></div>
<p>&nbsp;</li>
<li>Once the install is complete, it will direct you to the backend login page, where you can login and see the inside of the CMS.</li>
</ol>
