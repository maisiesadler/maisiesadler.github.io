---
layout: post
title:  "Resilience in Distributed Systems"
tags: distributed-systems resilience design architecture
---

Modern systems are increasingly complex and distributed.
Understanding the requirements of the system is important as it will help to guide our decisions for which behaviours are essential and which can be compromised.

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must decide for each operation when we encounter a failure; wait, retry or cancel?

## Wait

How likely is it that a successful result is just about to be returned?
Observability tools can guide us to know what is an acceptable time to wait for a successful result.

Events and queues can be used to wait for available resources and messages can be consumed at a steady rate.
This is a particularly good choice if the operation is an action to be processed rather than a query for data.

## Retry

In a distributed system it is possible that a process is hanging due to a transient failure and that retrying the call could be successful.

Messages can be duplicated or out of order meaning we might need to add an ordered identifier to help us process correctly.
If a system is failing and cancels all in-progress calls and all are retried at the same time this will cause extra load on an already struggling system.

## Cancel

Sometimes it is better to degrade functionality to allow other, potentially more critical, operations can continue.

Functionality can be automatically switched off under certain error conditions using circuit breakers, this allows downstream resources to recover and the application to continue processing other operations.
Traffic can be intermittently let through to test if service can be resumed.

## Prevention

There are some patterns we can use in an attempt to make it less likely to encounter a failure.

### Scaling

We can add more resources to be able to handle more requests, this can be increased and decreased as the load changes and, idle instances can be added for redundancy.

We must be cautious of overwhelming downstream resources.

### Cache

If we know what queries we expect to receive, we can serve the results from a cache.
This can be populated after a query or when the data is updated.

However, the user will not always see the latest data.

## Testing system resilience

It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

[Chaos Engineering](https://principlesofchaos.org/) is the practice of running experiments on a system to observe how it reacts. This gives higher confidence in the system and can serve as on-call training. Engineers become familiar with the observability tools and are engaged and focused on resilience.

## Conclusion

Distributed systems are part of life working on modern software. We must understand the compromises we make with each decision - whether it's added complexity and maintenance, degraded experience, or just more expensive - there will always be a cost to added resilience.

- 🛡 Protect resources where possible
- 💡 Be aware of the trade-offs introduced by a pattern
- 🕵️‍♀️ Ensure the system is observable
- 🧪 Test knowns and experiment for unknowns
- 🤓 Learn, improve, repeat
