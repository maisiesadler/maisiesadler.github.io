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

Desirable properties of distributed systems

- **Fault-tolerant** - Can recover from component failures without performing incorrect actions
- **Highly available** - Can restore operations, allowing it to resume service after a component has failed
- **Consistent** - Can coordinate actions by multiple components in the presence of concurrency and failure
- **Transparent** - The user is unaware that processes and components are distributed across multiple computers
- **Cost-efficient** - We must keep spending in check to keep the product adding value

Understanding the requirements of the system is important as it will help to guide our decisions for which behaviours are essential and which can be compromised.

The [fallacies of distributed systems](http://wiki.c2.com/?EightFallaciesOfDistributedComputing) tell us that we cannot rely on the network to be reliable or secure. We must make a decision for each operation when we encounter a failure; wait, retry or cancel?

## Observability

Observability is key to be able to navigate operations through a complex distributed system. It gives us operational visibility and helps us understand system bottlenecks.

We can define the expected behaviour of our system under normal conditions, for example, "We expect 99% availability while processing 200 orders per second".

When we make changes to the system we can monitor metrics such as latency and error rate to ensure we're not unintentionally degrading the user experience.

## Design patterns

So what patterns can we use to add resilience into our system?

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

### Isolation

Critical operations can be isolated into separate components and their resources can be isolated, this prevents them from being affected by failure in other parts of the system.

Isolation also allows teams to independently maintain different system capabilities.

### Degrading functionality

Is it sufficient to return a partial response?

Can we turn off parts of the system to protect other parts?

#### Bulkheads

A bulkhead is a pattern where part of the functionality is switched off to allow the rest of the system to continue.
The system is designed into isolated components that can be switched off in the event of failure.

#### Feature toggle

Feature toggle can be used to manually enable and disable features and paths without a deployment.

#### Circuit breakers

Logic for automatically sealing a bulkhead on certain failures, requests can be intermittently let through to test if the downstream resources are available before resuming service.
This protects downstream resources and allows the application to continue processing other potentially successful operations.

#### Timeouts and Retries

If a call is taking a long time in a distributed system, it could be because it is connected to a bad or overwhelmed instance and sometimes it is better to cancel the call and retry.

"A long time" could be different for each operation and we can use our observability tools to guide us here.

Similar to events, if the operation is not idempotent then the request can only be retried if provisions have been put in place to deal with duplicates.

## Tackling complexity

Although complex systems can be unpredictable, they share some common traits that can help us model how they might behave.

Complexity theory gives us a set of tools to understand behaviours and what levers we might have to ... . 
Levers that we can pull for certain behaviours

Complex systems are made up of individual components interacting with each other, as a system scales it becomes more complex.
The dependencies, relationships and interactions of these componenets make them hard to model. 

The methods we use to add resilience add more complexity but this isn't necessarily a bad thing.
Though we let the number of states and interdependencies increase, we have a level left for us to pull - reversibility.
If we introduce a behaviour into the system that wasn't expected then we can revert the change, learn and try again.

Theory emerging in many domains,
Tackled by many different diciplines into a core set of common features known as complexity theory.

Theoretical framework for modelling complex systems in a variety of domains.

- **Emergence** - Isolated component behaviour leads to non-aparent system behaviour
- **Non-linearity** - Given the same input, a system response might be different given different state
- **Adaptive** - Systems regulate themselves with adaptive agents cooperation and competition

Although seemingly random the outputs are governed by the inputs, just unpredictable.

Chaos theory is the theory that chaos systems, though unpredictable, are not random and that there are patterns that govern their behaviour.
Governed by feedback loops not linear equations.

Prediction is difficult even though system is deterministic, not random.

### Irreversibility

Effects of decisions can't be predicted, it is expensive if decision can't be reversed.

Most available lever to control complexity in software.

Frequent pushes, using observabilty, 
Letting data inform if a feature has been good or no

### System design and team structure

Our software solution is a complex system, and our organisation is a complex system. [Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of the system will reflect the organization that built it and so it is interesting to consider both systems when modelling complexity.

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of the system will reflect the organization that built it.
For this reason it is interesting to consider both the software system and the organisation when thinking about complexity in our system.

Self-organisation model tells us that global patterns form out of local interactions.

Adaptive theory tells us that systems will regulate themselves using cooperation and competition to persue their goals.
- How a learner adapts and reacts to stimuli

Imagine system we are trying to build and start with team structure

- Keep teams aligned with shared values, principles, and practices
- Focus on quality
- Defining domain boundaries lowers the cognitive complexity for engineers
- Ensuring the whole team understand _why_ helps them to make the right decision and keep the code clean
- Test first approach to ensure we only write the code we need

[Conway's law](https://www.thoughtworks.com/insights/blog/demystifying-conways-law) tells us that that the structure of the system will reflect the organization that built it. It follows that loosely coupled systems are created by loosely coupled teams.

## Testing system resilience

It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

By accepting that the system will fail we can experiment and learn how it will react under certain conditions.

### Chaos Engineering

Chaos Engineering is the practice of running experiments on a system to observe how it reacts.
This builds confidence in the resilience of the system as it allows you to monitor system failure in a controlled setting.

1. Define steady state of the system
2. Build hypothesis around steady-state behaviour under failure conditions, for example "We expect the system to maintain 99.9% availability while handling 200 requests per second while 20% of nodes are failing"
3. Run an experiment to test the theory
4. Verify - did something unexpected happen?
5. Improve system using learnings from experiment, redefine steady state and go again!

Start with one-off experiments or game days. When the experiment is well defined it can be automated and run continuously.

- Choose experiments based on real world events and incidents
- Once confident, run the experiment in production
- Have a rollback plan in place and revert once you have learned something

Engineers get a sense of the chaos already in the system, 

### Benefits

- Higher confidence in the system
- Increased understanding of resilience of system
- Serves as on-call training and reduces on-call burden
- Engineers familiar with observability tools
  - Engaged and focused on resilience
  - Have these tools in mind when implementing new features

## Conclusion

Distributed systems are part of life working on modern software. We must understand the compromises we make with each decision - whether it's added complexity and maintenance, degraded experience, or just more expensive - there will always be a cost to added resilience.

🛡 Protect resources where possible
💡 Be aware of the trade-offs introduced by a pattern
🕵️‍♀️ Ensure the system is observable
🧪 Test knowns and experiment for unknowns
🤓 Learn, improve, repeat
