---
layout: post
title: Using an IUserType with an Id field in NHibernate with Mapping-By-Code
categories:
- NHibernate
tags:
- IUserType
- Mapping-By-Code
- NHibernate
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p align="justify">I couldn't find any documentation on how to do this on the <a href="http://www.urbandictionary.com/define.php?term=interwebs" target="_blank">interwebs</a>, so I've chucked it up here. This is mostly so I can revert to it when I do need it again and have almost certainly forgotten how to do it!</p>
<p align="justify">I've used the ComposedId property of NHibernate Mapping-By-Code. This also requires you to override the Equals and GetHashCode methods on the entity class, as the ComposedId property is generally used to represent composite keys of a database table,</p>
<p align="justify">I have excluded the implementation of the IUserType classes to keep the post short, In&nbsp;case you wanted to know how to implement an IUserType, check out my <a href="http://pwee167.wordpress.com/2013/09/07/nhibernate-mapping-by-code-and-iusertypes/" target="_blank">post on it</a>.</p>

```csharp
public class ExampleEntity
{
    public virtual String MyIdColumn { get; set; }
    public virtual Country Country { get; set; }
	
	public override bool Equals(object obj)
    {
        var exampleEntity = obj as ExampleEntity;
        if (exampleEntity != null)
            return exampleEntity.MyIdColumn == MyIdColumn ;
		
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
```

<p align="justify">Hope it helps someone else as well!</p>
