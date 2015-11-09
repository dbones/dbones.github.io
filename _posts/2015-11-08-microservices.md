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

##TL;DR;

general guidance is that each Microservice follows

- Single Responcibility Pincipal.
- Look for domain boundaries to identify a Microservice.
- Any resource is encapsulated, ie no other service can access my database.
- Consider an EDA architecture, with eventual consistancy.
- Make services stateless.

# Microserice

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-boundary.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary.JPG" title="Microserive">Microservice</a>.</figcaption>
</figure>


Considering the definition for a Service Oriented Architecure **(SOA)**, a Microserice is another iteration on this idea where each service is meant to have a single purpose. The goal is to cut up an application into smaller bounded contexts, and host each context in their own service. In the past people would implement a single service teir for the entire application, this would now be seperated into several smaller services (microservices).

Each Microservice can have

- its own API (which could be HTTP, AMQP etc).
- its own resources (databases, files etc), which cannot be accessed by anyone else.


##API

there are a number of ways to expose an API with services. The 2 main protocols I have seen beening used are HTTP and AMQP. More importantly we need to consider the style of of interaction we have, as both protocols can be used:

- Request Response **(R/R)**, mainly sync - a client will send a request and the service will provide a response.
- Publish Subscribe **(P/S)**, async, normally indirect - a client will publish information, called an Event, and the service will subscribe and act accordingly.

As mentioned you can achive both sytles with both HTTP and AMQP.

###HTTP

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-http.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-http.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-http.JPG" title="Microserive">Access services via HTTP</a>.</figcaption>
</figure>

With the HTTP, you can support both R/R and P/S. the key with this interface is to embrace the HTTP Verbs so choose a framework which supports these easliy (Express/NancyFx/Sinatra).

- R/R is pretty much out of the box, note to achive async feedback you can return the HTTP status 202.
- P/S, this could be done via WebHooks. where the subscribers will regierster an endpoint with the service, and the service will call these endpoints when an events occour.

The challange with this style has been service discovery and loadbalancing (I will have a post on that later), the good news this is becoming easier by the day, with tools such as Rancher, Kuberneties, etc.

###AMQP

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-amqp.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-amqp.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-amqp.JPG" title="Microserive">Access services Via AMQP</a>.</figcaption>
</figure>

Using an AMQP broker we can achive both R/R and P/S, with servers/components such as RabbitMq make support both styles easier.

- R/R, this is achived by the client providing a return queue (each client instance will have their own return queue) and sending a requst/message with a corralation ID and location of the return queue. The service will send the response back to the return queue with the correct corralation ID. This seems a little complex but there are frameworks which do this all for you, such as EasyNetQ. 
- P/S, this is out of the box for all the frameworks which support AMQP.

This solution provides a couple of interesting side effects.

- indirection
- service location
- load balancing (when you set the prefetch, a round robin is then initiated)
- a bit of a challange with regards to R/R. consider if more than one service responds.

##Resources

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary-donot access-others.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/service-boundary-donot access-others.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/service-boundary-donot access-others.JPG" title="Encapsulte resources">Microservice</a>.</figcaption>
</figure>

As part of following good engineering parctices we aim to **Encapsulate** our internals from others. This is true for microservices as well. This means that other services cannot access the database or internal resouces of the current service. Services will only access one another via their API's. 

**Note** that a service can cache or store their own version of the truth about the data.


##Scaling

The best way to scale a service is to make it stateless, allowing to add addtional instance of a service where any of the instances can handle a request.

Also ensure you follow any clustering reconmendations for any stateful components/dependencies, ie databases, queues, will support clustering for High avalibilty and scale.