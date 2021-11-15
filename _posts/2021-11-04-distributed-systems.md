---
layout: post
title:  "Resilience in Distributed Systems"
tags: distributed-systems resilience design architecture
---

This article explores the decisions we must make when we encounter failure, some patterns we can use to increase resilience and prevent failure and finally how embracing the failure within the system can lead us to more appropriate models and tooling.

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

It is possible that an operation failed due to a transient issue and that retrying the call could be successful.

Some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must mitigate this, for example by adding a request identifier and de-duplicating on the server-side
- **Ordering** - Out-of-order messages can cause updates to be applied incorrectly, we can timestamps or versions to help us here
- **Retry Storm** - If many requests are cancelled at the same time then retrying at fixed intervals it can make it hard for a system to recover

### Cancel

Is it acceptable to degrade certain functionality if it means other, potentially more critical, operations can continue?

A bulkhead is a pattern that does exactly that, the system is designed such that isolated functionality can be switched of in the event of failure. This could be a manual process with feature toggle or automated with circuit breakers.

Circuit breakers switch off functionality on given error conditions and intermittently let traffic through to test if service can be resumed.
This protects downstream resources and also allows the application to continue processing other potentially successful operations.

## Preventing Failure

There are some patterns we can use to attempt to reduce the chance of encountering a failure.
However, we must be aware of the trade-offs.

### Isolation

Critical operations can be split up into separate components and their resources can be isolated, this prevents them from being affected by failure in other parts of the system.

Isolation also allows teams to independently maintain different system capabilities.
A large system can be split into logical [bounded contexts](https://martinfowler.com/bliki/BoundedContext.html) with explicitly defined relationships.

Within these domain boundaries there is less complexity since there are fewer states and interactions that can occur.

<note on coupling here?>

### Scaling

Adding more resources to be able to handle more requests.
This can be _horizontal_ by adding more similar resources, or _veritical_ by increasing the size of existing resources.

This should be controlled so we do not overwhelm downstream resources.

### Caching

Is it ok to return slightly stale data if it can be accessed much more quickly?

Can we preempt what the user will request and precalculate the value ahead of time?

If so, [caching](https://aws.amazon.com/caching/) the data could be a good option to increase availability and reduce load on the rest of the system.

When we encounter a network failure we do not have partition tolerance and so the decision must be consistency or availability ([CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)).
Using caching to increase availability is at the expense of consistency, the system should eventually update and become consistent. This pattern is known as eventually consistent.

Caching is not one size fits all and the logic around how to access a cache could be different per operation.
It adds complexity and, if we're not careful, can give another place for something to go wrong.

### Events and Queues

Events can be used to durably capture a request for an operation.
This increases availability by allowing an operation to resume after a failing component has recovered.

The consuming component can process messages at a steady rate, this rate can be increased by scaling the component.

Event-driven architecture is loosely coupled, not decoupled and so we are still bound to contracts. The publishing component is not aware of how it's events are used and the consequences it can cause.

This pattern brings with it complexity as it can become complicated if the operation needs to be awaited.
It can be hard to monitor and trace an operation as it flows through the system.

## Embracing Failure

With every new feature our software systems have more states and interactions making them harder to model.
Since complex systems are made up of individual components interacting with each other, as a system grows and scales it becomes more complex.

We add more logic and states to increase resilience and in turn add more complexity.

All is not lost, if we accept our system is and will continue to be complex then we can look to [complexity theory](https://www.youtube.com/watch?v=i-ladOjo1QA&list=TLPQMTAxMTIwMjHtudT33UyVOA) to give us a set of tools to understand the patterns and behaviours we are seeing.

### Verifying Resilience

[Chaos Engineering](https://principlesofchaos.org/) is the practice of running experiments to uncover systemic weakness. It helps us to understand how much failure a system can tolerate and still operate within acceptable boundaries.

We can define the expected behaviour of our system under normal conditions and then hypothesise that this will still be true during real-world events.
For example, _"We expect 99% availability while processing 200 orders per second while 30% of nodes are unavailable."_

We can then run an experiment to either prove the theory or learn something new.

This can reduce the on-call burden not only by giving higher confidence in the system but can serve as on-call training. Engineers get a sense of the chaos already in the system, become familiar with the observability tools, and are engaged and focused on resilience.

### Reversibility

In an article about [tackling complexity](https://m.facebook.com/nt/screen/?params=%7B%22note_id%22%3A681695435785808%7D&path=%2Fnotes%2Fnote%2F&refsrc=deprecated&_rdr) Martin Fowler argues that complexity is made up of four factors; states, interdependencies, uncertainty and irreversibility. In many software systems the states, interdependencies and uncertainty inevitably grow and so we have one lever left for us to pull - reversibility. If the effects of a decision can't always be predicted, then it is expensive if that decision can't be reversed.

Practices such as frequent pushes of small changes and canary releases allow us to minimise the impact of a bad change, and roll it back as soon as we can.

When we make changes to the system we can monitor metrics such as latency and error rate to ensure we're not unintentionally degrading the user experience.
We can also monitor the effect of new features to measure the impact and to see if they add value.

### System design and team structure

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of the system will reflect the organization that built it.
It follows that to create a decoupled system with isolation and certain boundaries we must first structure our organisation in that way.

We can help to reduce accidental complexity by ensuring the team understands the domain and using consistent language.

Keeping teams well aligned with shared values, principles, and practices will help keep a consistent codebase.

## Conclusion

Distributed systems are part of life working on modern software and over time the system grows and becomes more distributed and complex.
We add more complexity to add resilience and do our best to prevent accidental complexity.
But chaos and complexity will inevitably prevail in distributed systems, if we accept this then we can begin to use better-suited models and techniques.

- 🛡 Protect resources where possible
- 💡 Be aware of the trade-offs introduced by a pattern
- 🕵️‍♀️ Ensure the system is observable
- 🧪 Test knowns and experiment for unknowns
- 🤓 Learn, improve, repeat
