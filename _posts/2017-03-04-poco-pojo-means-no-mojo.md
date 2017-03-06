---
published: true
layout: post
title: Poco, Pojo means no Mojo.
description: what does Pojo and Poco mean to you?
date: 2017-03-04
tags: [architecture, dotnet, java]
comments: false
---

Off the shelf frameworks are there to help you, however using them has a cost, aka Technical Debt, the cost of dependency.

To reduce this dependency we look for frameworks which promote Pojo's/Poco's. Some frameworks say they achieve this however they do not.

This article looks at approaches which reduce the level of dependency.

**NOTE:** sometimes we do not care about a direct dependency, coupling, ie we have encapsulated the area, or we have a road map which has tactical (short term) and strategic (long term) goals

**NOTE:** Im not saying any framework is bad, however pointing out a way to further decouple your classes.

## TL;DR;

- Poco - Plan old CLR object.
- Pojo - Plan Old Java Object.

An object which is unaware of any framework around it, except the base/reference framework or required dependencies, which can be injected in.

**Meta/config data methods, which will be used by frameworks**

- Attributes - the Poco is not ignorant of the framework = debt
- XML, Yaml setup - Poco is ignorant, all is good, but not parsed by a Compiler, consider typo's are caught at runtime, or by good IDE's (consider CI servers though).
- Fluent code setup -  Poco is ignorant, all is good, and in most cases its strongly typed and compiler parsed.
- Sensible Conventions - Poco is ignorant, there is no extra setup code, all is good. The side issue is if the conventions are not transferable between supporting frameworks.

**cross cutting concerns**

- Proxy/AOP - Poco will be ignorant of additional applied logic, all is good
- Middleware - Poco will be ignorant of additional applied logic, however it may be aware of the middleware framework

more on that over here: http://dbones.github.io/2017/02/Middleware-is-the-new-aop/

# Quick definition

The definition of POCO/POJO, used here, is any class which:

- does not use annotation/attributes.
- does not have to extend/inherit from a base class 
- does not have to implement a class

I think this should be similar to the wikipedia definition.

if your class has a dependency injected which it is uses to achieve a task, this is ok (its SOLID OO, pun intended).

If you use an attribute from the base framework, **strictly this is considered not correct**. *I am ok with some exceptions to this rule* IE .NET's "System.ComponentModel.DataAnnotations", mainly the validation ones, as validaion logic belongs to the class, these annotations just clean up the code.

# Quick overview

the **goal** is to have classes which are dependent only on other classes which are required in order to do its task.

Lets take 2 examples into account:

- A service/command class
- A simple entity class (with only properties on it).

In both cases we will look at 2 different ways, to see how we can reduce the dependencies and make the code cleaner (closer to a Poco).

# Service - IoC dependency

**Goal:** that the IoC container should be invisible to the core classes of application, ie services, controllers, commands, queries etc.

<figure>
	<a href="http://dbones.github.io/images/posts/2017/poco-pojo-mojo/sevice-attribute-bean.PNG"><img src="http://dbones.github.io/images/posts/2017/poco-pojo-mojo/sevice-attribute-bean.PNG" /></a>
    <figcaption><a href="http://dbones.github.io/images/posts/2017/poco-pojo-mojo/sevice-attribute-bean.PNG" title="setup">(left) IoC aware, (right) not IoC aware.</a>.</figcaption>
</figure>


### Left

On the left we have an Attribute/Annotation, which indicates to the the IoC container what to inject on. In this case the **ISession** on the **Constructor**

### Right

Simply removing the **[inject]** we decouple the code from the IoC container.

The example on the right, we take advantage of the IoC container's defult conventions. Where the container will select the Constructor which it can inject the most dependencies on.

# Entity - Persistence dependency 

**Goal:** entity classes should not be aware of persistence. Arguably it should not know of serialisation, or any mapping.

<figure>
	<a href="http://dbones.github.io/images/posts/2017/poco-pojo-mojo/entity-attribute-vs-pojo.PNG"><img src="http://dbones.github.io/images/posts/2017/poco-pojo-mojo/entity-attribute-vs-pojo.PNG" /></a>
    <figcaption><a href="http://dbones.github.io/images/posts/2017/poco-pojo-mojo/entity-attribute-vs-pojo.PNG" title="entity">(left)persitence attributes (right) closer to a Poco</a>.</figcaption>
</figure>


### Left

On the left we have attributes to help an ORM to understand how to persist this entity

### Right

By removing these *persistence* attributes, our entity becomes persist ignorant (which is good).

For the code to work on the right, we *should* be able to do nothing in addition, as long as we are ok with the default conventions of that ORM (ie Fluent NHibernate or Entity framework). 

If we wanted to provide more meta data, then we will need to provide a Class Map (which can be in code, and therefor refactoring tools will support).

**NOTE:** Strictly the *[Required]* breaks the rules, however I am normally ok with this one as this does not add a further dependency, as the attribute is a standard one from .NET.

**NOTE** we could move all the transaction code to AOP or Middleware action.

# Quick observations

In both examples we remove unnecessary from the classes, meaning our classes uphold:

- Separation of concerns
- SOLID
- Loose coupling, strong cohesion

Most of the above means the same thing (its repeated several times, maybe take the hint)

it all adds up to make code more **maintainable**.  


## Other observations

If you look at Java, for example Spring (including Spring Boot), they have moved away from XML files which they used to setup their frameworks in favor for Annotations/Attributes. This brings some advantages. 

In .NET you will find frameworks now support *"Conventions over Configuration"* and allow to override using *"Fluent Configuration"* which try to support compile time validation.

I would like to see Java frameworks make more use of Conventions and Fluent Configuration as an alternative to Annotations and XML config.