---
layout: post
title:  "Distributed Systems"
tags: distributed-systems
---

A system is distributed if there are multiple components spread over multiple hosts. This might be to connect more users or to be resilient and maintain availability through failure.
Or it could be to isolate critical services or allow teams to independently maintain different system capabilities.

One way a system becomes distributed is by splitting different capabilities into different components and another is to horizontally scale a component to introduce resilience to faults.

Operations through a distriuted system must design around hardware and software failures; wait, retry or cancel?

Is the operation querying for data or triggering an operation? Does the query need to collect or process a lot of data to respond? Does the requester need up to date data or is it ok to be slightly stale? Can we prempt what a user might want and pre process the response ahead of time? Would the user prefer to wait for the correct response, do they mind coming back later or if we let them know when it's done?

New problems -> identifiers, a common language for them to be able to ask what has happened, and for us to find out in the system.

What is the bottleneck? How can we protect downstream resources or databases from being oberwhelmed? Isolate resources for critical operations so they are not effected by other issues in the system.

In this article I will discuss some of the considerations to be made when designing a distributed system

A distributed system must design around failure of components and 
A well designed distributed system will be fault tolerant, highly available, consistent, recoverable and transparent to the end user.

Modern systems are likely to have at least some distribution between components and must be weary of the trade offs involved when introducing more distribution into the system.

Might be introducing distribution to allow new products to be added, teams to work in isolation.

A distributed system is concurrent by design, many processes will be executing at the same time.

To be able to keep the cognitive complexity for a team down we define domain boundaries matching the business for a team to work within.

Fallacies of distributed systems tell us that the system will fail and we must ensure we have a plan in place for this eventuality.

## Challenges

Before we discuss the workaround for failures in a system, it's good to discuss the type of failures that occur and that we must design for.

- Security
- Fault tolerance
  - node failure
  - partition failure
- Coordination

Security ... networking, could be helped by things likea....
Zero trust networking, fine grained controls, minimal permissions?

Coordinaton - components in distributed systems run simulataneously and lack a global clock which can make it hard to design around.

I want to focus mainly on designing around fault tolerance - node failure and partition failure.

## Understanding requirements of the system

It is important to understand the requirements of a system as the architecture and patterns we choose will likely be a trade off between certain characteristics. Defining what is an acceptable response time and level of availability is a good place to start.

We must continuously ask ourselves what happens if certain parts of the system fail.

## Observability

Monitor how changes to the system effect latency and availability. Was it the right decision. Important to collect data and revert if it isn't helping. It's a bet and we can learn and retry.

Use observability to find bottlenecks.

## Characteristics

What characteristics are available and do we want to optimise for?

There are some charecteristics available to us in a distributed system and given the fallacies listed above we cannot have all of them all the time. There are different patterns available that optimise for certain characteristics at the expense of others

- **Fault tolerant** - Can recover from component failures without performing incorrect actions
- **Highly available** - Can restore operations, 

If not fault tolerant, a system might require some manual adjustments after a failure.

## Design patterns

Ok ok get to it Maisie...

### Caching

Caching is good when it is more important to return something quickly than to return the most up to date data or if the data does not change very often. If there are many requests using the same data caching, even over a small amount of time, can be beneficial. This can also be done at various parts of the stack.

The system should eventually update and be correct and this type of system is known as eventually consistent.

### Events

If the work to be done to execute a command or retrieve data for a request could take a lot of time or processing power then events could be a good choice for coordinating this work.

Events allow us to durably capture a request and process it without overwhelming the system. If latency is becoming an issue then the component processing the messages from the queue can be horizontally scaled. If there is an error while processing then the messages can be requeued and reprocesssed.
The queue might require the processing application to explicitly delete the message once it has completed, in this case either the message should be idempotent or a way to de-duplicate the request before a critical point.

One way to achieve availability and return the latest response if the client does not need a response immediately is to use events. , for each incoming request save the event to a durable queue and . Because the initial request is simple, it is much more likely to return successfully.

Increases availability by allowing an operation to resume after a component has failed.

### Isolating resources

### Degrading functionality

Is it sufficient to return a partial response?

Can we turn off parts of the system to protect other parts?

Consider feature toggle for quickly enabling and disabling new features and paths.

#### Bulkheads and Circuit breakers

A bulkhead is a term taken from shipping where the ship is designed so that in the event of a failure a part of a ship can be sealed off to protect the rest of the ship. Similarly we can design our systems into isolated components that can be switched off in the event of failure to allow the rest of the system to continue running smoothly.

A circuit breaker is a mechanism to automatically seal a bulkhead, protecting downstream resources and also not exhausting the application and allowing it to continue processing other potentially successful operation.

### Timeouts and Retries

