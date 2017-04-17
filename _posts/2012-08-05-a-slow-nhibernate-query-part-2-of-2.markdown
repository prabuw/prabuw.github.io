---
layout: post
title: "A slow NHibernate query (Part 2 of 2)"
date: 2012-08-05
categories: [NHibernate]
keywords: "NHibernate, Slow NHibernate query, Ansistring"
description: "A slow NHibernate query I came across, root cause of it and how I finally the got to the solution."
comments: true
---
Following on from the [original post](/posts/a-slow-nhibernate-query-part-1-of-2/ "original post"), I started using
NHibernate's mapping-by-code to map a database table to my POCO. Some background to the problem:

- The type of database is Oracle 10g.
- It's not hosted on my local machine (i.e. some server away from me).
- I am using the latest version NHibernate (3.3.1.4) from NuGet.

Here is the mapping code:

``` csharp
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class Person
    {
        public virtual string PersonId { get; set; }
        public virtual string FirstName { get; set; }
        public virtual string LastName { get; set; }
    }

    public class PersonMap : ClassMapping<Person>
    {
        public PersonMap()
        {
            Schema("MyDB");
            Table("Person");
            Id(i => i.PersonId);
            Property(i => i.FirstName);
            Property(i => i.LastName);
        }
    }
}
```

Here is the code used to connect to the build the session factory, open the session and execute the query:

``` csharp
using System;
using System.Diagnostics;
using System.Reflection;
using NHibernate.Cfg;
using NHibernate.Dialect;
using NHibernate.Driver;
using NHibernate.Mapping.ByCode;

namespace Learning.NHibernate
{
    class Program
    {
        static void Main(string[] args)
        {
            var stopwatch = new Stopwatch();

            var mapper = new ModelMapper();
            var cfg = new Configuration();

            mapper.AddMappings(Assembly.GetExecutingAssembly().GetExportedTypes());

            cfg.DataBaseIntegration(c =>
            {
                c.ConnectionString = @"User Id=user;Password=password;Data Source=MyDB;";
                c.Driver<OracleClientDriver>();
                c.Dialect<Oracle10gDialect>();

                c.LogSqlInConsole = true;
                c.LogFormattedSql = true;
                c.AutoCommentSql = true;
            });

            cfg.AddMapping(mapper.CompileMappingForAllExplicitlyAddedEntities());
            var sessionFactory = cfg.BuildSessionFactory();

            stopwatch.Stop();
            Console.WriteLine("Setting up: {0}", stopwatch.ElapsedMilliseconds);
            stopwatch.Restart();

            Person entity = null;

            using (var session = sessionFactory.OpenSession())
            using (var tx = session.BeginTransaction())
            {
                stopwatch.Stop();
                Console.WriteLine("Opening session: {0}", stopwatch.ElapsedMilliseconds);
                stopwatch.Restart();

                entity = session.Get<Person>("1");

                stopwatch.Stop();
                Console.WriteLine("Retrieving data: {0}", stopwatch.ElapsedMilliseconds);
                stopwatch.Restart();

                tx.Commit();
            }

            stopwatch.Stop();
            Console.WriteLine("Finish: {0}", stopwatch.ElapsedMilliseconds);           
        }
    }
}
```

Pretty straight forward&nbsp; stuff here. Even with the overhead of an ORM, this should be very fast. It's taking
roughly *80-100 seconds* to get accessed to the retrieved object!

### Debugging the problem
So I begin to investigate the situation, first up the generated query:

``` sql
SELECT person0_.PersonId as PersonId0_0_, 
person0_.FirstName as FirstName0_0_, person0_.LastName as LastName0_0_,  
FROM MyDB.Person person0_
WHERE person0_.PersonId=:p0;
:p0 = '1' 
```

This looked normal to me. So the checklist goes as follows:

+ The **PersonId** column is indexed, in fact it is the primary key.
+ I ran the query via PL/SQL Developer and it took around 32ms.
+ The query plan on the query confirmed the use of a index unique scan and not a dreaded full table scan.
+ I ran the un-parameterized query via ADO.Net and it took roughly 180ms.
+ I got [NHProf](http://nhprof.com/ "NHProf") (trial version) to profile my code and got the following:

<div class="centered">
    <img src="/images/nhprof.png"  alt="Profiling results" />
</div>

It felt like the query gets executed and the results are returned, but something else was taking up the time. Finally,
I decided it was time to get the [source to NHibernate](https://github.com/nhibernate/nhibernate-core "source to NHibernate")
and step through it. The execution inside NHibernate is moving along nicely until it hits this line:

``` csharp
for (count = 0; count < maxRows && rs.Read(); count++)   
```

Specifically, the code that holds up the execution is `rs.Read()`, the oracle datareader. Why? Still stuck, I struck
some luck and found someone with the same problem, and who blogged about it [here](http://www.gitshah.com/2009/10/issue-with-systemdataoracleclient-and.html)

### What's the issue?
Nhibernate uses ADO.Net's [parameterised queries](http://msdn.microsoft.com/en-us/library/yy6y35y8.aspx "parameterised queries")
The data type of a parameter is specific to the data provider, in my case: *System.Data.OracleClient*. The issue lies
with the database type associated with the parameter in the parameterised query.

+ I have defined the **PersonId** column in the NHibernate mapping as a .NET Type of **String**.
+ NHibernate determines (via reflection in the case of mapping-by-code) the **[DbType](http://msdn.microsoft.com/en-us/library/system.data.dbtype.aspx "DbType") of PersonId**
  to be of **String**, a type representing Unicode characters.
+ This translates to a database type of **NVarchar or Char**.
+ However, **PersonId** is of type **Varchar2**, which is based on the ANSI standard.

I believe this mismatch results in a full table scan, rather than an index scan. This probably is not as noticeable on
a smaller sized table, but when the size of the table is larger (~3 millions records), the performance deteriorates
significantly.

### Solution
Firstly, I updated to [Oracle's .NET data provider](http://www.oracle.com/technetwork/topics/dotnet/index-085163.html "Oracle's .NET data provider"),
including installing [ODAC](http://www.oracle.com/technetwork/developer-tools/visual-studio/downloads/index.html "Oracle Data Access Components")
(Oracle Data Access Components). While this had no bearing on resolving the issue, it is recommended to do so as
[System.Data.OracleClient](http://msdn.microsoft.com/en-us/library/system.data.oracleclient.aspx "System.Data.OracleClient")
has been deprecated. The solution that finally resolved the problem was to explicitly set the type of the property to
**Ansistring**, as seen in the code below.

``` csharp
using NHibernate;
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class PersonMapping : ClassMapping<Person>
    {
        public PersonMapping()
        {
            Schema("dbo");
            Table("Person");
            Property(x => x.Id, m => m.Type(NHibernateUtil.AnsiString));
            Property(x => x.FirstName);
            Property(x => x.LastName);
        }
    }
}
```

In conclusion, if your database contains non-Unicode columns and you are using NHibernate as your ORM of choice,
remember to specify the type as **Ansistring** in the NHibernate mapping.

