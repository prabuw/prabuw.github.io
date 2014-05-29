---
layout: post
title: "Umbraco 7: What I&#8217;ve learned so Far - Document types and templates"
date: 2014-01-27
categories: [Umbraco]
keywords: "Umbraco 7, CMS, Content Management System, Umbraco basics, Umbraco Document Types, Umbraco Properties, Umbraco Tabs, Umbraco Templates, 
Umbraco Content"
description: "An introduction to the concepts in the Umbraco 7 CMS."
---
Once you login to the CMS, which can be accessed by placing '/umbraco' after the top level url. For example,
*www.examplesite.com/umbraco*. This will bring up the admin section of the CMS, and it should look something like the
image below.

<div class="centered">
    <img src="/images/umbracoadmin.jpg"  alt="Umbraco admin" style="width: 640px; height: 407px" />
</div>

####Document Types####

I equate document types to a model holding the data of a page. These models are built up of fields, which can be
modified by a user of the CMS, especially one that is not a developer. For example, a model can hold fields to represent
various content on the page, varying from text to images.

To create a document type, go to Settings &rarr; Document Types (right-click) &rarr; Create.

####Properties####

After creating a document type, I will typically add the **properties** that can be modified. Properties are stored in
the database. A property contains the following parts:

+ Name
+ Alias
    + This is what you will refer to retrieve from the data programmatically from C# code or in a template via an extension
      method Umbraco provides.
+ Type
    + This defines the type of data the property will hold, such as text, true/false, a media item etc.&nbsp; Custom data
      types can be created, but that is more advanced and beyond the scope of this post.
+ Tab
+ Mandatory
    + Handy to force a user to fill in this property.
+ Validation
    + This is useful for enforcing the format of the inputted data using Regex.
+ Description
    + This will appear under the property name, and can be handy to give properties more clarification or instructions for a CMS user.

####Tabs####

Umbraco also allows you to **group properties into tabs**. When creating a property, you can select the tab you want the
property to appear under, otherwise it will appear in a tab called **Properties**. This tab will contain Umbraco
specific information, such as the id of the page in the database. I prefer to create tabs and organise the properties
into logical groups, so it is easier users to manage the content of a page.

<div class="centered">
    <img src="/images/documenttypes.jpg"  alt="Document types" style="width: 640px; height: 327px" />
</div>

Umbraco has a concept of inheritance built into document types. This means I can create a document type as a child of
another document type. Consequently, the properties and tabs of the parent document type appear automatically in the
child. This is useful, when a group of document types share common properties.

<div class="centered">
    <img src="/images/childdocumenttypesjpg.jpg"  alt="Child document types" style="width: 244px; height: 115px" />
</div>

Umbraco also contains a feature to control the creation of document types. By default, all document types can be created
at the root of the website. However, a child document type by default won't be able to be created under the parent
unless explicitly set in parent document type. This can be done under the **Structure** tab in the document type.

An example of this is show below, where we allow users to create a blog post under the blog list document type.

<div class="centered">
    <img src="/images/structure.jpg"  alt="Structure" style="width: 640px; height: 365px" />
</div>

####Templates####
A template is defining how a page will be rendered into HTML/CSS, as well as integrating the data from the document type.
A template has a corresponding physical file created in the Views folder. A document type can be associated with zero or
more templates. This allows you to render the content held by the document type in multiple ways. A document type without
a template cannot be rendered.

####Content####
This leads to the combination of these two concepts: a **page**. In Umbraco there is a content section, which holds the
content of the website being built. Essentially, it is a structure of the website's pages. When creating a page you are
first prompted to select a document type and give it a name. You can optionally choose a layout or the default layout is
picked.

<div class="centered">
    <img src="/images/contentsection.jpg"  alt="Content section" style="width: 640px; height: 352px" />
</div>

With these three things, you should be able to get something to come up.