If a call is taking a long time in a distributed system, it could be because it is connected to a bad or overwhelmed instance and so it is worth considering if it is better to cancel the call and retry.

It is worth considering how long is "a long time" for each operation and adjusting this logic accordingly.

### Use these requirements to drive architectural decisions

A plan in place for when things do go down
- Can we degrade functionality?
  - Is it sufficient to return a partial response?
- Implement circuit breaker to fail faster to clients and allow downstream to recover
- Bulkhead where functionality is taken out of service to allow other parts to run successfully
  - If something is saturating network or has high CPU it can be switched off to save the ship
  - The more isolated components are the less likely this will be necessary
- Idempotency and retries
  - If a call is idempotent it can be retried

## Coupling

- coupling related to cost of change
- highly coupled systems are hard to change

## Scaling

Scaling
- adding more resources
- adding bigger resources
- worker processes for async tasks and batch jobs

Must be cautious of overwhelming downstream resources or availability

## Event based systems

Using events for durability
- Capture request as an event, low failure
- Request can be processed and retried on failure
- Return when completed

Can use events to cache for writes
- Save data to a queue and process the messages in a worker service to avoid overloading the database
- Need to use a durable queue.

Event driven architecture is loosely coupled because the event doesn't know about the consequences of it's cause.

Events allow asynchronous processing but are not a way to decouple components. The producing and consuming components share a contract for the event and face the same challenges to contract changes as a direct call, if not more complicated because an unknown number of consumers could be using the event.

## Databases

Scaling stateless systems is easy, but what about when you have data.

Again it depends on the use case,
A AP system that is prioritising Availability over consistency can 
 if optimising the system to scale for reads and the system can be eventually consistent
--> also adds resilience

If the system is scaling for writes, then a databae that supports sharding might be a good solution.

### Consistency or Availablity

CAP theorem states that a system must choose 2 out of Consistency, Availability and Partition Tolerance.

Network partitions will happen so a distributed system must decide whether to be Available or Consistent. In other words should it cancel the operation reducing availability or should it proceed with the operation risking inconsistency?
A system favouring availability is known as eventually consistent, it favours low latency and successful responses over returning the most up to date data.
A system favouring consistency is known as strongly consistent, it favours up to date data over latency.

## Resilience

The fallacies of distributed systems tell us to design expecting an unreliable network, latency and bounded bandwidth.

## Other considerations

Team structure

Conways law, domain boundaries

XP teams, good practices, lots of small changes ability to roll back, focus on observability.

### System design and team structure

Teams should be able to move independently, High performing teams are highly aligned and loosey coupled.

Conway's law
- loosely coupled teams create loosely coupled systems?

alignment, values -> principles and practices
- Visibility over what other teams are doing, problems that are being solved

## Testing system resilience

It is good to understand how much failure a system can tolerate and still operate within acceptable boundaries.

Chaos Engineering is a practice where you run experiments on a system to observe how it reacts under a fault. This allows you to monitor system failure in controlled setting instead of waiting for the failure to happen out of business hours.

Be of the mindset that it will fail and force it to happen in hours when you can be around to monitor observe and improve.

making system robust

### Chaos Engineering

A practice to gain confidence in the uncertainty of distributed system by facilitating experiments to uncover systemic weaknesses.

One off experiments
Game days
Automated continuous failures

Define steady state of system.

Create a hypothesis about how the system will perform under failure, for example "We expect the system to maintain 99.9% availability while handling 200 requests per second while 20% of nodes are failing".

Run an experiment to test the theory, build confidence in test environment and work towards being able to run in production to see how it reacts under real load.

Verify - did something unexpected happen?

Improve system using learnings from experiment, redefine steady state and go again!

Once happy with the results, 
The test can be automated and re run periodically to act as a regression test.

### Benefits

- Higher confidence in system
- Increased understanding of resilience of system
- Serves as on call training and reduces on call burden
- Engineers familiar with observability tools
  - Engaged and focused on resilience
  - Have these tools in mind when imlementing new features

## Conclusion

Distributed systems are part of life working on modern software

- 🤔 Review if splitting up a component is the right thing to do
- 🕵️‍♀️ Ensure the system is observable
- 📝 Hypthesise about the behaviour of your system under failure
- 🧪 Test the failure in a controlled experiment
- 🤓 Learn, improve, repeat

---------

## Product thinking and knowing your customers

Domain driven design and having a shared language between engineers and domain experts.

Extend this to having shared language between customers and engineers.

## XP Processes

Things to get in
- Product thinking
- DDD
- XP and TDD?
  - Need this basic layer in to be able to scale successfully
- Coupling
- CAP Theorem
- what happens if the process crashes right now?
- Defining domain boundaries

