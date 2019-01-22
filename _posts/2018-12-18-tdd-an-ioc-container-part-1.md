---
published: true
layout: post
title: TDD an IoC container - part 1 - primer
description: using TDD to develop a simple IoC container.
date: 2019-01-22
tags: [dotnet, tdd]
comments: false
---

Lets use TDD to build an IoC container, because we can.

This post series will look into how I apply TDD in order to have **confidence** in what has developed.

This post starts the series with creating automated tests that will describe the API and just enough code to ensure it complies.

## TL;DR;
- Its ok to do upfront design (just consider how much).
- We should have a nice idea on the initial API look and feel.
- TDD is being applied at the API level (this is where our requirements are understood by all)
- Just enough code to allow us to compile and run the tests (a lot of not implemented exceptions) 

# goal

The goal is write an IoC container in .NET Core, and only to write code that is required, each test should represent a requirement and that should force us to write code to pass (max out our code coverage).

This will not be a perfect implementation (look at autofac or windsor for that), but it will show that you can write tests first (with a bit of design, as we have an idea of what we want).

A number of tests will be written up front, (we can retro on this later) with the perceived value that we should write code knowing a few of the use cases not just 1 at a time (hopefully this will speed this up).

No unit (class level tests), hmm this is a controversial one for some people. The concept is that we only change the test when the requirement changes, this means when we refactor our code we can be sure that our test is ensuring that nothing is broken.

that seems enough to start with, lets actually start

# Defining our interface

Defining a large complex API is very hard to separate into clear and concise tests. I **will cheat a little** and get and idea of the overall interface.

As i am a large fan of both Castle Windsor and AutoFac, I will draw inspiration from both of them (and recommend you use them)


Before we fall into any implementation or design, lets look over (some of) the features we want (or at least consider):

- Registration of contracts
- Scopes (Transient, Singleton, and others)
- Determinalistic disposal
- constructor, property and method injection
- Resolve objets as lazy or resolve all objects of a type


The first design decision is how we create the container, for this I prefer how AutoFac **seperates the setup from the use**. this will lead to a interface which looks like this:

```
var builder = new ContainerBuilder();
builder.SetupModules(new RegisterContracts());
var container = builder.Create();
```

This tells a little more, that we want to support remigration through **Modules** (again similar to AutoFac and Windsor)

Ok lets look how we want the setup to work inside a module (again at the high level, we have no implementation), a small fluent interface is always a nice touch.

```
public void Setup(ContainerBuilder builder)
{
    builder.Register<LoggerPlain>().As<ILogger>().Scoped<Transient>();
}
```

When it comes to the use of the container, the following is the desired use, note that this container will support nested scopes.

```
using(var scope = container.CreateScope())
{
    var logger = scope.Resolve<ILogger>();
}
```

This is a simple overview of the intended interface use, it provides us with the major look and feel of the component, we have some upfront design, no tests and no implementation at this point.


# Defining the structure of a test.

Now we have an idea on the API, we can define a consistent structure for our tests (consistency should lead to simpler to maintain code). 

the goal of each test

- to assert against the public api.
- to test 1 thing.
- to be completely isolated.
- to have a human readable output
- to provide details on failure

Using MSpec we can take advantage of the 1 class per test, by **encapsulating** the registration module inside the test class.

```
[Subject("Container")]
public class When_resolving_a_simple_service
{
    Establish context = () => 
    {
        var builder = new ContainerBuilder();
        builder.SetupModules(new RegisterContracts());
        var container = builder.Create();
        
        _subject = container.CreateScope();
    };

    Because of = () => _service = _subject.Resolve<ILogger>();

    It should_provide_an_instance = () => PAssert.IsTrue(() => _service != null);
    It should_provide_an_instance_which_is_associated_with_the_registration = 
        () => PAssert.IsTrue(() => _service is LoggerPlain);
    
    static IScope _subject;
    static ILogger _service;
    
    class RegisterContracts : IModule
    {
        public void Setup(ContainerBuilder builder)
        {
            builder.Register<LoggerPlain>().As<ILogger>().Scoped<Transient>();
        }
    }
}
```

This looks a little on the large size, however it allows each test to have their own setup.

As we create this code, we need to create skeleton implementation classes (in order to make this to compile)

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/01%20-%20gettings%20specs%20to%20compile%2C%20minimal%20interface%20impl.PNG)

However if we look at the test above it covers our goals:

- to assert against the public api. - we test against what we resolve
- to test 1 thing. - MSpec does this with its **Because of** delegate  
- to be completely isolated. - Registration is inside the test class
- to have a human readable output - MSpec provides human readable output
- to provide details on failure - PAssert shows exactly what is going on.

# Putting this together.

Now we have a design, high level feature set and how we can test. We can now put this into action in a TDD style.

We start with some tests, and ensure we leave the solution in a state which will compile (we will now be at RED, ready to develop)