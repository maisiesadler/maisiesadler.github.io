---
layout: post
title:  "Resilience in Distributed Systems"
tags: distributed-systems resilience design architecture
---

This article outlines some patterns and practices for adding resilience to complex, distributed systems.

First a few defintions,

- **Resilience** - Resilience is the ability to maintain acceptable service levels during a system failure
- **Complex Systems** - A system is complex if there are enough moving parts that an individual cannot hold the model in their head, the outputs can be unpredictable
- **Distributed Systems** - A system is distributed if there are multiple components spread over multiple hosts

Modern systems are increasingly complex and distributed.
Every engineer must understand the trade offs of every decisions and keep observability and resilience in mind.

Desirable properties of distributed systems

- **Fault tolerant** - Can recover from component failures without performing incorrect actions
- **Highly available** - Can restore operations, allowing it to resume service after a component has failed
- **Consistent** - Can coordinate actions by multiple components in the presence of concurrency and failure
- **Transparent** - The user is unaware that processes and components are distributed across multiple computers
- **Cost efficient** - We must keep spending in check to keep the product adding value

Understanding the requirements of the system is important as it will help to guide our decisions for which behaviours are essential and which can be comprimised.

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must make a decision for each operation when we encounter a failure; wait, retry or cancel?

## Observability

Observability is key to be able to navigate operations through a complex distributed system. It gives us operational visibility and helps us understand system bottlenecks.

We can define expected behaviour of our system under normal conditions, for example "We expect 99% availability while processing 200 orders per second".

When we make changes to the system we can monitor metrics such as latency and error rate to ensure we're not unintentionally degrading the user experience.

## Design patterns

So what patterns can we use to add resilience into our system?

### Scaling

Adding more resources to be able to handle more requests.
This can be _horizontal_ by adding more similar resources, or _veritical_ by increasing the size of existing resources.

Must be cautious of overwhelming downstream resources.

### Caching

Increase availability and resilience by [caching](https://aws.amazon.com/caching/) the result to queries that might be accessed again or precalculating the result of something that will likely be queried later.
The trade off here is that the user will not always see the most up to date data.

If many queries now access the cache instead of getting the latest data this can greatly reduce load on the rest of the system.

### Events and Queues

Events can be used to durably capture a request for an operation.
This increases availability by allowing an operation to resume after a component has failed.

A component consuming messages from a queue can process messages at a steady rate.
This rate can be increased by scaling the component, ensuring the rate isn't so high we start overwhelming downstream resources such as databases.

This pattern introduces some complications to be aware of
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - An earlier message could undo the work of a more recent message, this can be configured in a queue, or potentially dealt with using timestamps or versions

### Isolation

Critical operations can be isolated into separate components and their resources can be isolated, this prevents them from being affected by failure in other parts of the system.

Isolation also allows teams to independently maintain different system capabilities.

### Degrading functionality

Can we turn off parts of the system to protect other parts?

A bulkhead is a pattern that does exactly that, the system is designed such that isolated functionality can be switched of in the event of failure.

This could be a manual process with feature toggle. Or automated with circuit breakers.

Circuit breakers switch off functionality on given error conditions and intermittently let traffic through to test if service can be resumed.
This protects downstream resources and also allows the application to continue processing other potentially successful operations.

#### Timeouts and Retries

If a call is taking too long in a distributed system, it could be because there is a transient fault with the network or a component downstream and sometimes it is better to cancel the call and retry.

"Too long" could be different for each operation and we can use our observability tools to guide us here.

Similar to events, if the operation is not idempotent then the request can only be retried if provisions have been put in place to deal with duplicates.

## Testing system resilience

It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

### Chaos Engineering

[Chaos Engineering](https://principlesofchaos.org/) is a practice of running experiments on a system to observe how it reacts.

This can reduce on call burden not only by giving the engineers higher confidence in the system, but can serve as on call training. Engineers become familiar with the observability tools and are engaged and focused on resilience.

- Start with one-off experiments or game days
- Choose experiments based on real world events and incidents
- Once confident, run the experiment in production
- Have a rollback plan in place and revert once you have learned something

## Other considerations

Although the system is complex we can do our best to minimise accidental complexity.

- Defining domain boundaries lowers cognitive complexity for engineers
- Ensuring the whole team understand _why_ helps them to make the right decision and keep the code clean
- Test first approach to ensure we only write the code we need

### System design and team structure

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of system will reflect the organization that built it. It follows that loosely coupled systems are created by loosely coupled teams.

Though loosely coupled, teams should be highly aligned. Visibility between teams helps to form shared values, principles and practices.

## Conclusion

Distributed systems are part of life working on modern software and it's important we understand the trade offs we make with each decision.

- 🤔 Review if splitting up a component is the right thing to do
- 🕵️‍♀️ Ensure the system is observable
- 📝 Hypthesise about the behaviour of your system under failure
- 🧪 Test knowns and experiment for unknowns
- 🤓 Learn, improve, repeat
