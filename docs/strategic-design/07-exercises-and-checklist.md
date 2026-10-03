# 7. Exercises and a checklist for your own systems

Reading strategic design is easy; doing it is not. These exercises force the decisions. Write your answers down before reading the hints.

## Exercises on Northlane HR

**E1. Strategy shift.** Northlane's board decides to run payroll itself in the Netherlands instead of sending changes to providers. Which subdomains change type? What happens to the Payroll Integration context?

<details><summary>Hint</summary>

Dutch payroll becomes a new subdomain Northlane builds. Is it core? Only if Northlane plans to win deals with it; if it is just to reduce costs, it is supporting, and buying a payroll engine may still be smarter. Payroll Integration keeps existing for the other countries, and a new Payroll context appears for the Netherlands, downstream of the same events. Notice that the published events did not have to change: that is the payoff of a good Published Language.
</details>

**E2. A new word.** Policy owners start talking about "blackout periods": dates when nobody in a team may take leave (for example, year-end closing in Finance). Which context owns this concept? Does Employee Records need to know about it?

<details><summary>Hint</summary>

It is an approval rule, so it belongs to Leave, probably as part of the policy or the approval route. Leave needs to know which team someone is in, which it already has from its copy of the reporting line or org unit. Employee Records does not need to know blackout periods exist.
</details>

**E3. The tempting shortcut.** A developer suggests that Payroll Integration should read Leave's ledger table directly to calculate the leave payout at termination, "because it's just a sum". What is wrong, and what would you do instead?

<details><summary>Hint</summary>

It couples Payroll Integration to Leave's schema, and it copies a rule (how to value unused leave, which depends on policy and country) into the wrong context. See [D6](06-design-decisions.md#d6-when-a-rule-needs-another-contexts-data-let-the-owner-decide): Leave settles and publishes `FinalLeaveBalanceSettled` with the result; Payroll Integration forwards it.
</details>

**E4. Run a mini Event Storming on sickness.** Employees report sick, sometimes with a doctor's note, sometimes for weeks, sometimes during approved leave. Some countries pay sick days differently after a certain number of days. What are the events? Is sickness part of Leave, or a new context? What are its relationships, and how do you handle the fact that sickness details are sensitive health data?

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
- [ ] Is there a "God entity" (an `Employee`, `Order` or `Customer` class with fields from many teams)? If yes, that is your first split.
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
