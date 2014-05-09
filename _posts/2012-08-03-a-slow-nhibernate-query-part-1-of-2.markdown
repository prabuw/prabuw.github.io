---
layout: post
title: A slow NHibernate query (PART 1 of 2)
categories:
- NHibernate
tags:
- Fluent NHibernate
- Mapping-By-Code
- NHibernate
- NHibernate Attributes
- ORM
status: publish
type: post
published: true
meta: {}
author: 
---
<p align="justify">I haven’t had much experience with <a href="http://www.nhforge.org">NHibernate</a>, coming from mostly working with Microsoft’s <a href="http://msdn.microsoft.com/en-us/data/aa937709">Entity Framework</a>. The learning curve is not very steep, however it does take time to go through the documentation and examples. In some cases, documentation is scarce, for example with the mapping-by-code flavour of NHibernate. This post is about my experience with it, and in particular an issue I came across.</p>
<p align="justify"><strong><u>A very brief background into the flavours of NHibernate </u></strong></p>
<p align="justify">The mapping of a database table to a <abbr title="Plain Old CLR Object">POCO</abbr> can be achieved via a few flavours. These include:</p>
<ul>
<li>
<div align="justify">XML </div>
<li>
<div align="justify">NHibernate.Mapping.Attributes </div>
<li>
<div align="justify">Fluent </div>
<li>
<div align="justify">Mapping by code</div>
</li>
</ul>
<p align="justify">Mapping by XML is the original method to describe a mapping provided by NHibernate. My initial impressions were that the XML files tend to end up being verbose and hard to manage. I’ve&nbsp; included an example on what a very simple mapping below looks like.</p>

```csharp
namespace Learning.NHibernate
{
    public class Person
    {
        public virtual int PersonId { get; set; }
        public virtual string FirstName { get; set; }
        public virtual string LastName { get; set; }
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8" ?>
<hibernate-mapping xmlns="urn:nhibernate-mapping-2.2"
                           assembly="Learning.NHibernate"
                           namespace="Learning.NHibernate">
    <class name="Person"
     	    table="Person">
        <id name="PersonId">
            <generator class="identity"/>
        </id>
        <property name="FirstName" Column="First" length="200" />
        <property name="LastName" length="200" />
    </class>
</hibernate-mapping>
```

<p align="justify">Moving on to the next flavour, <a href="http://www.nhforge.org/doc/nh/en/index.html#mapping-attributes">NHibernate.Mapping.Attributes</a>. Decorating your POCO with attributes to describe the mapping means everything can be managed in C#. A pretty neat idea. </p>
<p align="justify">A side note: The attributes are used to generate the XML files in memory and then are used as seen above. This is what the above example looks like implemented using NHibernate.Mapping.Attributes.</p>

```csharp
using NHibernate.Mapping.Attributes;

namespace Learning.NHibernate
{
    public class Person
    {
        [Id]
        public virtual int PersonId { get; set; }

        [Property(Column=&quot;First&quot;)]
        public virtual int FirstName { get; set; }

        public virtual int LastName { get; set; }
    }
}
```

<p align="justify">Next is <a href="http://www.fluentnhibernate.org/">Fluent NHibernate</a> by James Gregory. This flavour of mapping has the beauty of <a href="https://github.com/jagregory/fluent-nhibernate/wiki/Auto-mapping">auto mapping</a>. Auto mapping in summary allows the use of convention over configuration to define your mappings, i.e. less code. This might not suit your situation, and it didn’t with mine.</p>
<p align="justify">The alternative requires defining the mapping explicitly. So using the Person object above, the mapping class required looks like below:</p>

```csharp
using FluentNHibernate.Mapping;

namespace Learning.NHibernate
{
    public class PersonMap : ClassMap&lt;Person&gt;
    {
        public PersonMap()
        {
            Id(x =&gt; x.PersonId);
            Map(x =&gt; x.FirstName, &quot;First&quot;);
            Map(x =&gt; x.LastName);
        }
    }
}
```

<p align="justify">Sidenote: Fluent NHibernate also generates the XML documents in-memory and follows the same procedure as mapping by XML. I really like this flavour of NHibernate and was settled on using it for my project, till I read about mapping-by-code.</p>
<p align="justify">As of NHibernate 3.2, NHibernate has included a new feature called mapping-by-code. It essentially is a method to describe the mapping component via code, similar to Fluent NHibernate. However, this is in the core library of NHibernate, unlike Fluent NHibernate, which is a third-party library. Furthermore, to my understanding, mapping-by-code does not generate any in-memory XML documents.</p>
<p align="justify">There isn’t any official documentation for this feature (to my knowledge and after scouring on the net). I have been using <a href="http://notherdev.blogspot.com.au/2012/02/nhibernates-mapping-by-code-summary.html">NOtherDev’s summary post</a> to learn.&nbsp; Similar to Fluent NHibernate, here is the mapping class using NHibernate mapping-by-code.</p>

```csharp
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class PersonMapping : ClassMapping&lt;Person&gt;
    {
        public PersonMapping()
        {
            Id(x =&gt; x.PersonId);
            Property(x =&gt; x.FirstName, m =&gt; m.Column(&quot;First&quot;));
            Property(x =&gt; x.LastName);
        }
    }
}
```

<p align="justify">After playing with NHibernate mapping-by-code, I really started liking it’s API and naming. The only problem was the lack of documentation.</p>
<p align="justify">Now that I’ve settled on the mapping flavour, on to the problem I hit that stumped me! Continue on to the <a href="http://pwee167.wordpress.com/2012/08/05/a-slow-nhibernate-query-part-2-of-2/">next post</a> to find out…</p>
