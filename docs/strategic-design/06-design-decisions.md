# 6. Design decisions and trade-offs

Strategic design is a series of decisions with trade-offs. Writing them down (in the style of lightweight Architecture Decision Records) lets future you, and future teammates, know *why* the system looks the way it does, and *when* to change it.

Each decision lists the choice, the reasoning, what it costs, and the signal that should make us revisit it.

---

### D1. Leave is the core domain, not Employee Records

**Decision:** Invest the deepest modelling and best people in Leave & Absence Management.

**Why:** Northlane wins deals on correct, compliant leave balances across countries. Employee records are expected, not differentiating.

**Cost:** Employee Records still needs to be accurate, and it is upstream of everything, so it cannot be neglected. It just gets a simpler model.

**Revisit when:** The business strategy changes, for example if Northlane starts selling working-time compliance or its own payroll.

---

### D2. Organisation structure lives inside Employee Records for now

**Decision:** Organisation Structure is a separate subdomain but is implemented in the Employee Records context.

**Why:** Same experts (HR administrators), same team, simple rules. A separate context would add an integration with no benefit.

**Cost:** If organisation logic grows, Employee Records becomes a mixed model.

**Revisit when:** Workforce planning appears: budgeted positions, headcount approvals, org scenarios.

---

### D3. Leave keeps its own copy of the employment facts it needs

**Decision:** Leave stores a local, effective-dated copy of each employee's country and region, working pattern, approver and employment dates, built from Employee Records' events. It never queries Employee Records during a calculation.

**Why:** Calculations need the facts *as they were on specific dates*, in Leave's own shape. Leave keeps working when Employee Records is down. Leave never needs, and so never stores, personal details.

**Cost:** The copy can be a few seconds behind. Events must be processed in order per employee, and replaying them must be safe (idempotent).

**Revisit when:** Never in principle. The shape of the copy will grow as Leave needs new facts.

---

### D4. Everything about employment is effective-dated

**Decision:** Employment terms, reporting lines and working patterns are stored as a history of values with effective dates, never overwritten. Events carry the effective date.

**Why:** HR changes are routinely entered late ("she went part-time on the 1st, we're entering it on the 20th") or early ("his raise starts in January"). Leave, Payroll Integration and audits all need to know what was true on a given date.

**Cost:** Every query must say "as of when". Effective dating makes the model and the database harder; it is the main source of complexity in Employee Records.

**Revisit when:** Never. This is a property of the domain, not a technical choice.

---

### D5. Leave balances are a ledger, not a number

**Decision:** A `LeaveAccount` holds an append-only list of ledger entries (grant, accrual, deduction, restoration, adjustment, carry-over, expiry, payout). The balance is calculated from them. There is no editable balance field.

**Why:** In an employment dispute, the company must explain every day. A ledger answers "why is my balance 7.5?" by listing the entries. It also makes recalculation safe: correcting a past mistake is a new entry, not a silent overwrite. Accountants have done it this way for 500 years.

**Cost:** More rows, and the balance must be computed (or cached in a projection). HR administrators cannot "just fix the number"; they make an adjustment with a reason.

**Revisit when:** Never for the source of truth. Add read-side caching if performance requires it.

---

### D6. When a rule needs another context's data, let the owner decide

**Decision:** When an employee goes part-time, Employee Records only publishes `EmploymentTermsChanged`. Leave decides how to pro-rate the entitlement. When an employee leaves, Employee Records publishes `EmploymentTerminated`; Leave decides the final settlement and publishes `FinalLeaveBalanceSettled`; Payroll Integration only forwards the result.

**Why:** Pro-rating and settlement rules belong to the leave policy, and only Leave knows them. If Employee Records or Payroll Integration calculated leave payouts, they would need a copy of the policy rules, and the copies would drift apart.

**Cost:** Some outcomes take a few seconds and several events to appear. The HR administrator who records a termination sees the leave settlement shortly afterwards, not instantly.

> **General lesson:** when a business rule needs data from another context, do not copy the rule. Send the facts to the context that owns the rule, and let it decide.

---

### D7. Leave policies are versioned

**Decision:** A policy change creates a new version with an effective date. A request is evaluated against the version in force for its dates. Approved leave is not silently recalculated when a policy changes; a recalculation is an explicit action that produces ledger entries.

**Why:** Customers change policies mid-year ("from July we give one extra day for long service"). Employees must be able to trust that approved leave stays approved.

**Cost:** Policy storage and evaluation must handle versions.

---

### D8. Anti-corruption layers around vendors; conform to standards

**Decision:** The applicant tracking system, the holiday data feed and every payroll provider are accessed through an ACL. The identity provider is used as a Conformist through OIDC and SCIM.

**Why:** ATS and payroll models are foreign and vary per vendor; the holiday feed feeds the core directly. OIDC and SCIM are clean industry standards that never touch the core.

**Cost:** One translation adapter per vendor, with tests.

**Revisit when:** A customer needs a new vendor (add an adapter; this is the point of the ACL).

---

### D9. Personal data lives in one place

**Decision:** Only Employee Records stores personal data (name, address, date of birth, national ID, bank details, salary). Every other context stores an `EmployeeId` and the minimum non-personal facts it needs. Events never carry more personal data than their consumers need.

**Why:** GDPR requires data minimisation, strict access control, and erasure on request. With personal data in one context, an erasure request is handled in one place, and a bug in Time Tracking cannot leak home addresses.

**Cost:** Screens that show names next to leave requests must combine data from two contexts (a read model built for the UI).

**Revisit when:** Never. If a new context asks for personal data, the first question is whether it really needs it.

---

### D10. No Shared Kernel; share only published event schemas

**Decision:** No shared domain library between contexts, not even for `DateRange` or `EmployeeId`. Contexts share versioned event schemas (the Published Language) and nothing else.

**Why:** Shared code couples release cycles and silently spreads one context's rules into others.

**Cost:** Small value objects are written more than once. They are tiny, and each context's version can evolve with its own needs.

---

### D11. Contexts are eventually consistent with each other

**Decision:** A contract change is committed in Employee Records. Leave recalculates entitlements afterwards, when it processes the event. There is no distributed transaction.

**Why:** A transaction across contexts would couple their databases and their availability. A few seconds of delay is acceptable to the business.

**Cost:** An employee could request leave in the seconds between a contract change and its recalculation. Leave handles this: on recalculation it re-checks pending requests and flags any that no longer fit the balance. **This failure mode is a business decision, not a bug**, and policy owners agreed to it.

---

### D12. Start as a modular monolith

**Decision:** One Spring Boot application. One top-level package and one database schema per bounded context. Every aggregate belongs to exactly one customer (tenant), and every table carries the tenant ID. ArchUnit tests enforce that contexts only talk through their public API or events.

**Why:** Getting boundaries right matters far more than deploying them separately. A modular monolith lets us move a boundary with a refactoring instead of a migration project.

**Cost:** Discipline must be enforced by tests, because the compiler alone will happily let one package call another.

**Revisit when:** A context needs independent scaling, a different release cadence, or a different technology. Payroll Integration (heavy, bursty exports at month end) is the most likely first candidate.

Next: [exercises and a checklist for your own systems](07-exercises-and-checklist.md).
