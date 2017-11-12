---
published: true
layout: post
title: Microservices - part 3 - Service Location and Load Balancing
description: Microservices - Part 3 - routing messages to the correct service instance.
date: 2016-01-10
tags: [microservice, architecture]
comments: false
---

When we split our application into multiple services we need a way to locate an instance of a service.

## TL;DR;

A component, or components, which handle the 

- location of service instance, 
- load balance requests to a service instance. 

this can be done by a few different ways, including:

- Dns lookup
- Service locater + load balancer
- Queues (such as rabbit)

note: the first 2 require something to maintain the registry.

# Service location

With the of hosting multiple instances of a service, located 1 or more machines. We need to be able to route massages to the correct endpoint.

for example, here is the Api-Gateway requiring to route messages to services.

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/client-depends-on-services-directly.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/apiGateway-requires-service-location.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/apiGateway-requires-service-location.JPG" title="Microserive">How does the Api Gateway know where the services are located</a>.</figcaption>
</figure>
 

A possible (limited) solution would be to assign Host entires to each service, however this will not solve the issue of multiple instances and load balancing (check out the DNS solution).

Here are some solutions

## DNS

Using the DNS is a simple solution, where all the instances of a service register with the DNS. 

Now when when we require the location of a service, we do a DNS lookup and it returns the location of all (active) instances of the service (multiple A records).

It is up to the us (the calling client) [to randomly pick an entry](https://github.com/rancher/rancher/issues/1401).

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-dns.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-dns.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-dns.JPG" title="Dns lookup">Dns service lookup</a>.</figcaption>
</figure>

## Load-balancer + Registry

Another way is to combine a couple of components such as a Load-Balancer + Registry. 

In this case the service instances will register with a Registry (ZooKeeper, Ectd, or a DNS for example). This is where the list of active services will be maintained.

Now each request will be routed through a load-balancer (HA proxy, Nginx). In this case we can make use of the loadbalancer stratgies (round robin or more advance ones), effectively removing the responsibility of routing from the client.

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-registry.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-registry.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-registry.JPG" title="loadbalancer + registry lookup">loadbalancer + registry service lookup</a>.</figcaption>
</figure>

 
## Queue Broker

With queues such as rabbit, you will naturally get service location (as it naturally routes messages), also you can achieve load balancing (with rabbit this is done via the pre-fetch). 

By setting the number of messages each consumer can actively handle, the broker will now load balance messages (instead of sending all of them to single consumer instance).

<figure>
	<a href="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-Amqp.JPG"><img src="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-Amqp.JPG" /></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/microservices/serviceDiscovery-Amqp.JPG" title="amqp lookup and balancer">Amqp as the loadbalancer + registry service lookup</a>.</figcaption>
</figure>

# Side Note

With the first 2 options you actively need to keep the registry up-to-date, with the last solution this is automatically taken care of.

## Off the shelf

Given the side note it may be worth investing in a mange solution such as:

- [Rancher](http://rancher.com/)
- [Kuberneties](http://kubernetes.io/)
- [Docker - swarm mode](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/) - v1.12

these offer a solution towards the first 2 scenarios mentioned here, as well as keeping the registry up-to-date.

and for the 3rd scenario, consider using 

- [RabbitMq](https://www.rabbitmq.com/)