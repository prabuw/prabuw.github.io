---
layout: post
title: 'NHibernate Mapping By Code : Mapping relationships'
categories:
- NHibernate
tags:
- Mapping-By-Code
- NHibernate
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
To set the scene for the code, here is a diagram of the database.

<div class="centered">
    <img src="/images/database.png"  alt="schema" style="width: 640px;" />
</div>

The relationships in the above diagram are as follows:

+ One to Many (i.e. A country has many clubs but a club belongs to only one country)
+ One to One (i.e. A club has one club detail and a club detail extends only one club)
+ Many to Many (i.e. A club has many sponsors and a sponsor works with many clubs)

```csharp
using System.Collections;
using System.Collections.Generic;
using NHibernate.Mapping.ByCode;
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class Club
    {
        public virtual int Id { get; set; }
        public virtual string Name { get; set; }

        public virtual Country Country { get; set; }
        public virtual ClubDetail ClubDetail { get; set; }
        public virtual ICollection<Sponsor> Sponsors { get; set; }
    }
       
    public class ClubMapping : ClassMapping<Club>
    {
        public ClubMapping()
        {
            Schema("dbo");
            Table("Club");

            Id(i => i.Id);            
            Property(x => x.Name);

            ManyToOne(x => x.Country, x =>
            {
                x.Column("CountryId");
            });

            OneToOne(x => x.ClubDetail, mapper =>
            {
                mapper.PropertyReference(typeof(ClubDetail).GetProperty("Id"));
            });

            Set(x => x.Sponsors, collectionMapping =>
            {
                collectionMapping.Table("ClubSponsor");
                collectionMapping.Key(map => map.Column("ClubId"));                  
            },
            map => map.ManyToMany(p => p.Column("SponsorId")));
        }
    }
}
```

```csharp
using System.Collections;
using System.Collections.Generic;
using NHibernate.Mapping.ByCode;
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class Country
    {
        public virtual int Id { get; set; }
        public virtual string AbbreviatedName { get; set; }  
        public virtual string Name { get; set; }
        public virtual ICollection<Club> Clubs { get; set; }
    }
       
    public class CountryMapping : ClassMapping<Country>
    {
        public CountryMapping()
        {
            Schema("dbo");
            Table("Country");

            Id(i => i.Id);

            Property(x => x.AbbreviatedName);         
            Property(x => x.Name);

            Set(x => x.Clubs, mapping =>
            {
                mapping.Key(k =>
                {
                    k.Column("CountryId");
                });
                mapping.Inverse(true);                
            },
            r => r.OneToMany());
        }
    }
}

```

```csharp
using System.Collections;
using System.Collections.Generic;
using NHibernate.Mapping.ByCode;
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class ClubDetail
    {
        public virtual int Id { get; set; }
        public virtual string AbbreviatedName { get; set; }
        public virtual string WebsiteURL { get; set; }        
        public virtual Club Club { get; set; }

        public override bool Equals(object obj)
        {
            if (ReferenceEquals(null, obj))
                return false;

            if (ReferenceEquals(this, obj))
                return true;

            var a = obj as ClubDetail;
            if (a == null)
                return false;

            return a.Club.Id == Club.Id;
        }

        public override int GetHashCode()
        {
            unchecked
            {
                var hash = 21;
                hash = hash * 37 + Club.Id.GetHashCode();

                return hash;
            }
        }
    }
       
    public class ClubDetailMapping : ClassMapping<ClubDetail>
    {
        public ClubDetailMapping()
        {
            Schema("dbo");
            Table("ClubDetail");

            ComposedId(i => i.ManyToOne(x => x.Club, mapper =>
            {
                mapper.Column("Id");
                mapper.ForeignKey("FK_ClubDetail_Club");
            }));

            Property(x => x.Id);
            Property(x => x.AbbreviatedName);
            Property(x => x.WebsiteURL);         
        }
    }
}

```

```csharp
using System.Collections;
using System.Collections.Generic;
using NHibernate.Mapping.ByCode;
using NHibernate.Mapping.ByCode.Conformist;

namespace Learning.NHibernate
{
    public class Sponsor
    {
        public virtual int Id { get; set; }
        public virtual string Name { get; set; }
        public virtual ICollection<Club> Clubs { get; set; }
    }

    public class SponsorMapping : ClassMapping<Sponsor>
    {
        public SponsorMapping()
        {
            Schema("dbo");
            Table("Sponsor");

            Id(i => i.Id);

            Property(x => x.Name);

            Set(x => x.Clubs, collectionMapping =>
            {
                collectionMapping.Table("ClubSponsor");
                collectionMapping.Key(map => map.Column("SponsorId"));
            },
            map => map.ManyToMany(p => p.Column("ClubId")));
        }
    }
}

```
