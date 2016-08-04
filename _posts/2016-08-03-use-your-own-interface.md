---
published: true
layout: post
title: use your own interface
description: thoughts for abstracting out 3rd party dependencies
date: 2016-08-03
tags: [framework, architecture, dotnet]
comments: false
---

Should you use other peoples interfaces, what is the impact of doing this? consider the ASP.NET IDependencyResolver, or Caslte's ILog.

**NOTE** a question which you should bear in mind, if your services are small enough do you even care, as you could just as well write the service from scratch in another language

**NOTE 2** all the libraries/Frameworks I mention are excellent, i am not saying they are not.

## TL;DR;

the following are the points to consider.

- be careful of using a nice interface in a 3rd party app directly, as it will bind you to the library (IE ASP.NET IDependencyResolver used in a windows service, does not sound correct)
- 3rd party libraries have a shelf life (coupling on them has a level of risk)
- consider what you abstract (apply an interface to), IE logger is easy, ORM is not
- if you cannot easily abstract, then encapsulate the functionality
- consider creating your own copy of an interface, which you can manage
- if you have a number of interfaces consider a separate solution to store them in

## Interfaces in 3rd party libs

Through out using .NET I have come across libraries which come into and out-of favor. Each of these libraries offered interfaces and their concrete implementations.

Examples of libraries with interfaces which appear to be helpful

- Asp.Net [IDependencyResolver](https://msdn.microsoft.com/en-us/library/system.web.mvc.idependencyresolver(v=vs.118).aspx)
- Castle [ILogger](https://github.com/castleproject/Windsor/blob/master/docs/logging-facility.md)
- MassTransit [IBus](https://github.com/MassTransit/MassTransit/blob/develop/src/MassTransit/IBus.cs)
- NHibernate [ISession](https://github.com/nhibernate/nhibernate-core/blob/master/src/NHibernate/ISession.cs)

Out of the above Castle's ILogger is probably the best, as it does not tie me/us down to a Logger implementation, I can use NLog or Log4Net.

Having said that, you may know that using any of the above will couple you to that library and in some cases the interface is not suitable for other implementations.

## The definition of the interface

The interface is important to be correct abstraction (mostly can-do, not is-a), in some cases this is very easy, but others this can be a world of pain. in any case we have to ask if it is worth abstracting an implementation or not.

An easy interface is logging, the following should be easy enough to create a Log4Net or NLogger implementation.

```
public interface ILogger
{
    void Log(Func<string> message, LogLevel level);
    void Log(Func<Exception> exception, LogLevel level);
}
```

A harder interface would be data-access, and this gets a fair bit of slating from people who have tried, [a good post on that](https://lostechies.com/jimmybogard/2012/09/20/limiting-your-abstractions/).

In the latter case I prefer to encapsulate the complex parts entirely. If we take the querying part of data access, we can use a query dispatcher which would call a query handler allowing us to encapsulate the query implementation.

```
//querying for the default plan. under the covers this could use ADO, Elastic
var defaultPlan = _queryDispatcher.Query<ByDefaultPlan, Plan>(new ByDefaultPlan());
```

## Coupling

One point of Object Oriented software is loose coupling. We can achieve this via several means.

In the case of using the *IDependencyResolver* from ASP.NET, the interface looks correct and i want to use it in my QueryDispatcher (for example). If I use the ASP.NET interface, I have coupled my codebase to ASP.NET, which is not ideal. Consider we may have a service which only reacts to queues, it does not require ASP.NET dlls.

By supplying our own *IDependencyResolver* we can depend on that one instead through out other parts of the application, and easily supply implementation wrappers/adaptors when we need them.

An advantage of supply your own interface it provides a route to replace an implemetation, with minimal refactor of existing code. for example I have a QueryDispatcher, which depends on IDependencyResolver (my copy), which allows me to change from StructureMap to Windsor to whatever.

```
public class QueryDispatcher : IQueryDispatcher
{
    private readonly IDependencyResolver _resolver;

    public QueryDispatcher(IDependencyResolver dependencyResolver)
    {
        _resolver = dependencyResolver;
    }

    public virtual TResult Query<TMessage, TResult>(TMessage message) where TMessage : class
    {
        var query = _resolver.Resolve<IQuery<TMessage,TResult>>();
        try
        {
            query.Execute(message);
            return query.Result;
        }
        finally
        {
            _resolver.Release(query);
        }
    }
}
```

the code is not dependent on ASP.NET or Windsor.

This tries to mitigate a scenario where a dependency goes out of favor (Stagnant).

## Core Interfaces project(s)

**NOTE** a reminder if our service impl is small enough we may not care, and just use the dependencies directly.

If we look at our application we want to have access to multiple libraries (Bus/messaging, Data-access, Logging, Cache, Monitoring, Commands, etc). and the interfaces to these services will be used through the code-base. if we have many services (soluitons) then it may be worth placing these interfaces into a core library.

Then for each implementation of an interface create a separate dll (project) which impalements a given interface(s). these implemented dll;s can then be setup in your applications Bootstrap. I breif mentioned this before [here](http://dbones.github.io/2016/07/dotnet-three-project-setup/).


for example using the query dispatcher and dependencyResolver, here is a possible project outline

```
Epic.Core                   (dll/proj)
    Messaging               (folder)
        IBus.cs
    Querying                (folder)
        IQueryDispatcher.cs
    Ioc                     (folder)
        IDependencyResolver.cs

Epic.Messaging.Default      (dll)
    Bus.cs

Epic.Messaging.EasyNetQ     (dll)
    EasyNetQBus.cs

Epic.Querying.Default       (dll)
    QueryDispatcher.cs

Epic.Ioc.Windsor            (dll)
    WindsorDependencyResolver.cs

Epic.Ioc.AutoFac            (dll)
    AutoFacDependencyResolver.cs
```

here we have 2 Ioc which can be used (note I do not abstract the setup of the container, as it is normally limited to the host project).

normally only one implemetation is registered in with the container.

note that the Core contains all the interfaces used by the Core project of your service (again using the 3 project terminology) 

if you want Core can be split into multiple dlls (I have done this as some interfaces are not required by all applications)

also I use *"Default"* to suggest that the impl is custom coded.
