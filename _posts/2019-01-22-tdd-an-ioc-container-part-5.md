---
published: true
layout: post
title: TDD an IoC container - part 5 - Retrospective
description: using TDD to develop a simple IoC container.
date: 2019-01-25
tags: [dotnet, tdd]
comments: false
---

This post series looks into how I apply TDD in order to have **confidence** in what is developed.

## catch-up

At this point we have

- developed a small IoC container, its not complete, but it does work 
- A number of automated tests which detail the features and can double up as documentation
- Automated CI pipeline
- project available to all to view

# Retro

### Initial Goal

- Prove how you may write tests first as a way to manage requirements and gain confidence in what you have developed

### Scope

- Write some tests
- Write some code
- Refactor


Did we succeed ?


**what went well:**

- A number of tests covering a major set of functionality was written up front.
- Tests drove a TDD cycle (Red -> Green -> Refactor), where we did refactor code several times.
- Tests were aimed at the API, not at each individual class, this allowed for major refactoring of the code, without the need to touch the tests.
- dotnet tooling was excellent to support this journey.
- We did hit 80% coverage without really trying, i know some have struggled with this.
- Really enjoyed the MSpec structure, i would like to see what others think of this too.

**what could be improved:**

- Code was still added without tests, thus we did not hit 100%, we can easily fix this to get closer to full coverage.
- More focus on quality as we progress the library.
- We do not really have any docs as of yet.

I'm sure there are more places to improve.

# Conclusion

We have a **start** of a fun **product**, it needs more refactoring to be fully battle hardened, however it does offerer some advanced features already.

We have shown how tests can be written first

So did we succeed... given the constraints, I would like to say.... **yes** ;)
