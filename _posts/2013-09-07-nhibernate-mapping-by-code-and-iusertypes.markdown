---
layout: post
title: "NHibernate, Mapping-By-Code and IUserType"
date: 2013-09-07
categories: [NHibernate]
keywords: "NHibernate, NHibernate Mapping-by-Code,IUserType, Custom NHibernate Mapping, Extending NHibernate, "
description: "An example on how to implement NHibernate's IUserType interface to extend NHibernate's default mapping. In 
this example, the custom mapping will involve trimming whitespace from string fields coming from a database."
comments: true
---
IUserType is an extension point in NHibernate that allows you to handle the mapping process yourself. This is useful for
certain scenarios such as trimming strings or mapping a website's URL stored as a string in the database to a
[System.Uri](http://msdn.microsoft.com/en-us/library/system.uri.aspx "System.Uri") object in your model.

I've included the code needed to achieve the examples mentioned above (TrimmedString and UriType) below.

``` csharp
using System;
using NHibernate.Mapping.ByCode.Conformist;

namespace ConsoleApplication1
{
    public class Person
    {
         public virtual int Id { get; set; }
         public virtual string FirstName { get; set; } 
        public virtual string LastName { get; set; }
        public virtual Uri Website { get; set; }
    }

    public class PersonMap : ClassMapping<Person>
    {
        public PersonMap()
        {
            Table("Person");
            Id(i => i.Id);
            Property(i => i.FirstName, map => map.Type<TrimmedString>());
            Property(i => i.LastName);
            Property(i => i.Website, map => map.Type<UriType>());
        }
    }
}
```

``` csharp
using NHibernate;
using NHibernate.SqlTypes;
using NHibernate.UserTypes;
using System;
using System.Data;

namespace ConsoleApplication1
{
    public class TrimmedString : IUserType
    {
        public object NullSafeGet(IDataReader rs, string[] names, object owner)
        {
            var resultString = (string) NHibernateUtil.String.NullSafeGet(rs, names[0]);
            return resultString != null ? resultString.Trim() : null;
        }

        public void NullSafeSet(IDbCommand cmd, object value, int index)
        {
            if (value == null)
            {
                NHibernateUtil.String.NullSafeSet(cmd, null, index);
                return;
            }

            value = ((string) value).Trim();

            NHibernateUtil.String.NullSafeSet(cmd, value, index);
        }

        public object DeepCopy(object value)
        {
            return value == null ? null : string.Copy((String) value);
        }

        public object Replace(object original, object target, object owner)
        {
            return original;
        }

        public object Assemble(object cached, object owner)
        {
            return DeepCopy(cached);
        }

        public object Disassemble(object value)
        {
            return DeepCopy(value);
        }

        public SqlType[] SqlTypes
        {
            get
            {
                var types = new SqlType[1];
                types[0] = new SqlType(DbType.String);

                return types;
            }
        }

        public Type ReturnedType
        {
            get { return typeof (String); }
        }

        public bool IsMutable
        {
            get { return false; }
        }

        public new bool Equals(object x, object y)
        {
            if (ReferenceEquals(x, y)) return true;

            var xString = x as string;
            var yString = y as string;
            
            if (xString == null || yString == null) return false;

            return xString.Equals(yString);
        }

        public int GetHashCode(object x)
        {
            return x.GetHashCode();
        }
    }
}
```

``` csharp
using NHibernate;
using NHibernate.SqlTypes;
using NHibernate.UserTypes;
using System;
using System.Data;

namespace ConsoleApplication1
{
    public class UriType : IUserType
    {
        public object NullSafeGet(IDataReader rs, string[] names, object owner)
        {
            var resultString = (string)NHibernateUtil.String.NullSafeGet(rs, names[0]);

            if (!String.IsNullOrEmpty(resultString) &amp;&amp; !String.IsNullOrWhiteSpace(resultString) &amp;&amp; Uri.IsWellFormedUriString(resultString.Trim(), UriKind.RelativeOrAbsolute))
                return new Uri(resultString.Trim());

            return null;
        }

        public void NullSafeSet(IDbCommand cmd, object value, int index)
        {
            if (value == null)
            {
                NHibernateUtil.String.NullSafeSet(cmd, null, index);
                return;
            }

            value = ((string)value).Trim();

            NHibernateUtil.String.NullSafeSet(cmd, value, index);
        }

        public object DeepCopy(object value)
        {
            return value == null ? null : new Uri(value.ToString());
        }

        public object Replace(object original, object target, object owner)
        {
            return original;
        }

        public object Assemble(object cached, object owner)
        {
            return DeepCopy(cached);
        }

        public object Disassemble(object value)
        {
            return DeepCopy(value);
        }

        public SqlType[] SqlTypes
        {
            get
            {
                var types = new SqlType[1];
                types[0] = new SqlType(DbType.String);

                return types;
            }
        }

        public Type ReturnedType
        {
            get { return typeof(String); }
        }

        public bool IsMutable
        {
            get { return false; }
        }

        public new bool Equals(object x, object y)
        {
            if (ReferenceEquals(x, y)) return true;

            var xString = x as string;
            var yString = y as string;
            
            if (xString == null || yString == null) return false;

            return xString.Equals(yString);
        }

        public int GetHashCode(object x)
        {
            return x.GetHashCode();
        }
    }
}
```

Hope this helps!
