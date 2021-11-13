---
layout: post
title:  "Resilience in Distributed Systems"
tags: distributed-systems resilience design architecture
---

Modern systems are increasingly complex and distributed.
This article outlines some patterns and practices for adding resilience to complex, distributed systems.

First a few definitions,

- **Resilience** - Resilience is the ability to maintain acceptable service levels during a system failure
- **Complex Systems** - A system composed of many parts interacting with each other
- **Distributed Systems** - A system is distributed if there are multiple components spread over multiple hosts

Understanding the requirements of the system is important as it will help to guide our decisions for which behaviours are essential and which can be compromised.

## Encountering Failure

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must decide for each operation when we encounter a failure; wait, retry or cancel?

### Wait

How likely is it that the result is just about to be returned?

If a network connection is open for an extended period not only does it have more time to fail but it is consuming resources and could block potentially successful calls.

Use observability tools to guide what is an acceptable time to wait and use timeouts to cancel after it is unlikely to return successfully.

### Retry

In a distributed system it is possible that a process is hanging due to a transient failure and that retrying the call could be successful. This can be done in memory or by re-queuing a message.

Some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - Out of order messages can apply updates incorrectly, this can potentially be dealt with using timestamps or versions
- **Retry Storm** - Frequent retries can make it hard for a system to recover

### Cancel

Is it acceptable to degrade certain functionality if it means other, potentially more critical, operations can continue?

Isolated functionality can be switched off using bulkheads and circuit breakers, this protects downstream resources and also allows the application to continue processing other potentially successful operations.
Traffic can be intermittently let through to test if service can be resumed.

## Preventing Failure

There are some patterns we can use to attempt to reduce the chance of encountering a failure. However, we must be aware of the trade-offs.

### Isolation

Critical operations can be split up into separate components and their resources can be isolated, this prevents them from being affected by failure in other parts of the system.

Isolation also allows teams to independently maintain different system capabilities.
A large system can be split into logical [bounded contexts](https://martinfowler.com/bliki/BoundedContext.html) with explicitly defined relationships.

Within these domain boundaries there is less complexity since there are fewer states and interactions that can occur.

### Scaling

Adding more resources to be able to handle more requests.
This can be _horizontal_ by adding more similar resources, or _veritical_ by increasing the size of existing resources.

This should be controlled so we do not overwhelm downstream resources.

### Caching

Is it ok to return slightly stale data if it can be accessed much more quickly?

Can we preempt what the user will request and precalculate the value ahead of time?

If so, [caching](https://aws.amazon.com/caching/) the data could be a good option to increase availability and reduce load on the rest of the system.

The system should eventually update and be correct and this type of system is known as eventually consistent.

### Events and Queues

Events can be used to durably capture a request for an operation.
This increases availability by allowing an operation to resume after a failing component has recovered.

The consuming component can process messages at a steady rate, this rate can be increased by scaling the component.

This pattern introduces some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - An earlier message could undo the work of a more recent message, this can be configured in a queue, or potentially dealt with using timestamps or versions

Event-driven architecture is loosely coupled, the event doesn't know about the consequences it can cause.

## Embracing Failure

With every new feature our software systems have more states and interactions making them harder to model.
Since complex systems are made up of individual components interacting with each other, as a system grows and scales it becomes more complex.

We add more logic and states to increase resilience add in turn add more complexity.

All is not lost, if we accept our system is and will continue to be complex then we can look to complexity theory to give us a set of tools to understand the patterns and behaviours we are seeing.

### Complexity Theory

These systems often exhibit **non-linear** behaviour, meaning that the same inputs do not always produce the same outputs and a small change in inputs can produce un-proportional changes to the outputs.

**Chaos theory** is the theory that complex systems, though unpredictable, are not random and that there are patterns that govern their behaviour - feedback loops not linear equations.

**Self-organisation** model tells us that global patterns form out of local interactions.

**Adaptive theory** tells us that system components act and react to each other and will regulate themselves using cooperation and competition to pursue their goals.

Though we let the number of states and interdependencies increase, we have a level left for us to pull - reversibility.

#### Reversibility

If the effects of a decision can't always be predicted, then it is expensive if that decision can't be reversed.

Practices such as frequent pushes of small changes and canary releases allow us to minimise the impact of a bad change, and roll it back as soon as we can.

When we make changes to the system we can monitor metrics such as latency and error rate to ensure we're not unintentionally degrading the user experience.
We can also monitor the effect of new features to measure the impact and to see if they add value.

### Chaos Engineering

Chaos Engineering is the practice of running experiments to uncover systemic weakness. It helps us to understand how much failure a system can tolerate and still operate within acceptable boundaries.

We can define the expected behaviour of our system under normal conditions and then hypothesise that this will still be true during real-world events.
For example, _"We expect 99% availability while processing 200 orders per second while 30% of nodes are unavailable."_

We can then run an experiment to either prove the theory or learn something new.

This can reduce the on-call burden not only by giving higher confidence in the system but can serve as on-call training. Engineers get a sense of the chaos already in the system, become familiar with the observability tools, and are engaged and focused on resilience.

### System design and team structure

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of the system will reflect the organization that built it.
It follows that to get to the system we want to build we must first structure our organisation in that way.

Ensuring the team understands the domain and using consistent language will flow down into the code.

Keeping teams well aligned with shared values, principles, and practices will help keep a consistent codebase.

## Conclusion

Distributed systems are part of life working on modern software. We must understand the compromises we make with each decision - whether it's added complexity and maintenance, degraded experience, or just more expensive - there will always be a cost to added resilience.

- 🛡 Protect resources where possible
- 💡 Be aware of the trade-offs introduced by a pattern
- 🕵️‍♀️ Ensure the system is observable
- 🧪 Test knowns and experiment for unknowns
- 🤓 Learn, improve, repeat
