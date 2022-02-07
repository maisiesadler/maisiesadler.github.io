---
title: Hexagonal Architecture
layout: note
tags: design
---

Hexagonal architecture, or ports and adapters, is a pattern where the domain/business logic is separated from the calling code and repositories. The adaptors/outer layers reference inwards rather than the business logic depending on the repositories.

Pattern that appears under many names

- [Ports and adapters](https://alistair.cockburn.us/hexagonal-architecture/) - 2005, Alistair Cockburn (kou-burn)
  - In this article he actually links to other patterns that are similar again
  - Dependency Injection

- [Onion](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/) - 2008, Jeffrey Palermo
  - Externalize infrastructure and write adapter code so that the infrastructure does not become tightly coupled.

- [Clean Architecure](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) - 2012, Bob Martin

- [Screaming architecture](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html) - 2011, Bob Martin
  - Use case driven architecture
  - Framework is a tool to be used, not architectures to be conformed to

## Traditional approach, problems

![](https://yuml.me/592177fb.svg)

- Database objects throughout the code
- Mocking http calls, db objects
- Unnecessary coupling, coupling of UI to data access layer

## Layered Architecture

Facade hiding implementation details

Module details depend on another module facade
- Abstraction of module should not depend on implementation details

- High cohesion and low coupling
  - Changes are made in same module

### Classic layered architecture

- no cyclic dependencies
- easy to reason about layers responsibilities

If Domain needs dependency on certain behaviour from layer above, use interface and DI.
--> Expose requirements as interface
--> Give me something that implements this contract

## Hexagonal

Infrastructure/domain dependency goes the other way
- Sounds simple but is actually really powerful

Domain/application layer defines ports of the dependencies it requires

- Domain is clean of any technology concerns
- Not subject to change with these
  - Can switch to different web framework, DB
  - OR framework you use to access database
    - possibly more likely than changing DB

## Differences

- Data source is not at center, it is external

## Benefits

- Testing
  - Gives a distinct boundary to write tests against
  - Test at domain boundaries
  - Keeps tests usable and not tightly bound to exact class structure
- Readibility - promotes expressive code, instead of [over abstracting](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
- Pattern that can give commonality across our multiple languages.
  - Looking at a rust service I can see api/domain/adapters
  - Independent of framework
- Independent of external systems
  - Databases
  - Type of queue
  - Request is HTTP or gRPC

# Terminology

Netflix conventions for hexagonal architecture [source](https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749)
- To define business logic
  - Entities - _Domain Objects_
  - Respositores - _Interfaces to get and update entities_
  - Interactors - _Classes to orchestrate and perform domain actions_
- Outside business logic
  - Data sources - _Adapters to different storage implementations_ 
  - Transport layer - _Trigger interactor to perform business logic_

#### Ports and adapters

Another name for hexagonal pattern
- [alistair cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
- https://www.thinktocode.com/2018/07/19/ports-and-adapters-architecture/

- Port - _Boundary between application and adapters_
  - Input port - _Drive application, e.g. Port used by Controllers_
  - Output port - _Driven by application, e.g. Port used to call database_ 
- Adapter - _Converts port definition to logic needed for device_
