---
layout: post
title: "A slow NHibernate query (Part 1 of 2)"
date: 2012-08-03
categories: [NHibernate]
keywords: "NHibernate, NHibernate XML, NHibernate Mapping By Code, Fluent NHibernate, NHibernate Mapping"
description: "A brief comparison of the different mapping strategies for NHibernate including examples."
comments: true
---
I haven't had much experience with [NHibernate](http://www.nhforge.org "NHibernate"), coming from mostly working with
Microsoft's [Entity Framework](http://msdn.microsoft.com/en-us/data/aa937709 "Entity Framework"). The learning curve is
not very steep, however it does take time to go through the documentation and examples. In some cases, documentation is
scarce, for example with the mapping-by-code flavour of NHibernate. This post is about my experience with it, and in
particular an issue I came across.

### A very brief background into the flavours of NHibernate

The mapping of a database table to a <abbr title="Plain Old CLR Object">POCO</abbr> can be achieved via a few flavours.
These include:

- XML
- NHibernate.Mapping.Attributes
- Fluent
- Mapping by code


Mapping by XML is the original method to describe a mapping provided by NHibernate. My initial impressions were that
the XML files tend to end up being verbose and hard to manage. I've included an example on what a very simple
mapping below looks like.

``` csharp
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

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<hibernate-mapping xmlns="urn:nhibernate-mapping-2.2" assembly="Learning.NHibernate" namespace="Learning.NHibernate">
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

Moving on to the next flavour, [NHibernate.Mapping.Attributes](http://www.nhforge.org/doc/nh/en/index.html#mapping-attributes "NHibernate mapping attributes").
Decorating your POCO with attributes to describe the mapping means everything can be managed in C#. A pretty neat idea.
A side note: The attributes are used to generate the XML files in memory and then are used as seen above. This is what
the above example looks like implemented using NHibernate.Mapping.Attributes.

``` csharp
using NHibernate.Mapping.Attributes;

namespace Learning.NHibernate
{
    public class Person
    {
        [Id]
        public virtual int PersonId { get; set; }
        
        [Property(Column="First")]
        public virtual int FirstName { get; set; }
        
        public virtual int LastName { get; set; }
    }
}
```

``` csharp
using NHibernate.Mapping.ByCode.Conformist;

namespace ConsoleApplication1
{
    public class Person
    {
        public virtual int Id { get; set; }
        public virtual string FirstName { get; set; }
        public virtual string LastName { get; set; }
    }

    public class PersonMap : ClassMapping<Person>
    {
        public PersonMap()
        {
            Table("Source");
            Id(i => i.Id);
            Property(i => i.FirstName, map => map.Type<TrimmedString>());
            Property(i => i.LastName, map => map.Type<TrimmedString>());
        }
    }
}
```

Next is [Fluent NHibernate](http://www.fluentnhibernate.org "Fluent NHibernate") by James Gregory. This flavour of
mapping has the beauty of [auto mapping](https://github.com/jagregory/fluent-nhibernate/wiki/Auto-mapping "auto mapping").
Auto mapping in summary allows the use of convention over configuration to define your mappings, i.e. less code.
This might not suit your situation, and it didn't with mine. The alternative requires defining the mapping explicitly.
So using the Person object above, the mapping class required looks like below:

``` csharp
using FluentNHibernate.Mapping;

namespace Learning.NHibernate
{
    public class PersonMap : ClassMap<Person>
    {
        public PersonMap()
        {
            Id(x => x.PersonId);
            Map(x => x.FirstName, "First");
            Map(x => x.LastName);
        }
    }
}
```

**Sidenote:** Fluent NHibernate also generates the XML documents in-memory and follows the same procedure as mapping by XML.
I really like this flavour of NHibernate and was settled on using it for my project, till I read about mapping-by-code.

As of NHibernate 3.2, NHibernate has included a new feature called mapping-by-code. It essentially is a method to
describe the mapping component via code, similar to Fluent NHibernate. However, this is in the core library of
NHibernate, unlike Fluent NHibernate, which is a third-party library. Furthermore, to my understanding, mapping-by-code
does not generate any in-memory XML documents. 

There isn't any official documentation for this feature (to my knowledge
and after scouring on the net). I have been using [NOtherDev's summary post](http://notherdev.blogspot.com.au/2012/02/nhibernates-mapping-by-code-summary.html "NOtherDev's summary post")
to learn. Similar to Fluent NHibernate, here is the mapping class using NHibernate mapping-by-code.

``` csharp
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class PersonMapping : ClassMapping<Person>
    {
        public PersonMapping()
        {
            Id(x => x.PersonId);
            Property(x => x.FirstName, m => m.Column("First"));
            Property(x => x.LastName);
        }
    }
}
```

After playing with NHibernate mapping-by-code, I really started liking it's API and naming. The only problem was the
lack of documentation. Now that I've settled on the mapping flavour, on to the problem I hit that stumped me!

Continue on to the [next post](/posts/a-slow-nhibernate-query-part-2-of-2/ "next post")
to find out...
