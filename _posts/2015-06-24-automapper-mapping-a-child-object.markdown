---
layout: post
title: "Automapper: Mapping a child object"
date: 2015-06-24
categories: [Automapper]
keywords: "Automapper, Setup guide, configuration"
description: "Mapping a child object with Automapper"
comments: true
---
It's been a while since I've posted something here, and so I'm going to post something small to get my feet wet
again. 

I started using Automapper again after a while away from it. My simple goal was to map an object with a 
child object in it, which left me stumped!

###  First some boiler plate code to set the scene.

My source: let's pretend they are domain objects.

``` csharp
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime DateOfBirth { get; set; }
    public Address Address { get; set; }
}

public class Address
{
    public int HouseNumber { get; set; }
    public string StreetName { get; set; }
    public string Suburb { get; set; }
    public State State { get; set; }
}
```

My destination: Let's pretend they are my DTOs.

``` csharp
public class PersonViewModel
{
    public string Name { get; set; }
    public DateTime Birthday { get; set; }
    public AddressViewModel Address { get; set; }
}

public class AddressViewModel
{
    public int Number { get; set; }
    public string StreetName { get; set; }
    public string Suburb { get; set; }
    public State State { get; set; }
}
```


###  Setting up the mapping

``` csharp
public static class Mappings
{
    public static void Configure()
    {
        Mapper.CreateMap<Address, AddressViewModel>()
            .ForMember(dest => dest.Number, opt => opt.MapFrom(src => src.HouseNumber));

        Mapper.CreateMap<Person, PersonViewModel>()
            .ForMember(dest => dest.Name, opt => opt.MapFrom(src => String.Join(" ", src.FirstName, src.LastName)))
            .ForMember(dest => dest.Birthday, opt => opt.MapFrom(src => src.DateOfBirth))
            .ForMember(dest => dest.Address, opt => opt.MapFrom(src => Mapper.Map<Address, AddressViewModel>(src.Address)));
    }
}
```

Once I've defined the mapping for the Address/AddressViewModel pair, I can call that in the mapping of the address property in the parent object. 
The line that achieves this is:

``` csharp
.ForMember(dest => dest.Address, opt => opt.MapFrom(src => Mapper.Map<Address, AddressViewModel>(src.Address)))
```

Boom - simple!