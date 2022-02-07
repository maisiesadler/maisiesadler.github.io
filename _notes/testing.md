---
title: Testing, mocking
layout: note
tags: testing mocking
---

Test pyramid has 3 levels

1. Unit Tests
1. Service Tests
1. User Interface Tests

Top of pyramid has more integration, bottom has more isolation.
Top tests take longer to run, bottom are much faster.

Even if the test types are different from those three the essence is the same:
1. Write tests with different granularity
1. The more high-level you get the fewer tests you should have

Shorten feedback loops by automating tests, moving more scenarios down the testing pyramid. Less manual testing.

Test each thing as far down the pyramid as you can.

## Types of tests

Some definitions

### Unit tests

- Test against public interface of module

### Integration

E.g. HTTP client tested against an API

- Narrow integration with external system mocks

### Integration, again?

- Whole module with external system mocks

### ...Integration...?

Or maybe component tests?

- Service started in memory, external system mocks

### Contract tests

- Matches integration testing definition, but producer and consumer agree on contract and test against that
- Automated contract tests ensure that the consumer and provider stick to defined contract.

### Consumer Driven Contract tests

Let consumers drive the implementation of a contract.

- Allows the providing team to implement only what is necessary
- Fetch and run tests continuously
- Ensure they don't break anything

CDC tests are a step towards having autonomous teams.

### End-to-End/Acceptance testing

- Deployed service, running against real dependencies
- Dev, staging or prod
- Black box - does it work from the users perspective

#### Pros

Integration tests alone can be brittle, how can we be sure that the fake service is behaving correctly
(What if the real service actually returns and 200 with {"error": "..."})

### Manual tests

- Ad hoc manual tests

Further up the chain you need more stuff to be set up
- Whatever you're testing for you should try and move as far down the pyramid as you can
  - Faster, cheaper to run
  - Faster feedback loop

Moving error finding into the lower testing sections to get quickest feedback.

> Narrow integration tests at the boundaries (adapters?) are faster and easier to reason about. (component tests?)

## Test doubles

Test double
- Replace real thing with a fake version of that thing
- Fake thing answers with canned responses you define yourself at the beginning of the test

Stubs = return canned data
Mock = Assert call made with correct params
Spy = Return real value but listen to calls

Unit tests run fast, mock or stub db calls, file system calls and http queries.

Overmocking can lead to tests being tied to structure/implementation of code rather than behaviour.

### Mocking and the need to use actual data format

- Using mock db
- Using external running API
  - Cannot assert against the calls
  - Stub not mock
  - Can't add conditionals to responses
  - Can add different responses but there is a disconnect between mock response and test

#### WireMock

Nice interface for setting up conditional replies
- Similar to mocking frameworks
- Sets up server on localhost

-> Not good for load testing

#### Using JSON vs using C# objects

- Serialization

-> Tests in here for successful, error requests

## What to test

- Test for observable behaviour
- Don't tie tests too closely to implementation
- Private methods are an implementation detail, shouldn't need to test

## Other nice test properties

Decoupling
- tests should run independently of external systems
  - other system maintainance

Clean test code

- Test code is as important as Production code.
- Test one condition per test.
- Arrange, Act, Assert.
- Readability > DRY
- DAMP over DRY code [src](https://stackoverflow.com/questions/6453235/what-does-damp-not-dry-mean-when-talking-about-unit-tests)

## Test duplication

Avoid having test cases at multiple levels - there's such a thing as too many tests.

- Rule = push test as far down the test pyramid as you can
- Lower down = less bloated, faster

If a higher level test gives you more confidence that your application works correctly, keep it.

Controller test won't give you confidence that service will respond to HTTP request - move test up pyramid to test just that.
- Don't need to test all conditional logic that is already covered by lower level tests
- Higher level test focus only on parts that lower level tests couldn't cover

Even if the test was hard to write, be aware of [sunk cost](https://en.wikipedia.org/wiki/Sunk_cost) fallacy and delete test.

## Testing modules

- Internal by default
- Test public interface BEHAVIOUR

## WebApplicationFactory

## Test structure

- arrange, act, assert
- readable test setup, without too much indirection

## Resources

- https://martinfowler.com/articles/practical-test-pyramid.html
- [TDD](https://www.amazon.co.uk/Test-Driven-Development-Addison-Wesley-Signature/dp/0321146530)
