---
layout: post
title: NHibernate SessionFactory Template
categories:
- NHibernate
tags:
- Mapping-By-Code
- NHibernate
- SessionFactory
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p>After having to come up with this boilerplate code over and over again, I have decided to write it up and put it on <a href="https://github.com/pwee167/NHibernateTemplate" target="_blank">GitHub</a>.  The code will allow you to build a Nhibernate Sessionfactory given a connection string to a database.</p>

```csharp
using System;
using System.Data;
using NHibernate;
using NHibernate.Cfg;
using NHibernate.Dialect;
using NHibernate.Driver;
using NHibernate.Mapping.ByCode;
using System.Reflection;
using System.Diagnostics.Contracts;
namespace NHibernateTemplate
{
    public class SessionFactoryFactory : ISessionFactoryFactory
    {
        private ISessionFactory _sessionFactory;
        private readonly string _connectionString;
		
		public SessionFactoryFactory(string connectionString)
        {
            Contract.Assume(!String.IsNullOrEmpty(connectionString), "Connection string cannot be null or empty.");
            _connectionString = connectionString;
        }
		
		private ISessionFactory BuildSessionFactory()
        {
            var mapper = new ModelMapper();
            var configuration = new Configuration();
			mapper.AddMappings(Assembly.GetExecutingAssembly().GetExportedTypes());
			configuration.DataBaseIntegration(c =>
            {
                c.ConnectionString = _connectionString;
                c.IsolationLevel = IsolationLevel.ReadCommitted;
                c.Driver();
                c.Dialect();
                c.BatchSize = 50;
                c.Timeout = 30;
#if DEBUG
                c.LogSqlInConsole = true;
                c.LogFormattedSql = true;
                c.AutoCommentSql = true;
#endif
            });
#if DEBUG
            configuration.SessionFactory().GenerateStatistics();
#endif
			configuration.AddMapping(mapper.CompileMappingForAllExplicitlyAddedEntities());
            var sessionFactory = configuration.BuildSessionFactory();
          
			return sessionFactory;
        }
		
		public ISessionFactory Instance
        {
            get
            {
                if (_sessionFactory == null)
                {
                    _sessionFactory = BuildSessionFactory();
                }
              return _sessionFactory;
            }
        }
    }
}
```