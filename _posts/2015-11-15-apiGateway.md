---
published: true
layout: post
title: Microservices - part 2 - Api Gateway
description: Microservices - Part 2 - Api Gateway, encapsulate your services.
date: 2015-11-15
tags: [microservice, architecture]
comments: false
---

As part of working with microservices a common practice is to employ an API Gateway, either off the shelf or custom. This article contains some notes on this component.

## TL;DR;

A service which is responsible to encapsulate all the other microservices so clients have 1 place to go. 

- encapsulates your microservices to represent a single product
- coordinate microservice (questionable)
- apply any proxy level code, cross cutting concerns, this is code which does not belong to a single service, ie Security, Caching.

# Api gateway

Before pushing an Api gateway into your architecture lets understand the problem. 

Given we have a client, it would need to contact to various microservices, which leads to complexities such as Service Discovery, Security, handling different protocols etc. From the service provider we could see issues as increased surface attack area, difficult to encapsulate logic. 

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/client-depends-on-services-directly.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/client-depends-on-services-directly.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/client-depends-on-services-directly.JPG" title="Microserive">Direct access to services</a>.</figcaption>
</figure>

Introducing the Api Gateways will simplify the connection between the client and the entire application, by being the single (please ensure it can be scaled) consistent endpoint, and provide a place to tackle some of the concerns mentioned before.

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/apiGateway-encapsulates-services.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/apiGateway-encapsulates-services.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/apiGateway-encapsulates-services.JPG" title="Microserive">Api Gateway encapsulates services</a>.</figcaption>
</figure>

This entry point into the application provides us with the following features

- consistence api for the client, your microservice may expose different protocols, which can be managed and adapted within the Api Gateway.
- encapsulate functionality, the microservices may have an API where only a subset of its functionality, these can be encapsulated behind the Api-gateway.
- encapsulate change, services can make changes without directly impacting the end client.
- security surface, only expose the Gateway to the external world, not all the microservices, which reduces the attack surface.
- cross cutting concerns can be applied here, ie security, caching, monitoring, throttling etc

**notes**

For the HTTP interfaces, consider this as the point where you expose your swagger docs.

All requests originate on the Api Gateway, so we ensure a Correlation Id is generated at this level, for distributed transactions.

Some may put Service Discovery into this component, however there are some great alternatives.

## Off the shelf
You have options to bringing this component into your solution, either bespoke or off the shelf. This will depend on your requirements and product constraints (time, budget, skills, tech, private cloud etc). Here is a list of Gateways which can be used.

- [NGINX Api Gateway](https://www.nginx.com/solutions/api-gateway/)
- [Kong](https://getkong.org/)
- [StrongLoop Api Gateway](https://strongloop.com/node-js/api-gateway/)
- [Oracle Api Gateway](http://www.oracle.com/us/products/middleware/identity-management/api-gateway/overview/index.html)

If you are using a [AWS](https://aws.amazon.com/api-gateway/), Azure etc, they most likely offer their own solution for this.

## More info

- [Netflix](http://techblog.netflix.com/2012/07/embracing-differences-inside-netflix.html)
- [Microservices.io](http://microservices.io/patterns/apigateway.html) - a great resource!
