---
layout: post
title:  "Resilience in Distributed Systems"
tags: distributed-systems resilience design architecture
---

Modern systems are increasingly complex and distributed.
This article outlines some patterns and practices for adding resilience to complex, distributed systems.

First a few defintions,

- **Resilience** - Resilience is the ability to maintain acceptable service levels during a system failure
- **Complex Systems** - A system is complex if there are enough moving parts that an individual cannot hold the model in their head, the outputs are unpredictable given the inputs
- **Distributed Systems** - A system is distributed if there are multiple components spread over multiple hosts

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must make a decision for each operation when we encounter a failure; wait, retry or cancel?

## Understanding requirements of the system

Understanding the requirements of a system is important as it will help to guide our decisions for which behaviours are essential and which can be comprimised.

Deciding what an acceptable response time and level of availability is a good place to start.

### Properties of distributed systems

- **Fault tolerant** - Can recover from component failures without performing incorrect actions
- **Highly available** - Can restore operations, allowing it to resume service after a component has failed
- **Consistent** - Can coordinate actions by multiple components in the presence of concurrency and failure
- **Transparent** - The user is unaware that processes and components are distributed across multiple computers
- **Cost efficient** - We must keep spending in check to keep the product adding value

## Observability

Observability is key to be able to navigate operations through a complex distributed system. It gives us operational visibility and helps us understand system bottlenecks.

We can define expected behaviour of our system under normal conditions, for example "We expect 99% availability while processing 200 orders per second".

When we make changes to the system we can monitor metrics such as latency and error rate to ensure we're not unintentionally degrading the user experience.

## Design patterns

So what patterns can we use to add resilience into our system?

### Caching

Caching is good for operations that prefer to return something quickly than to return the most up to date data. If there are many requests using the same data caching, even over a small amount of time, can be beneficial. This can also be done at various parts of the stack.

The system should eventually update and be correct and this type of system is known as eventually consistent.

Can we prempt what the user will request and cache the value ahead of time?

### Isolation

Critical operations can be isolated onto separate components and their resources can be isolated, this prevents them from being affected by failure in other parts of the system.

Isolation also allows teams to independently maintain different system capabilities.

#### Coupling

Components are coupled if changing one implies changing the other. Coupled systems are hard to change.

### Events and Queues

Using events for durability, capture the request in a queue and processing it without overwhelming the system.

Increases availability by allowing an operation to resume after a component has failed.

Take the following example

![]()

A partition failure between the edge API and processor could cause the request to fail, similarly if the processor crashes or is overwhelmed by requests.

![]()

The request is captured in a queue which can be processed when there are available resources.
Failed operations can be requeued and reprocessed when the system is healthy.
The queue length can be monitored and the processor can be scaled horizontally to handle more requests.

New issues are introduced here
- **Duplicates** - If the operation is not [idempotent](https://en.wikipedia.org/wiki/Idempotence) we must add a request identifier and de-duplicate
- **Ordering** - An earlier message could undo the work of a more recent message, this can be configured in a queue, or potentially dealt with using timestamps or versions

Event driven architecture is loosely coupled, the event doesn't know about the consequences it can cause.

### Degrading functionality

Is it sufficient to return a partial response?

Can we turn off parts of the system to protect other parts?

#### Bulkheads

A bulkhead is a term taken from shipping where the ship is designed so that in the event of a failure a part of a ship can be sealed off to protect the rest of the ship. Similarly we can design our systems into isolated components that can be switched off in the event of failure to allow the rest of the system to continue running smoothly.

#### Feature toggle

Feature toggle can be used to manually enable and disable features and paths without a deployment.

#### Circuit breakers

Logic for automatically sealing a bulkhead on certain failures, requests can be intermittently let through to test if the downstream resources are available before resuming service.
This protects downstream resources and allows the application to continue processing other potentially successful operations.

#### Timeouts and Retries

If a call is taking a long time in a distributed system, it could be because it is connected to a bad or overwhelmed instance and sometimes it is better to cancel the call and retry.

"A long time" could be different for each operation and we can use our observability tools to guide us here.

Again if the process is not idempotent then the request can only be retried if provisions have been put in place to deal with duplicates.

### Scaling

Adding more resources to be able to handle more requests.
This can be _horizontal_ by adding more similar resources, or _veritical_ by increasing the size of existing resources.

Must be cautious of overwhelming downstream resources.

## Other considerations

Defining domain boundaries lowers cognitive complexity for engineers.

Using common language between engineers and domain experts, and ensuring the whole team understand _why_ will contribute towards them making the right decision and keeping the code clean.

A step further would be having the whole team interacting with customers and involved in product discovery.

### System design and team structure

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of system will reflect the organization that built it. This implies that loosely coupled systems are created by loosely coupled teams.

Though loosely coupled, teams should be highly aligned. Visibility between teams helps to form shared values, principles and practices.

One person cannot hold all pieces of the system in their head and that the responsibility of system design is distributed among the team, we must all understand the trade offs of charateristics made with each decision.

## Testing system resilience

It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

By accepting that the system will fail we can experiment and learn how it will react under certain conditions.

Chaos Engineering is a practice where you run experiments on a system to observe how it reacts under a fault. This allows you to monitor system failure in controlled setting instead of waiting for the failure to happen out of business hours.

### Chaos Engineering

1. Define steady state of system.
2. Build hypothesis around steady state behaviour under failure conditions, for example "We expect the system to maintain 99.9% availability while handling 200 requests per second while 20% of nodes are failing".
3. Run an experiment to test the theory, build confidence in test environment and work towards being able to run in production to see how it reacts under real load.
4. Verify - did something unexpected happen?
5. Improve system using learnings from experiment, redefine steady state and go again!

![one](maisiesadler/maisiesadler.github.io/assets/chaos-eng.png)
![one](maisiesadler/maisiesadler.github.io/docs/assets/chaos-eng.png)
![again](./chaos-eng.png)

![two](./img/chaos-eng.png)

Start with one-off experiments or game days. When the experiment is well defined it can be automated and ran continuously.

- Choose experiments based on real world events and incidents
- Once confident, run the experiment in production
- Have a rollback plan in place and revert once you have learned something

### Benefits

- Higher confidence in system
- Increased understanding of resilience of system
- Serves as on call training and reduces on call burden
- Engineers familiar with observability tools
  - Engaged and focused on resilience
  - Have these tools in mind when imlementing new features

## Conclusion

Distributed systems are part of life working on modern software and it's important we understand .?

- 🤔 Review if splitting up a component is the right thing to do
- 🕵️‍♀️ Ensure the system is observable
- 📝 Hypthesise about the behaviour of your system under failure
- 🧪 Test the failure in a controlled experiment
- 🧪 Test knowns and experiment for unknowns
- 🤓 Learn, improve, repeat
