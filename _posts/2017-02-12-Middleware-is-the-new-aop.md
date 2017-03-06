---
published: true
layout: post
title: Middleware - is the new AOP?
description: a popular way to provide extensions in todays frameworks.
date: 2017-02-12
tags: [architecture, dotnet]
comments: false
---

Many libraries and frameworks are allowing for extensions via a middleware pattern. For example 2 MVC frameworks: .NET Core MVC/WebApi and Node.js Express.

This article has a quick look into this middleware pattern, as found in the mentioned frameworks, and how it can be implemented.

**Note** impl can vary, depending on your framework.

**Demo code** I have written a quick impl of a middleware component located here: https://github.com/dbones/middleware-example

## TL;DR;

Middleware is a pattern to support separation of concerns, allowing each action to extend functionality without modifying the target code.

It fills a similar goal to Chain-of-responsibility or AOP, where the main difference is AOP proxies (assuming a proxy approach) the entire target class.

An implementation of this (middleware) pattern can be easy, in comparison to AOP, as it can be achieved without post compile weaving or dynamic proxy IL code (assuming the use of .NET).

Note: AOP is meant to be invisible to the rest of app, where-as middleware style, the app tends to be designed with it in mind.

# Quick overview

If we take for example a Transfer Command, it Transfers money between account A and B.

The TransferCommand is only interested in Transfer money logic, however in this over simplified example we want to *Log before and after* the method, and *time how long the method took*.

the last 2 requirements are cross cutting concerns, and not to do with transferring money.

In this example we can use middleware actions to accomplish this logic.

<figure>
	<a href="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/overview.PNG"><img src="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/overview.PNG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2016/middleware-the-new-aop/overview.PNG" title="overview">overview</a>.</figcaption>
</figure>

The thing to note, the execution of code is past into an action, where the action can run its own logic.

each action has access to the 2 things 

- context, the state which the action is being executed in.
- next, a delegate which will invoke the next action in the chain, and and if invoked. 

# Code

ok we mentioned that we have a TransferCommand, it does not really do much so insert imagination here:

```
public class TransferCommand
{
    public virtual Task Execute(Transfer transfer)
    {
        //do some logic here.
        Console.WriteLine($"Transfering {transfer.Amount}, "+
                          $"from: {transfer.SourceAccount} "+
                          $"to {transfer.DestinationAccount}");

        return Task.CompletedTask;
    }
}
``` 

**Small Note:**

- Use of virtual, other than its a good practice, I added this to demo code to support Castle.Core AOP, as the middleware did not require this.


## Action

Now we want to log before and after. 

The following 2 code snippets are the middleware Action and AOP interceptor, just to show how similar they are.

**Action**

```
public class LogAction : IAction<Transfer>
{
    public async Task Execute(Transfer context, Next<Transfer> next)
    {
        Console.WriteLine($"before {context}");
        await next(context);
        Console.WriteLine($"after {context}");
    }
}
```

**interceptor**

```
public class LogInterceptor : IInterceptor
{
    public void Intercept(IInvocation invocation)
    {
        Console.WriteLine($"before {invocation.Arguments[0]}");
        invocation.Proceed();
        invocation.After(() =>
            Console.WriteLine($"after {invocation.Arguments[0]}"));
    }
}
```

Both ways implement small parts of logic (cross cutting concerns), both have the power to stop calling to the next function in the chain.

## Setup

At the moment we have the separate elements such as the TransferCommand and Actions.

To make this complete, we need to configure the app to use it. As this is demo code, the following is minimal impl to show the concept.

<figure>
	<a href="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/setup.PNG"><img src="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/setup.PNG" /></a>
    <figcaption><a href="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/setup.PNG" title="setup">the setup of the middleware (left) and AOP (right)</a>.</figcaption>
</figure>


this is quite interesting

- both can use the IoC container to support scope of the actions (singleton, transient etc), allowing state. 
- AOP is applied in a transparent way to the TransferCommand, in this case via a dynamic proxy, and the actual app is unaware.
- AOP is fully supported within the IoC container (most containers support this)
- AOP, this impl, is applied to all methods on the target class.
- Middleware, well this implementation, is great for a singleton object.
- Middleware requires an Action to invoke the the target (CommandAction in this case).
- Middleware does not inherit off the TransferCommand, and does not impose any impl rules, i.e. virtual on methods are not required.

# Running

As the app has been setup, we can run and see that the actions intercept the execution.

<figure>
	<a href="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/running.PNG"><img src="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/running.PNG" /></a>
    <figcaption><a href="http://dbones.github.io/images/posts/2017/middleware-the-new-aop/running.PNG" title="running the code.">running the code</a>.</figcaption>
</figure>