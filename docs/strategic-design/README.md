# Module 1: Strategic design workshop

Strategic design decides **what to build, where to draw boundaries, and where to invest**, before any class is written. Most failed "DDD projects" failed here, not in their aggregates.

This module designs **Kestrel Supply**, an online home goods retailer whose competitive edge is a delivery date it actually keeps. Later modules build its bounded contexts in Spring Boot, so treat this as the blueprint.

## Read in order

1. [The domain and the big picture](01-domain-and-big-picture.md): the business, the people, an Event Storming timeline, pivotal events and hotspots
2. [Subdomains](02-subdomains.md): core, supporting and generic, and why Ordering is *not* core here
3. [Bounded contexts](03-bounded-contexts.md): how the boundaries were found, one card per context, rejected alternatives
4. [Context map](04-context-map.md): every relationship with its pattern and reasoning, plus a decision guide
5. [Ubiquitous language](05-ubiquitous-language.md): one glossary per context and the words that mean different things
6. [Design decisions](06-design-decisions.md): ten decisions, their costs, and when to revisit them
7. [Exercises and checklist](07-exercises-and-checklist.md): practice questions and a checklist for systems at work

## The result on one page

| Context | Type | Team | Key relationship |
|---------|------|------|------------------|
| **Fulfilment** | **Core** | Fulfilment | Partnership with Ordering; ACL to the WMS |
| Inventory | Supporting | Fulfilment | Supplier to Fulfilment |
| Ordering | Supporting | Storefront | Partnership with Fulfilment; customer of Catalog and Payments |
| Catalog | Supporting | Storefront | Open Host Service for product and price data |
| Payments | Generic (thin) | Platform | ACL over the payment provider |
| Shipping | Generic (thin) | Platform | ACL over the carrier aggregator |
| Notifications | Generic (thin) | Platform | Conformist to everyone's events |

## Seven ideas to take to work

1. **The core domain comes from business strategy.** Ask "what would hurt most if it were mediocre?", not "what looks most central?"
2. **Complexity does not make something core.** Warehousing and payments are complex and generic. Buy them.
3. **Boundaries follow language.** When a word changes meaning, you are crossing a boundary. Pivotal events show you where.
4. **A bounded context is a model boundary, not a microservice.** Start as a modular monolith and enforce the boundaries with tests.
5. **Upstream/downstream is about whose model wins, not which way data flows.**
6. **Protect the core with anti-corruption layers; be a Conformist on purpose elsewhere.** Both are valid; accidental conformity is the problem.
7. **When a rule needs another context's data, ask that context to decide.** Do not copy the rule.

## What comes next

- **Module 2** builds the **Ordering** context in Spring Boot with Clean Architecture and ArchUnit. Ordering is not the core, but its rules (an order and its lines, money, a price snapshot, placing and cancelling) are the clearest way to learn aggregates, value objects and domain events.
- **Module 3** adds **Fulfilment**, the core, and connects it to Ordering with `OrderPlaced` and `ShipmentDispatched`, an ACL to a fake WMS, and the outbox pattern.
