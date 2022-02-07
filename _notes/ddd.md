---
title: Domain Driven Design
about: Developer
---

Domain models are abstractions and _models_ to be useful not realistic

Narrow focus on a specific problem

Domain model problem can be described in psuedocode
Can have real discussions with domain experts about problem and what needs to be done
Ubiquitous language ... within a bounded context

Complexity of software is the understanding of the domain, if not careful the software is more complex than domain. Hope to model and make it simpler.

If problem changes and model doesn't fit well
- Might be worth changing the model
- Maybe another place with a different bounded context with a different model
- Bend model to fit new problem (default, if not thinking of the other options)

--> think of model that fits your most difficult problems

## Bounded Contexts

Trying to find ubiquitous language, but within boundaries

Too broad if trying to find language to use for everything

--> wonder if we should define boundaries within the boundaries we have defined, per service?! Should this be outlined in the readme??

Create _many_ ubiquitous languages

One measure of system complexity is how much tracing we would have to do to be able to work out what was going on
For example from `order.Place()` -> what is `order` , `Placce` etc

Highly coupled systems -> need to look at lot of stuff. Maybe think ever will understand what will happen, won't change things

Aim for multiple models in multiple bounded contexts focused on distinct problem sets

Trying to find ownership and who is responsible for maintainance

## Finding bounded context

Looking for clues
- Linguistic boundaries
- Data: flow, ownership, uniqueness
- Domain expert boundaries
  - different experts might care about different things
  - this can give clues about good clean boundaries 
- Business process step

"All models are wrong, some are useful" - George Box

### Justifying bounded contexts

- Customers don't care

Create composite application with context that spans multiple other contexts that holds info relating to that

Why do we do it? Monolothic applications don't scale, trip over each other
-> move faster

Good boundaries improve flow -> Theory of Constraints

--> using bounded contexts to fix and remove bottlenecks

Use DDD to find clues about language for models --> Hypthesis
Use Theory of Contraints to choose correct model

Boundaries must evolve -> new business requirements or insights _challenge_ existing assumptions

Trade collaboration costs with innovation speed to meet current organisational needs

Contexts grow and multiply
- Split contexts as they grow

### Context cohesion

Which contexts/teams need to work together? --> they need to collaborate

What happens when new product spans two contexts?
- Create discovery team with people from each team to work on MVP -> Highly aligned tribe

How to create alignment for developers
- Games
- Show and tells
- Cross-team pairing
- Cross-functional pairing
- Event storming

<opinion>
Bounded contexts are a proxy heuristic for autonomy
- Autonomy contexts
- Owned by single team

To be effective with DDD then organisational model needs to reflect what you want it to look like

- Experiment with models
- Learn Theory of Constraints
- Justify your choices
- Strive for High Alignment

Art of dicovering bounded contexts is the art of aligning organisational and technical boundaries

## Resources

- [What is DDD - Eric Evans - DDD Europe 2019](https://www.youtube.com/watch?v=pMuiVlnGqjk)
- [The Art of Discovering Bounded Contexts by Nick Tune](https://www.youtube.com/watch?v=ez9GWESKG4I)
