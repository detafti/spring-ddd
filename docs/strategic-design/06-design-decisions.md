# 6. Design decisions and trade-offs

Strategic design is a series of decisions with trade-offs. Writing them down (in the style of lightweight Architecture Decision Records) lets future you, and future teammates, know *why* the system looks the way it does, and *when* to change it.

Each decision lists the choice, the reasoning, what it costs, and the signal that should make us revisit it.

---

### D1. Fulfilment is the core domain, not Ordering

**Decision:** Invest the deepest modelling and best people in Order Promising & Fulfilment.

**Why:** Kestrel wins on the reliability of its delivery promise. Checkout is ordinary.

**Cost:** The most complex model is also the one with the most integrations (Inventory, WMS, Shipping, Ordering).

**Revisit when:** The business strategy changes, for example if Kestrel starts competing on price or on a marketplace model.

---

### D2. Pricing lives inside the Catalog context for now

**Decision:** Pricing & Promotions is a separate subdomain but is implemented in the Catalog bounded context.

**Why:** Simple rules, same team, same experts (merchandisers). A separate context would add an integration with no benefit.

**Cost:** If pricing logic grows, Catalog becomes a mixed model.

**Revisit when:** Pricing gets its own experts or its own rules engine (dynamic pricing, customer-specific prices, B2B contracts).

---

### D3. Order lines copy the product name and agreed price

**Decision:** At checkout, Ordering copies SKU, name and unit price from Catalog into each `OrderLine`. It never reads Catalog again for that order.

**Why:** An order is a record of an agreement. Catalog's price is "today's price"; the order's price is "the price we agreed". These are different facts that happen to look alike.

**Cost:** Some data appears in two places. That is fine, because it is not the same data.

**Revisit when:** Never, for prices. For descriptive fields (images, names), a downstream might choose to show current data instead; that is a UI decision.

---

### D4. Ordering and Fulfilment are a Partnership with a tiny contract

**Decision:** The promise is a synchronous query at checkout. The hand-over after checkout is asynchronous events (`OrderPlaced` one way, `ShipmentDispatched` and friends the other way).

**Why:** The shopper must see the date before paying, so the promise has to be synchronous. Everything after placing the order can tolerate seconds of delay, and asynchronous events stop a Fulfilment outage from blocking checkout.

**Cost:** Two teams must coordinate releases. A synchronous call means checkout depends on Fulfilment being up. Mitigation: if the Promise API is down, checkout falls back to a conservative default promise (+3 days) rather than failing.

**Revisit when:** The teams stop coordinating smoothly, in which case make Fulfilment a clear upstream with an Open Host Service.

---

### D5. Anti-corruption layers around every vendor, except identity

**Decision:** WMS, payment provider and carrier aggregator are always accessed through an ACL. The identity provider is used as a Conformist.

**Why:** The WMS especially has a model (waves, LPNs, numeric status codes) that would pollute the core. The identity provider's model is small and clean, and we only need an ID from it.

**Cost:** Translation code to write and test.

**Revisit when:** We start needing richer customer data from the identity provider (then add an ACL), or we replace a vendor (the ACL is where the work happens, which is the point).

---

### D6. Cancellation is a request, not a command

**Decision:** Ordering records `CancellationRequested`. Fulfilment decides and replies with `CancellationAccepted` or `CancellationRejected`. Only then does the order become `CANCELLED`.

**Why:** The rule "you cannot cancel once the shipment has been released to the warehouse" belongs to Fulfilment, because only Fulfilment knows the release state. If Ordering cancelled directly, it would need a copy of Fulfilment's rules and state, and the two would drift apart.

**Cost:** The shopper does not get an instant final answer. The UI says "cancellation requested" and confirms a few seconds later.

**Revisit when:** The business wants instant cancellation, in which case Ordering could query Fulfilment synchronously first, but Fulfilment would still own the decision.

> **General lesson:** when a business rule needs data from another context, do not copy the rule. Send a request to the context that owns the data, and let it decide.

---

### D7. No Shared Kernel; share only published event schemas

**Decision:** No shared domain library between contexts, not even for `Money` or `Address`. Contexts share versioned event schemas (the Published Language) and nothing else.

**Why:** Shared code couples release cycles and silently spreads one context's rules into others.

**Cost:** Small value objects like `Money` are written more than once. They are tiny, and each context's version can evolve with its own needs.

**Revisit when:** Two contexts are owned by one team and genuinely need identical, very stable logic.

---

### D8. Contexts reference each other only by ID

**Decision:** No context holds an object from another context, only identifiers. Fulfilment's `Fulfilment` stores the `OrderId` it was created for; Ordering's `Order` stores nothing from Fulfilment at all, because events carry the `OrderId` back.

**Why:** Object references across contexts would let one context load and change another's data, which destroys the boundary.

**Cost:** Some views (for example the customer service screen showing an order and its shipments) need to combine data from two contexts. We build those as read-only projections fed by events.

---

### D9. Order placement and stock reservation are eventually consistent

**Decision:** `OrderPlaced` is committed in Ordering's transaction. Stock is reserved afterwards, when Fulfilment plans the order. There is no distributed transaction.

**Why:** A single transaction across contexts would couple their databases and availability. The business accepts a gap of a few seconds.

**Cost:** In rare cases, the last unit is sold twice within that gap. Fulfilment detects it during planning, re-plans from the other warehouse, or back-orders. Safety stock makes this rare. **This failure mode is a business decision, not a bug**, and the business agreed to it.

**Revisit when:** Oversells become frequent (for example, during flash sales). Then consider a short soft-hold on stock at promise-quote time.

---

### D10. Start as a modular monolith

**Decision:** One Spring Boot application. One top-level package and one database schema per bounded context. ArchUnit tests enforce that contexts only talk through their public API or events.

**Why:** Getting boundaries right matters far more than deploying them separately. A modular monolith lets us move a boundary with a refactoring instead of a migration project. Kestrel's scale (thousands of orders per day) does not need independent scaling.

**Cost:** Discipline must be enforced by tests, because the compiler alone will happily let one package call another.

**Revisit when:** A context needs independent scaling, a different release cadence, or a different technology. Fulfilment (heavy re-planning computations at cut-off time) is the most likely first candidate.

Next: [exercises and a checklist for your own systems](07-exercises-and-checklist.md).
