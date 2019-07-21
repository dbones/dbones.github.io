---
published: true
layout: post
title: tech radar
description: tech radars show how we can manage our technical platforms by mapping items in use against items of interest 
date: 2019-07-20
tags: [architecure]
comments: false
---

We can use a tech radar to show which technologies are of interest for use in our platform.

This post we look at a tech-radar for a .NET micro-services platform, of which is looking to migrate some of its technology.

The concept should be reuseable to other peoples/companies platforms. 

## TLDR;

- Use the radar to manage whats in use, or interest and what should be avoided.
- Can visualize and communicate intent to others
- A radar is individual to a platform, as each platform will have its own constraints and context. 
- start with a simple radar and grow it over time.
- a radar should be continually reviewed
- a radar is a snapshot in time. 

## Background

I first came across tech-radars from Thoughtworks, which showed technologies that the company observed, and provided them a mechanism to communicate their lessons learned from consulting on a number of projects.

A tech radar is used by multiple companies to allow them to manage their technology estate and coordinate its change.

# A dotnet platform

The platform we are reviewing is built using

- .NET services (running mono)
- Microservices
- Linux Containers

however given new changes in the landscape the platform owners want to address 4 key area's

- .NET Mono, upgrading to .NET Core 3.x
- Rancher 1.x being made obsolete
- Upgrading the backend database technology
- possible change of API Gateway technology

this platform has some key constraints mainly it has very limited resources (team size and money). This drives the platform to adopt open-source and SaaS components which support small projects.

# technology radar

Thougthworks provides a website which allows you to create your own radars, found here: https://radar.thoughtworks.com

We can use this to create a radar which shows the key technology and convey some of these concerns.

## Overview

The radar is split into 4 main quadrants, here are the default ones:

- Tools
- Language and frameworks
- Platforms
- Techniques

![overview](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/tech-radar/Overview.PNG "overview")


## Techniques

Technology is implemented using techniques, these are key and also impact other quadrants.

![techniques](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/tech-radar/techniques.PNG)

**observations:**

- This platform has a number of techniques which correlate with each other. its all about building small and composing big.

## Platform

Technologies which we run *our* platform on.

![platform](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/tech-radar/platform.PNG)

**observations:**

- Postgres is being considered as a new database technology
- Rancher 2.x is being looked into as a direct placement for 1.x, however Rio is also being understood.
- .NET core 3.x is in progress but not ready to obsolete Mono
- kong and ocelot are being looked at but not acted on yet.

## Languages and frameworks

The key components which we develop the platform with.

![languages-frameworks](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/tech-radar/language.PNG)

**observations:**

- Marten will be used to support the adoption of Postgres
- changing to .NET core runtime, has prompted trailing out Bonsai IoC and Polly to obsolete Castle Windsor.
- NancyFx has, unfortunately, had a slowdown in development, and ASPNET core WebApi is being adopted to replace it.

## Tools

Technology used in creating/maintaining the platform.

![tools](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/tech-radar/tools.png)

**observations:**

- Rancher 1.x along with Logentries provided a MVP to support DevOps. Prometheus is going to be trialed with Rancher 2.x to provide the required information.
- Zipkin is being trialed to improve on the platform monitoring
- development pipelines are looking to move over to CodeFresh

# Obsoleting technology

This radar shows a large story of a number of technologies being obsoleted and replaced, the radar itself is a snapshot in time. The following is a possible flow of how technology may happen over time:

1. New technology needs to be **assessed** to identity which one can be utilise.
2. A candidate technology is **trialed** in-order to find how it will be used in the platform and how we transition to it
3. the new technology is **adopted** into the platform
4. if the new technology replaces some existing technology, we can then put this on **hold**

In this case-study, the next radar will show a number of technologies being put on **hold** and other being **adopted**.

# Conclusion

The tech radar supports technical architects to manage the platform's technical landscape. 

It can be (and most likely should be) used to support the platform Roadmap.