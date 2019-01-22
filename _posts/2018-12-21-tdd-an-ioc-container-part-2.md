---
published: true
layout: post
title: TDD an IoC container - part 2 - initial slice
description: using TDD to develop a simple IoC container.
date: 2019-01-22
tags: [dotnet, tdd]
comments: false
---

This post series looks into how I apply TDD in order to have **confidence** in what has developed.

In this post we wil look at how to tackle the complex api in order to get a small working vertical.

## TL;DR;

- take the easiest test (with most value) and make it work (it may cover more code than you think), then we can refactor.
- think about the high level design to make refactoring slighly easier.
- spikes and looking at exiting code (my previous implementations) help articulate thoughts

## recap

In the previous post we created:

- a number of tests which show the intent of this software from the consumer side (essentially the requirements for it)
- skeleton classes which allow the project to build and the tests to run (and go RED)

this allows us now to start implementing the solution, and getting some green lights.

# The smallest, yet most value, vertical

Looking at our tests the simplest one to make a light go green (without smoke ane mirrors), is the test from the last post (where we register a single component and resolve it)

however the one at this stage (which offers the most value) is where a service has 1 dependency.

In order to make this happen we need to design and implement:

- some of the registration
- creation of the container / scope
- resolving of the logger (constuctor injection)

wow that is a lot.

In order to keep this as small as possible, we only need to be aware of property/method injection, provided ctor or instances, generics, lists, lazy, disposables, etc.

So we are delivering a smaller part, by far.

# Design

the main parts of the work is creating the container and resolving the instance.

ok we have a few areas which we need to consider (some of these are distinctly identifable from the tests).

- Planning - we need to identify which ctor, properties and methods to use.
- Factory method - a deleage which is called with the current scope will create an entire object tree
- Lifetime Styles - a way to control when objects are created and if they are disposed of.
- Scopes - essentially the mechanism to determin when to dispose of objects and resolve/create them. note the container is a scope as well.


# Thoughts on the implementation

## Tests

We could have argued that just resolving the single object would have been a better starting place, thats fine, i just wanted to implement a bit more and was not much effort (and allowed me to get more of the internal structure ironed out).

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/4%20passing%20tests.PNG)

By completing the implementation for the Service with 1 dependency, I actually passed 4 tests! 

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/4%20tests%2071%25%20covereed.PNG)

with these 4 tests we have 71% coverage. hmm what does that mean? well its not 100 as we have code which was implemented to make the project compile and this is not being covered, and i imeplemented some code to support other tests.

the main point though is the **correctness** (we have 4 passes now)

## Spiking planning for Generics

While creating the container, i **spiked** how to support generics and named services. (there is code to be cleaned up around this).

However this changed can impact how we initially setup the container, as the ioc container needed to know of all the used Generic arguments the application is going to use.

To do this, in planning it scans only the registrations with a concrete implementation, not an open generic, and then it figures out actual implementations from the injected dependencies

Example

```

//at this moment we do not know of all the actual implementations
public class Repository<T> : IRepository<T> { .... }

public class Service  
{
    //now from this constrcutor, we know that we need a Repository<User>
    public Service(IRepository<User> userRepo) {.....}
}
```

so by scanning the Service class's ctor, we now know of the generic args.

This was based off an small observation that most apps use frameworks, which the entry point for the IoC will be a concrete class, exmaples:

- Controllers (web api)
- Consumers (Masstransit, NServiceBus)
- Modules (Nancy)
- Hosted Services (.Net Core)
- RequestHandlers (MediatR)

thus we should know of any / all generic parameters by walking through the dependencies

## Implementation

The implementation is not the cleanest. some of the decisions are to get a starter for 10, get the green and refactor a little later.

For exmaple, we are using reflection to create the classes, it easier to code and understand, but we will need to replace this with Expresssions or IL code.

```
object ParameterLessCtor(IAdvancedScope scope) => method.Invoke(createParams.Select(x => x(scope)).ToArray());
```

Lifestyles do not contain instance caches, however provide a stratgy to when and where an object is created, cached and tracked for disposal.

```
    public class PerScope : ILifeSpan
    {
        public object Resolve(IAdvancedScope currentScope, Contract contract)
        {
            var entry = currentScope.InstanceCache.Get(contract.Id);

            if (entry == null)
            {
                entry = new Instance()
                {
                    Value = contract.CreateInstance(currentScope),
                    Contract = contract
                };
                currentScope.InstanceCache.Add(contract.Id, entry);
            }

            currentScope.Tracked.Push(entry);

            return entry.Value;
        }
    }
```


this lead to the scopes having to deal with their cache being modified from multiple threads. To enable this we used the Microsofts ```ReaderWriterLockSlim```, which should provide a performance access to the instances which are cached at that scope level. 

# pulling this together

we have an inital start of the project, there is a lot of work to do inorder to statify the requirements

we have some code which hightlights that we need to add some new tests (to represent a new requirement)

personally i like how the scope works, but im eager to see its performance, and get more support in for generics and named instances.

However this does seem like an interesting start.