---
published: true
layout: post
title: Microservices - part 1 - the boundary
description: Microservices - Part 1 - the boundary
date: 2015-11-08
tags: [microservice, architecture]
comments: false
---

This post series are a few notes on Microservices, based on observations and experience.

## TL;DR;

general guidance is that each Microservice follows

- Single Responsibility Principal.
- Look for domain boundaries to identify a Microservice.
- Any resource is encapsulated, ie no other service can access my database.
- API can use more than one protocol and there are 2 styles of interactions pub/sub & req/res.
- Consider an EDA architecture, with eventual consistency.
- Make services stateless.

# Microservice

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-boundary.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary.JPG" title="Microserive">Microservice</a>.</figcaption>
</figure>


Considering the definition for a Service Oriented Architecture **(SOA)**, a Microserice is another iteration on this idea where each service is meant to have a single purpose. The goal is to cut up an application into smaller bounded contexts, and host each context in their own service. In the past people would implement a single service tier for the entire application, this would now be separated into several smaller services (microservices).

Each Microservice can have

- its own API (including the protocol ie HTTP and the style Request/Response or Publish/Subscribe).
- its own resources (databases, files etc), which cannot be accessed by anyone else.


## API

there are a number of ways to expose an API with services. The 2 main protocols I have seen being used are HTTP and AMQP. More importantly we need to consider the style of of interaction we have, as both protocols can be used:

- Request Response **(R/R)**, mainly sync - a client will send a request and the service will provide a response.
- Publish Subscribe **(P/S)**, async, normally indirect - a publisher will publish information, called an Event, and a consumer will subscribe and act accordingly.

As mentioned you can achieve both styles with both HTTP and AMQP.

### HTTP

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-http.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-http.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-http.JPG" title="Microserive">Access services via HTTP</a>.</figcaption>
</figure>

With the HTTP, you can support both R/R and P/S. the key with this interface is to embrace the HTTP Verbs so choose a framework which supports these easliy (Express/NancyFx/Sinatra).

- R/R is pretty much out of the box, note to achieve async feedback you can return the HTTP status 202.
- P/S, this could be done via Web-Hooks. where the subscribers will register an endpoint with the service, and the service will call these endpoints when an events occur.

The challenge with this style has been service discovery and load-balancing (I will have a post on that later), the good news this is becoming easier by the day, with tools such as Rancher, Kuberneties, etc.

### AMQP

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-amqp.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-amqp.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-amqp.JPG" title="Microserive">Access services Via AMQP</a>.</figcaption>
</figure>

Using an AMQP broker we can achieve both R/R and P/S, with servers/components such as RabbitMq make support both styles easier.

- R/R, this is achieved by the client providing a return queue (each client instance will have their own return queue) and sending a requst/message with a corralation ID and location of the return queue. The service will send the response back to the return queue with the correct corralation ID. This seems a little complex but there are frameworks which do this all for you, such as EasyNetQ. 
- P/S, this is out of the box for all the frameworks which support AMQP.

This solution provides a couple of interesting side effects.

- indirection
- service location
- load balancing (when you set the prefetch, a round robin is then initiated)
- a bit of a challenge with regards to R/R. consider if more than one service responds.

## Resources (internal dependencies)

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary-donot access-others.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-boundary-donot access-others.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary-donot access-others.JPG" title="Encapsulte resources">Microservice</a>.</figcaption>
</figure>

As part of following good engineering practices we aim to **Encapsulate** our internals from others. This is true for microservices as well. This means that other services cannot access the database or internal resouces of the current service. Services will only access one another via their API's. 

**Note** that a service can cache or store their own version of the truth about the data.

## Eventual Consistency and Events

As with SOA, eventual consistency suites this architecture too. To achieve this consider using an Event Driven Architecture with Queues (such as RabbitMq) to handle guaranteed delivery for you.

for more info: http://udidahan.com/2011/09/18/inconsistent-data-poor-performance-or-soa-pick-one/

## Scaling

The best way to scale a service is to make it stateless, allowing to add additional instances of the service, where any of the instances can handle a request.

Also ensure you follow any clustering reconmendations for any stateful components/dependencies, ie databases, queues, will support clustering for High availability and scale.