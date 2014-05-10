---
layout: post
title: 'Umbraco 7: What I&#8217;ve learned so Far - Document types and templates'
categories:
- CMS
- Umbraco
tags:
- Document Types
- Templates
- Umbraco 7
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p align="justify">Once you login to the CMS, which can be accessed by placing ‘/umbraco' after the top level url. For example, <em>www.examplesite.com/umbraco</em>. This will bring up the admin section of the CMS, and it should look something like the image below</p>
<p align="justify"><a href="http://pwee167.files.wordpress.com/2014/01/admin.jpg"><img title="admin" style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" border="0" alt="admin" src="http://pwee167.files.wordpress.com/2014/01/admin_thumb.jpg" width="640" height="407"></a></p>
<h4 align="justify"><font style="font-weight:bold;"><u>Document Types</u></font></h4>
<p align="justify">I equate document types to a model holding the data of a page. These models are built up of fields, which can be modified by a user of the CMS, especially one that is not a developer. For example, a model can hold fields to represent various content on the page, varying from text to images.</p>
<p align="justify">To create a document type, go to Settings –> Document Types (right-click) –> Create.</p>
<h5 align="justify"><font style="font-weight:bold;"><u>Properties</u></font></h5>
<p align="justify">After creating a document type, I will typically add the <strong><u>properties</u></strong> that can be modified. Properties are stored in the database. A property contains the following parts:</p>
<ul>
<li>
<div align="justify">Name</div>
<li>
<div align="justify">Alias </div>
<ul>
<li>
<div align="justify">This is what you will refer to retrieve from the data programmatically from C# code or in a template via an extension method Umbraco provides.</div>
</li>
</ul>
<li>
<div align="justify">Type</div>
<ul>
<li>
<div align="justify">This defines the type of data the property will hold, such as text, true/false, a media item etc.&nbsp; Custom data types can be created, but that is more advanced and beyond the scope of this post.</div>
</li>
</ul>
<li>
<div align="justify">Tab</div>
<li>
<div align="justify">Mandatory</div>
<ul>
<li>
<div align="justify">Handy to force a user to fill in this property.</div>
</li>
</ul>
<li>
<div align="justify">Validation</div>
<ul>
<li>
<div align="justify">This is useful for enforcing the format of the inputted data using Regex.</div>
</li>
</ul>
<li>
<div align="justify">Description</div>
<ul>
<li>
<div align="justify">This will appear under the property name, and can be handy to give properties more clarification or instructions for a CMS user.</div>
</li>
</ul>
</li>
</ul>
<h5 align="justify"><font style="font-weight:bold;"><u>Tabs</u></font></h5>
<p align="justify">Umbraco also allows you to <u>group properties into tabs</u>. When creating a property, you can select the tab you want the property to appear under, otherwise it will appear in a tab called <em>Properties</em>. This tab will contain Umbraco specific information, such as the id of the page in the database. I prefer to create tabs and organise the properties into logical groups, so it is easier users to manage the content of a page. </p>
<p align="justify"><a href="http://pwee167.files.wordpress.com/2014/01/documenttypes.jpg"><img title="documenttypes" style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" border="0" alt="documenttypes" src="http://pwee167.files.wordpress.com/2014/01/documenttypes_thumb.jpg" width="640" height="327"></a></p>
<p align="justify">Umbraco has a concept of inheritance built into document types. This means I can create a document type as a child of another document type. Consequently, the properties and tabs of the parent document type appear automatically in the child. This is useful, when a group of document types share common properties.</p>
<p align="justify"><a href="http://pwee167.files.wordpress.com/2014/01/childdocumenttypesjpg.jpg"><img title="childdocumenttypesJPG" style="background-image:none;float:none;padding-top:0;padding-left:0;margin-left:auto;display:block;padding-right:0;margin-right:auto;border-width:0;" border="0" alt="childdocumenttypesJPG" src="http://pwee167.files.wordpress.com/2014/01/childdocumenttypesjpg_thumb.jpg" width="244" height="115"></a></p>
<p align="justify">Umbraco also contains a feature to control the creation of document types. By default, all document types can be created at the root of the website. However, a child document type&nbsp; by default won't be able to be created under the parent unless explicitly set in parent document type. This can be done under the <em>Structure</em> tab in the document type. An example of this is show below, where we allow users to create a blog post under the blog list document type.</p>
<p align="justify"><a href="http://pwee167.files.wordpress.com/2014/01/structure.jpg"><img title="structure" style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" border="0" alt="structure" src="http://pwee167.files.wordpress.com/2014/01/structure_thumb.jpg" width="640" height="365"></a></p>
<h4 align="justify"><font style="font-weight:bold;"><u>Templates</u></font></h4>
<p align="justify">A template is&nbsp; defining how a page will be rendered into HTML/CSS, as well as integrating the data from the document type. <u>A template has a corresponding physical file created in the Views folder</u>.&nbsp; A document type can be associated with zero or more templates. This allows you to render the content held by the document type in multiple ways. A document type without a template cannot be rendered.</p>
<h4 align="justify"><font style="font-weight:bold;"><u>Content</u></font></h4>
<p align="justify">This leads to the combination of these two concepts: a <em>page</em>. In Umbraco there is a content section, which holds the content of the website being built. Essentially, it is a structure of the website's pages. When creating a page you are first prompted to select a document type and give it a name. You can optionally choose a layout or the default layout is picked.</p>
<p><a href="http://pwee167.files.wordpress.com/2014/01/contentsection.jpg"><img title="contentsection" style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" border="0" alt="contentsection" src="http://pwee167.files.wordpress.com/2014/01/contentsection_thumb.jpg" width="640" height="352"></a></p>
<p>With these three things, you should be able to get something to come up.</p>
