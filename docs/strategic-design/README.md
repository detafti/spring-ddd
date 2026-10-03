# Module 1: Strategic design workshop

Strategic design decides **what to build, where to draw boundaries, and where to invest**, before any class is written. Most failed "DDD projects" failed here, not in their aggregates.

This module designs **Northlane HR**, a fictional HR platform for mid-size companies with staff in several European countries. Its competitive edge is leave balances that are always correct under each country's law. Later modules build its bounded contexts in Spring Boot, so treat this as the blueprint.

## Read in order

1. [The domain and the big picture](01-domain-and-big-picture.md): the business, the people, an Event Storming timeline of the employee lifecycle, pivotal events and hotspots
2. [Subdomains](02-subdomains.md): core, supporting and generic, and why Employee Records is *not* core even though everything depends on it
3. [Bounded contexts](03-bounded-contexts.md): how the boundaries were found, one card per context, rejected alternatives
4. [Context map](04-context-map.md): every relationship with its pattern and reasoning, plus a decision guide
5. [Ubiquitous language](05-ubiquitous-language.md): one glossary per context and the words that mean different things
6. [Design decisions](06-design-decisions.md): twelve decisions, their costs, and when to revisit them
7. [Exercises and checklist](07-exercises-and-checklist.md): practice questions and a checklist for systems at work

## The result on one page

| Context | Type | Team | Key relationship |
|---------|------|------|------------------|
| **Leave** | **Core** | Leave | Customer of Employee Records; Partnership with Time Tracking; ACL to the holiday feed |
| Employee Records | Supporting | Core HR | Supplier to everyone (Open Host Service + events); ACL to the recruitment tool |
| Onboarding | Supporting | Core HR | Conformist to Employee Records' events |
| Time Tracking | Supporting | Time & Pay | Partnership with Leave |
| Payroll Integration | Generic (thin) | Time & Pay | Conformist to everyone's events; ACL per payroll provider |
| Notifications | Generic (thin) | Platform | Conformist to everyone's events |

## Eight ideas to take to work

1. **The core domain comes from business strategy.** Ask "what would hurt most if it were mediocre?", not "what looks most central?"
2. **"Everything depends on it" does not make something core.** Employee Records is upstream of everything and still supporting.
3. **Complexity does not make something core.** Payroll is the most complex area of HR and generic for Northlane. Buy it.
4. **Boundaries follow language.** When "day" or "manager" changes meaning, you are crossing a boundary. Pivotal events show you where.
5. **Upstream/downstream is about whose model wins, not which way data flows.** Northlane sends data to payroll providers, and they are upstream.
6. **When a rule needs another context's data, let the owner decide.** Leave decides how a contract change affects holiday, not Employee Records.
7. **Keep personal data in one context.** Data protection is a boundary heuristic, and a strong one in HR.
8. **A bounded context is a model boundary, not a microservice.** Start as a modular monolith and enforce the boundaries with tests.

## What comes next

- **Module 2** builds the **Leave** context in Spring Boot with Clean Architecture and ArchUnit: leave policies, leave accounts with a ledger, leave requests, approval, working patterns and public holidays. It is the core, so it is where tactical DDD pays off most.
- **Module 3** adds **Employee Records** and connects it to Leave with `EmployeeHired` and `EmploymentTermsChanged` events, effective dating, the outbox pattern, and an ACL to a fake payroll provider.
