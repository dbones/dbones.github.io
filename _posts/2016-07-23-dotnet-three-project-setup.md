---
published: true
layout: post
title: dotnet three project setup
description: my typical project setup
date: 2016-07-23
tags: [microservice, architecture, dotnet]
comments: false
---

This post has a few notes on a .NET application, this does **not** mean the way you do it is right or wrong, its just how I like to do it.

if you are looking for an out the box solution have a look at sharp-architecture, or Paramore

note I will not cover Event Sourcing or CQRS here. this is upto you if you implement it or not. everything here works with(out) either.

## TL;DR;
- 3 projects per service
- **Core** contains Ports and Adapters
- **Dto** contains all classes which are shared outside the boundary context
- **Host** setups and runs the Core, along with additional Cross Cutting and Software Quality components
- i cheat and have a few classes and interfaces in my own framework (you could easily replicate with something like sharp-architecture)

# Service using 3 projects 

My solutions are made of many services (not quite a microservice), each responsible for a small part of the solution/platform.

each service I construct using 3 .NET projects (I could do it in 2, but I like 3) as follows

- [Service].Core
- [Service].Dto
- [Service].Host

this is still high level, and does not tie down the structure inside each project.

for each of these projects I have a mirror project called Common (going to renamed shortly) in my framework, which contains classes which are shared in my services, for example Service A and Service B, both will have a Host project, and both will make use of a class called App. so Common.Host will contain that type of class.

<figure>
	<a href="http://dbones.github.io/images/posts/2016/ms-project-struct/dotnet-3-project-structure.png"><img src="http://dbones.github.io/images/posts/2016/ms-project-struct/dotnet-3-project-structure.png" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2016/ms-project-struct/dotnet-3-project-structure.png" title="2 services">2 services</a>.</figcaption>
</figure>

you can see above that the sync service depends on the security.dto, in this case it subscribes to one of its events.

## Core Project

Loosely based on a hexagonal architecture, the core contains the domain classes, ports (commands and queries) and adopters (consumers, controlllers).

### Ports

ports to me are the protocol, the logic, the api of what the service can do. I am also a large fan of Command Query Segregation

- commands are classes which mutate state and publish events. 
- queries encapsulate a query.

I allow commands to use queries.

### Adapters

adapters, adapt the input to invoke commands or queries (ports)

- consumers, subscribe to events (messages) and invoke commands.
- controllers, normally http (note I use nancyfx or asp.net mvc directly), they adapt the http request message and call commands or queries as necessary before responding.
- responders, used to apply Request / Response over a bus, works the same way as controllers.

what about the other adapters such as the database, and etc? well the 3 above provide the means to invoke ports from another process. There are a number of other adaptors but these are invoked inside the quieries or adaptors.

The core project depends on interfaces such as ISession, or IBus, and not on concrete implementations. To enable this I have my own base framework, which provides both the interfaces and offers several implementations, such as IBus interface and 2 implementations, MemoryBus and EasyNetQBus.

### Models

models are classes which represent the domain classes, mainly entities which are persisted.

### typical structure

I like to try and structure each of my services the same way.  

```
Adaptors
    Controllers
    Consumers
    Responders
Ports
    Commands
    Queries
Models
```

## Dto Project

Any message definition which is shared outside the boundary context of the service is placed in here. These classes are shared with other services.

- Events are messages which provide context of something which has happened, past tense. **Consumers** subscribe to these events.
- Resources are classes which description HTTP Resources, I really like the Rancher Specification for these. **Controllers** use these.
- Requests are classes classes for Request and Resources over the bus. **Responders** use these.

In addition to events, I have an empty interface (used as a meta data marker) called IDomainEvent, which all events implement. this allows me to consume any message which implement it very simply for auditing.

All messages (events, requests) inherit a base message class, which contains some generic context data about the message.

## Host Project

Where you host your (micro)service, is it a windows service, IIS application, console line etc. Where it is hosted must not change any of your Core, or Dto projects. so i keep it separated. I use the last one, and run this in a docker container.

The host project is required to setup all the components (registering them into the IoC Container in my case), note that in IIS you have per-web-request, in a console app you do not

It should also starts the components which need to be started, and on close it needs to dispose of the components accordingly. 

this project normally contains

- Program - load config and call bootstrap
- Bootstrap - register all components with the IoC container 
- Configure - classes which can setup other registered components, such as Mapping, Index, etc (my implementation im currently reviewing)

I use castle Windsor, and the bootstrap calls a number of installers, which include support for a number of cross cutting concerns.

as i use Windsor, I take advantage of its IStartable, and Factory functionality.

# Cross cutting concerns and software qualities

Wait, there are a few things which are missing!

- logging - logging service
- context - apply to message or (http) request/response 
- database - setup, unit of work
- health check
- App Config - from file, environment etc, and overriding. 
- Code Config
- AOP components
- Cron tasks

and a few others, of which are still within the context of the code inside the process. If we expand this back to the solution level, such as Audit, service discovery etc.

From the list above, I have framework classes, which are setup in the host project. well except the cron task, that I use a small cron service.
