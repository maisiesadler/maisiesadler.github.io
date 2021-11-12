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

## Observability

To be able to navigate operations through a complex distributed system observability is key as it gives us operational visibility and helps us understand system bottlenecks.

We can define the expected behaviour of our system under normal conditions, for example, "We expect 99% availability while processing 200 orders per second".

When we make changes to the system we can monitor metrics such as latency and error rate to ensure we're not unintentionally degrading the user experience.

## Encountering Failure

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must decide for each operation when we encounter a failure; wait, retry or cancel?

### Wait

How likely is it that the result is just about to be returned?

If a network connection is open for an extended period not only does it have more time to fail but it is consuming resources and could block potentially successful calls.

Use observability to guide what is an acceptable time to wait and use timeouts to cancel after it is unlikely to return successfully.

### Retry

In a distributed system it is possible that a process is hanging due to a transient failure and that retrying the call could be successful. This can be done in memory or by re-queuing a message.

Some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - Out of order messages can apply updates incorrectly, this can potentially be dealt with using timestamps or versions
- **Retry Storm** - Too many retries can end up doing more harm than good

### Cancel

Is it acceptable to degrade certain functionality if it means other, potentially more critical, operations can continue?

Isolated functionality can be switched off using bulkheads and circuit breakers, this protects downstream resources and also allows the application to continue processing other potentially successful operations.
Traffic can be intermittently let through to test if service can be resumed.

## Preventing Failure

There are some patterns we can use to attempt to reduce the chance of encountering a failure. However, we must be aware of the trade-offs.

### Isolation

Critical operations can be isolated into separate components and their resources can be isolated, this prevents them from being affected by failure in other parts of the system.

Isolation also allows teams to independently maintain different system capabilities.
A large system can be split into logical [bounded contexts](https://martinfowler.com/bliki/BoundedContext.html) with explicitly defined relationships.

Within these domain boundaries there is a lower complexity since there are fewer states and interactions that can occur.

### Scaling

Adding more resources to be able to handle more requests.
This can be _horizontal_ by adding more similar resources, or _veritical_ by increasing the size of existing resources.

Must be cautious of overwhelming downstream resources.

### Caching

Is it ok to return slightly stale data if it can be accessed much more quickly?

Can we preempt what the user will request and cache the value ahead of time?

If so, [caching](https://aws.amazon.com/caching/) the data could be a good option to increase availability and reduce load on the rest of the system.

The system should eventually update and be correct and this type of system is known as eventually consistent.

### Events and Queues

Events can be used to durably capture a request for an operation.
This increases availability by allowing an operation to resume after a component has failed.

A component consuming messages from a queue can process messages at a steady rate.
This rate can be increased by scaling the component, ensuring the rate isn't so high we start overwhelming downstream resources.

This pattern introduces some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - An earlier message could undo the work of a more recent message, this can be configured in a queue, or potentially dealt with using timestamps or versions

Event-driven architecture is loosely coupled, the event doesn't know about the consequences it can cause.

## Tackling complexity

Complex systems are made up of individual components interacting with each other, as a system scales it becomes more complex.
The dependencies, relationships, and interactions of these components make them hard to model. 

These systems often exhibit non-linear behaviour, meaning that the response can be different given the same input depending on state and similar inputs does not mean similar outputs.

Although seemingly random the outputs are governed by the inputs, just unpredictable.
Complexity theory gives us a set of tools to understand the patterns and behaviours that occur in these types of systems.

Chaos theory is the theory that chaos systems, though unpredictable, are not random and that there are patterns that govern their behaviour.
Governed by feedback loops, not linear equations.

Self-organisation model tells us that global patterns form out of local interactions.

Adaptive theory tells us that system components act and react to each others behaviours and will regulate themselves using cooperation and competition to pursue their goals.

The methods we use to add resilience add more complexity but this isn't necessarily a bad thing.
Though we let the number of states and interdependencies increase, we have a level left for us to pull - reversibility.

### Reversibility

If the effects of a decision can't be predicted, then it is expensive if that decision can't be reversed.

Practices such as frequent pushes of small changes and canary releases allow us to minimise the impact of a bad change, and roll it back as soon as we can.
New features can be monitored to measure the impact and to see if they add value.

### Verifying Resilience


It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

#### Chaos Engineering

Chaos Engineering is the practice of running experiments to uncover systemic weakness.

Start by creating a hypothesis about the system behaviour during real world events and aim to either prove the theory or learn something new.

- Start with one-off experiments or game days, when the experiment is well defined it can be automated and run continuously
- Choose experiments based on real-world events and incidents
- Once confident, run the experiment in production
- Have a rollback plan in place and revert once you have learned something

This can reduce the on-call burden not only by giving higher confidence in the system but can serve as on-call training. Engineers get a sense of the chaos already in the system, become familiar with the observability tools and are engaged and focused on resilience.

### System design and team structure

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of the system will reflect the organization that built it.
It follows that to get to the system we want to build we must first structure our organisation in that way.

- Keep teams aligned with shared values, principles, and practices
- Focus on quality
- Ensuring the whole team understand _why_ helps them to make the right decision and keep the code clean
- Teams with well defined responsiblities


## Conclusion

Distributed systems are part of life working on modern software. We must understand the compromises we make with each decision - whether it's added complexity and maintenance, degraded experience, or just more expensive - there will always be a cost to added resilience.

🛡 Protect resources where possible
💡 Be aware of the trade-offs introduced by a pattern
🕵️‍♀️ Ensure the system is observable
🧪 Test knowns and experiment for unknowns
🤓 Learn, improve, repeat
