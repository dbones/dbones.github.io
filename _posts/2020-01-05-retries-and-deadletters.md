---
published: true
layout: post
title: RabbitMQ - retries and deadletters
description: Rabbitmq using deadletter queues
date: 2020-01-05
tags: [dotnet, messageing, microsevices, rabbitmq]
comments: false
---

RabbitMQ offers a lot of excellent features, but one which is not supported out of the box is message retries.

In this post we look at a way to support message retries, with processing back-off while supporting deadletter queues.

## TLDR;

- use Masstransit/NServiceBus if you can
- Services may fail processing messages due to 1) dependencies are down (databases) 2) the message may need to be fixed
- (ATM) there is no production ready plugin that solves this
- A number of solutions on the web will either involve an infinite loop or they will just erase the message
- Look at the overall solution drawing below (there is no real short cut)
- This is a complex problem and complex solution.
- This may not be for you (this is a generic solution)

## Background

When processing messages, over time our services will fail for some reason such as

1. The message payload is correct, however the service's database is down, or some other dependency resource (VM dies), and we need to provide a small delay to allow for the service to become operational again.
2. The payload is incorrect and it requires for some sort of manual intervention.
3. The service code is incorrect and an engineer needs to deploy a code fix.

These are some of the reason's which pop to mind will writing this post at 4am.

Rabbit has support [deadletter queues](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html) out of the box, which solves 2 and 3 listed above.

However it does not really have an answer for issue 1. (or at least not production ready). 

To address this the community have posted a number of solutions, which have some really excellent ways of retrying the message processing. however they loose the ability of the dead letter channel.

- [RabbitMQ - Retries the full story](https://engineering.nanit.com/rabbitmq-retries-the-full-story-ca4cc6c5b493) - really good read
- [RabbitMQ - Exponential backoff](https://felipeelias.github.io/rabbitmq/2016/02/22/rabbitmq-exponential-backoff.html) - ensure you read the comment left on the post
- [RabbitMQ - retries on nodejs](https://blog.forma-pro.com/rabbitmq-retries-on-node-js-1f82c6afc1a1)

This post will look at a possible solution, building on the above articles, we look into a possible solution.

(If you are in .NET you can use solutions like Mass-transit or NServiceBus which have solutions for this).


# Overview

This solution uses several Exchanges which can support any number of queues.

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/rabbit-retry-deadletter/overview.PNG)

wow, thats is a lot, lets break it down.

## Process

1. The service will process messages only from its topic-queues.
2. On failure it rejects (**nack**s) the message. 
3. Message it moved to the configured deadletter exchange. The key part is we need to set the 'x-dead-letter-routing-key' to the name of the topic-queue, that the message was in.
4. The failed message is queued in the failed queue 
5. A Failed process takes the messages, increments a custom header called **count** until it reaches the max reties (at that point it sets count to -1). It then re-publishes the same message to the deadletter exchange, and finally it **ack**s the old message off of the failed queue.
6. The deadletter exchanges routes the message to the correct **retry** queue using the **count** header.
7. Each retry queue keeps the message for the agreed queue TTL, which then invokes the queues deadletter action. The message is routed back to the correct topic queue using the routing key we set back in step 2.
8. When the message as been retried x times, it will be routed to the deadletter queue for intervention.

# Components

## Topic Queues

Each topic queue will be bound to the message topic that it is for as normal, additionally we

- bind it to the reply, using direct routing key
- add the **x-dead-letter-routing-key** to the same name as the key (it is this which ensure that the message will be routed back to the same topic-queue).

it should look something like this:

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/rabbit-retry-deadletter/topic-queuePNG.PNG)

## Processing Failed messages

Note that we publish the message to the deadletter exchange before we remove it, this is so we do not loose our message.

worst case we may publish the message more than once.

We need to ensure that we keep the message id intact, allowing for a message filter to deal with accidental duplicates.

```
var countValue = args.BasicProperties.Headers["count"];
var newCount = countValue + 1;

//see if we have exhausted the retry queues
if (newCount > _busConfiguration.RetryBackOff.Count)
{
    newCount = -1;
}
_logger.LogInformation($"retrying message: {args.RoutingKey}, id: {args.BasicProperties.MessageId}, count: {newCount}");

args.BasicProperties.Headers["count"] = newCount; 
_channel.BasicPublish(_busConfiguration.DeadLetterExchangeName, args.RoutingKey, args.BasicProperties, args.Body);

//remove it from the failed queue
_channel.BasicAck(args.DeliveryTag, true);
```

## Deadletter exchange routing

In order to support the deadletter channel pattern, we use a headers routing exchange

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/rabbit-retry-deadletter/dead-letter-queue.PNG)

this is where the count header is used to route the message to be retired or deadlettered.

```
_channel.QueueBind(
  retryQueue.QueueName,
  _busConfiguration.DeadLetterExchangeName, 
  "", 
  new Dictionary<string, object>()
  {
      { "x-match","all" },
      { "count", count }
  });
```

Note: there is not much documentation around routing via headers at the point of writing (which is a shame)

## retry queues

The retry queues are setup with TTL's. We should configure these to allow time for the struggling service time to heal itself.

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/rabbit-retry-deadletter/retry-queue.PNG)


## A rejected message

Finally, the error'ed message, as it was key that we did not loose our messages, we can find a full message in the deadletter queue.

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/rabbit-retry-deadletter/errored-message.PNG)

we can see that the message was 
- processed 4 times by the topic queue.
- that it timed out on each of the retry queues.

note that the x-death[0].count is not accessable to the headers routing exchange, thus we had to write our own count header.


# Conclusion

this is quite complex, but we have a solution which supports

- a max number of retries
- back off between each retry
- deadletter queue

the solution is some what generic, i.e. you can add more topic queues without changing any of the retry code.

it would be excellent if Pivital/VMWare could add a plugin, so we do not need this solution.