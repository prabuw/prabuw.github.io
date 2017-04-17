---
layout: post
title: "Using an IUserType with an Id field in NHibernate with Mapping-By-Code"
date: 2013-09-29
categories: [NHibernate]
keywords: "NHibernate, IUserType, Id Field, NHibernate Mapping-By-Code"
description: "A tutorial on how to integrate an IUserType implementation with an Id field using NHibernate's Mapping-By-Code 
mapping syntax."
comments: true
---
I couldn't find any documentation on how to do this on the [interwebs](http://www.urbandictionary.com/define.php?term=interwebs" "interwebs"),
so I've chucked it up here. This is mostly so I can revert to it when I do need it again and have almost certainly
forgotten how to do it!

I've used the ComposedId property of NHibernate Mapping-By-Code. This also requires you to override the Equals and
GetHashCode methods on the entity class, as the ComposedId property is generally used to represent composite keys of a
database table.

I have excluded the implementation of the IUserType classes to keep the post short. In case you wanted to know how to
implement an IUserType, check out my [post on it](/posts/nhibernate-mapping-by-code-and-iusertypes).

{% highlight csharp linenos %}
public class ExampleEntity
{
    public virtual String MyIdColumn { get; set; }
    public virtual Country Country { get; set; }
	
	public override bool Equals(object obj)
    {
        var exampleEntity = obj as ExampleEntity;
        if (exampleEntity != null)
        {
            return exampleEntity.MyIdColumn == MyIdColumn;
        }
        
        return false;
    }
	
	public override int GetHashCode()
    {
        return MyIdColumn.GetHashCode();
    }
}

public class ExampleEntityMap : ClassMapping
{
    public ExampleEntityMap()
    {
        Table("Table");

        ComposedId(i => i.Property(p => p.MyIdColumn, map =>
            {
                map.Column("MyIdColumn");
                map.Type<MyIdColumnUserType>();
            }));

        Property(i => i.Country, map =>
            {
                map.Column("Country");
                map.Type<CountryEnumUserType>();
            });
    }
}

public enum Country
{
	Australia,
	Sri Lanka,
	Brazil,
	Spain
}
{% endhighlight %}

Hope it helps someone else as well!
