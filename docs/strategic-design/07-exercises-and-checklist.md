# 7. Exercises and a checklist for your own systems

Reading strategic design is easy; doing it is not. These exercises force the decisions. Write your answers down before reading the hints.

## Exercises on Kestrel Supply

**E1. Strategy shift.** Kestrel's board decides to open a marketplace where third-party sellers ship their own products. Which subdomains change type? Is there a new core domain?

<details><summary>Hint</summary>

Seller onboarding and seller performance (are sellers keeping *Kestrel's* promise?) may become core, because the brand promise now depends on people outside the company. Fulfilment's sourcing now includes "warehouse" = seller, so its model has to generalise. Payments gets more complex (split payouts) but is still generic: marketplace payment products exist.
</details>

**E2. A new word.** Warehouse staff start talking about "priority orders" that get picked first. Which context does this term belong to? Does Ordering need to know about it?

<details><summary>Hint</summary>

Picking order is the WMS's business, but *deciding* that a shipment is a priority (because its promise is at risk) is a Fulfilment decision. Fulfilment sends a priority flag through its ACL; the WMS uses its own concept for it. Ordering does not need to know at all.
</details>

**E3. The tempting shortcut.** A developer suggests Ordering should read Inventory's table directly to show "only 2 left!" on the product page, because it is faster. What is wrong, and what would you do instead?

<details><summary>Hint</summary>

It couples Ordering to Inventory's schema, bypasses Inventory's definition of available-to-promise, and breaks the "contexts talk through APIs or events" rule (D10). Options: Inventory's availability API (OHS), or a read model in Catalog/Ordering updated from Inventory events. Also ask the business which number they actually want to show: on hand or ATP?
</details>

**E4. Draw the Returns process.** Customers can return items within 30 days. Run a mini Event Storming on paper: what are the events? Is Returns a new context, part of Fulfilment, or part of Ordering? What are the relationships?

**E5. Challenge a decision.** Pick one decision from [06-design-decisions.md](06-design-decisions.md) and argue for the opposite choice. Under what business conditions would the opposite be correct?

## Checklist: strategic design for a system at work

Use this on a real system (module 4 does exactly this). It works for new systems and for untangling existing ones.

### Understand the business

- [ ] Can you state in one sentence why customers choose this company or product over competitors?
- [ ] Have you listed the people (roles) involved and what each cares about?
- [ ] Have you run at least a small Event Storming session, with domain experts in the room, covering the happy path *and* the unhappy paths?
- [ ] Have you marked the pivotal events and the hotspots?

### Subdomains

- [ ] Is every subdomain classified as core, supporting or generic, with a written reason?
- [ ] Is the core domain small? (If "everything is core", nothing is.)
- [ ] For each generic subdomain, have you checked whether you can buy it instead of building it?
- [ ] Do your best developers work on the core?

### Bounded contexts

- [ ] Inside each context, does every important term have exactly one meaning?
- [ ] Does each context have a "does not own" list?
- [ ] Is each context owned by exactly one team?
- [ ] Is there a "God entity" (an `Order`, `Customer` or `Product` class with fields from many teams)? If yes, that is your first split.
- [ ] Did you write down the alternatives you rejected, and why?

### Context map

- [ ] Is every relationship labelled with upstream/downstream and a pattern?
- [ ] Is the core domain protected by an ACL from every vendor or legacy system?
- [ ] Is every Conformist relationship a conscious decision, not an accident?
- [ ] Is there any Shared Kernel? If so, is it tiny, stable and jointly owned?
- [ ] Are there two-way dependencies? Are they explicit Partnerships with a small contract?
- [ ] For each business rule that needs data from two contexts, is it clear which context decides?

### Language

- [ ] Is there a glossary per context, used in code (class, method, event and test names)?
- [ ] Do you have a table of words that mean different things in different contexts?
- [ ] Are vendor or legacy terms kept inside ACLs?

### Red flags in an existing system

| You see | It usually means |
|---------|-----------------|
| A table or class that every team edits | Missing boundary, God entity |
| `status` columns with 15+ values | Several lifecycles from several contexts squeezed into one |
| A "common" or "shared" module that keeps growing | Accidental Shared Kernel |
| Service A reads service B's database | No real boundary, regardless of deployment |
| Microservices that must always be deployed together | Boundaries in the wrong place (distributed monolith) |
| The same business rule implemented in two places | Unclear ownership of a decision |
| Vendor field names in your domain classes | Missing ACL |

Back to the [overview](README.md).
