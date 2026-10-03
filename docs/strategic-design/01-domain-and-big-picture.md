# 1. The domain and the big picture

Strategic design starts with the business, not with tables or services. Before drawing a single box, we need to know **how the company makes money and where it wins against competitors**, because that decides where we spend our modelling effort.

## The business: Northlane HR

Northlane is a fictional HR platform sold as SaaS to mid-size companies (100 to 3,000 employees) that employ people in several European countries: Germany, the Netherlands, Spain, France and the UK.

Every HR platform stores employee records. Northlane wins deals because of one promise:

> **"Every leave balance is correct and legally compliant in every country you employ people in, without a single spreadsheet."**

Before Northlane, the customers' HR teams spent days each month reconciling holiday balances by hand: part-timers whose hours changed mid-year, Spanish public holidays that differ per region, German carry-over rules, Dutch statutory versus extra-statutory days that expire at different times, sick days during holidays that must be given back. Mistakes cost money and lead to legal disputes. Northlane encodes those rules, so the balance an employee sees is the balance they are legally owed.

That one sentence is the most important input to everything in this module. Keep it in mind.

## Who is involved

| Person | Cares about |
|--------|-------------|
| Employee | How many days off they have, requesting leave quickly, getting paid correctly |
| Line manager | Approving leave without leaving the team short-staffed |
| HR administrator | Accurate employee records, hires, contract changes, leavers |
| HR policy owner | Configuring leave policies that comply with each country's law and the company's own extras |
| Payroll specialist | Receiving every pay-relevant change (salary, unpaid leave, overtime) before payroll closes |
| Recruiter | Handing a new hire over from the recruitment tool |
| IT administrator | Accounts created on day one, removed on the last day |

Each of these people uses some of the same words (employee, manager, day, absence, hours) to mean different things. That is a strong hint about where boundaries will go. We come back to it in the [glossary](05-ubiquitous-language.md).

## Big-picture Event Storming

Event Storming is a workshop where domain experts and developers put **domain events** (things that happened, written in the past tense) on a timeline. It is the fastest way to discover a domain without arguing about data models. Below is the result of a big-picture session, cleaned up. It follows one employee's life at a company.

### Happy path: the employee lifecycle

```
Joining                         Working life                        Changes                            Leaving
───────                         ────────────                        ───────                            ───────
OfferAccepted (recruitment)     LeaveAccrued (monthly)              EmploymentTermsChanged             ResignationReceived
★ EmployeeHired                 LeaveRequested                        (e.g. 5 days → 4 days a week)    ★ EmploymentTerminated
OnboardingPlanStarted           LeaveRequestApproved                EntitlementRecalculated            FinalLeaveBalanceSettled
ContractSigned                  ★ LeaveTaken                        EmployeeTransferred                PayChangesExported
UserAccountProvisioned          TimesheetSubmitted                    (new manager, new team)          UserAccountDeactivated
LeaveAccountsOpened             OvertimeApproved                    ApproverChanged
EntitlementGranted              TimeOffInLieuAccrued                SalaryChanged
                                ★ PayPeriodClosed                   LeaveYearClosed
                                PayChangesExported                  BalanceCarriedOver / BalanceExpired
```

### Unhappy paths (where the interesting rules live)

| Event | What it means | Why it matters |
|-------|---------------|----------------|
| `LeaveRequestRejected` | Manager says no | The balance must not change |
| `ApprovedLeaveCancelled` | Employee cancels holiday they already had approved | Days go back to the balance, and payroll may need a correction |
| `SicknessReportedDuringLeave` | Employee falls ill on holiday | In many EU countries the holiday days must be **given back**. This is a legal rule, not a nice-to-have |
| `PublicHolidayAnnounced` | A government adds a holiday after leave was approved | Approved leave that covers that date now deducts one day less |
| `EmploymentTermsChanged` with a past effective date | HR enters a contract change late ("she went part-time last month") | Entitlement and accruals must be recalculated from that date |
| `BalanceWentNegative` | Employee took more than they had | Allowed in some policies, forbidden in others; settled at termination |
| `ApproverUnavailable` | Manager is on leave themselves | Who approves now? |
| `PayPeriodClosedWithPendingChanges` | A change arrived after payroll closed | Must be carried into the next period, never lost |

### Pivotal events

Some events mark a hand-over where **the language and the people change**. These are called pivotal events (marked ★ above), and they are the best early clue for boundaries:

- **`EmployeeHired`**: before it, recruiters talk about candidates and offers. After it, HR talks about employees, contracts and start dates.
- **`LeaveTaken`**: once the leave dates pass, the request is no longer a plan. It becomes a fact that payroll and time tracking care about.
- **`PayPeriodClosed`**: after it, changes for that month cannot go into that month's pay.
- **`EmploymentTerminated`**: the end of the employment triggers settlements in several areas, each with its own rules.

### Hotspots (open questions raised during the session)

Event Storming marks disagreements and unknowns as hotspots. These were the important ones, and the answers shaped the design:

1. **"When someone goes part-time, who decides what happens to their holiday?"** HR records the contract change, but only the leave policy knows how to pro-rate entitlement. Answer: Employee Records publishes the change, Leave recalculates. See [decisions](06-design-decisions.md#d6-when-a-rule-needs-another-contexts-data-let-the-owner-decide).
2. **"Is a 'day' of leave eight hours?"** Policy owners, payroll and time tracking all said yes and all meant different things. See the glossary.
3. **"Who is the approver?"** The line manager in the org chart, except when they are absent, or when the policy says HR approves long leave. Answer: Leave owns the approval rules and keeps its own copy of the reporting line.
4. **"Can we change a balance by editing a number?"** HR administrators asked for it. Employment lawyers said every change must be explainable in a dispute. Answer: balances are a ledger. See [D5](06-design-decisions.md#d5-leave-balances-are-a-ledger-not-a-number).

Next: [classify the subdomains](02-subdomains.md).
