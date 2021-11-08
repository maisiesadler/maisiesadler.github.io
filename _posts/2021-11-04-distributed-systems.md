---
layout: post
title:  "Resilience in Distributed Systems"
tags: distributed-systems resilience design architecture
---

Modern systems are increasingly complex and distributed.
Every engineer must understand the trade-offs of every decision and keep observability and resilience in mind.

This article outlines some patterns and practices for adding resilience to complex, distributed systems.

First a few definitions,

- **Resilience** - The ability to maintain acceptable service levels during a system failure
- **Complex Systems** - A system with enough parts that an individual cannot hold the model in their head, the outputs can be unpredictable
- **Distributed Systems** - A system is with multiple components spread over multiple hosts

Understanding the requirements of the system is important as it will help to guide our decisions for which behaviours are essential and which can be compromised.

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must decide for each operation when we encounter a failure; wait, retry or cancel?

## Wait

How likely is it that the result is just about to be returned?

If a network connection is open for an extended period not only does it have more time to fail but it is consuming resources and could block potentially successful calls.

Use observability to guide what is an acceptable time to wait and use timeouts to cancel after it is unlikely to return successfully.

Another option for waiting for available resources is to use events and queues, messages can be consumed at a steady rate.
This rate can be increased by scaling the component, ensuring the rate isn't so high we start overwhelming downstream resources such as databases.

## Retry

In a distributed system it is possible that a process is hanging due to a transient failure and that retrying the call could be successful. This can be done in memory or by re-queuing a message.

This can introduce some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - Out of order messages can apply updates incorrectly, this can potentially be dealt with using timestamps or versions
- **Retry Storm** - Too many retries can end up doing more harm than good

## Cancel

Is it acceptable to degrade certain functionality if it means other, potentially more critical, operations can continue?

Isolated functionality can be switched off using bulkheads and circuit breakers, this protects downstream resources and also allows the application to continue processing other potentially successful operations.
Traffic can be intermittently let through to test if service can be resumed.

## Other provisions

There are some patterns we can use in an attempt to make it less likely to encounter a failure however, we must be aware of their trade-offs.

### Scaling

We can add more resources to be able to handle more requests.

However, we must be cautious of overwhelming downstream resources.

### Cache

If we know what queries we expect to receive, we can serve the results from a cache.

The trade-off here is that the user will not always see the latest data.

### Events

Reduce coupling between systems by using events for notifications.

This is not a good pattern if the caller needs to know that the message has been processed before returning.

## Testing system resilience

It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

It is important to have good observability in place to be able to define the expected behaviour of our system under normal conditions, for example, "We expect 99% availability while processing 200 orders per second".

### Chaos Engineering

[Chaos Engineering](https://principlesofchaos.org/) is the practice of running experiments on a system to observe how it reacts.

This can reduce the on-call burden not only by giving higher confidence in the system but can serve as on-call training. Engineers become familiar with the observability tools and are engaged and focused on resilience.

- Start with one-off experiments or game days
- Choose experiments based on real-world events and incidents
- Once confident, run the experiment in production
- Have a rollback plan in place and revert once you have learned something

## Conclusion

Distributed systems are part of life working on modern software and it's important we understand the compromises we make with each decision - whether it's added complexity and maintenance, degraded experience, or just more expensive there will always be a cost to added resilience.

- 🛡 Protect resources where possible
- 💡 Be aware of the trade-offs introduced by a pattern
- 🕵️‍♀️ Ensure the system is observable
- 🧪 Test knowns and experiment for unknowns
- 🤓 Learn, improve, repeat
