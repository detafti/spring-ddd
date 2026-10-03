# 5. Ubiquitous language

The **ubiquitous language** is the vocabulary that domain experts and developers share *inside one bounded context*. It is used in conversations, in documents, and **in the code**: class names, method names, event names and test names. If the policy owner says "the request is approved and the days are deducted from the balance", the code says `leaveRequest.approve()` and the ledger gets a `Deduction` entry, not `request.setStatus(2)` and `balance = balance - x`.

"Ubiquitous" means *everywhere within the context*, not *everywhere in the company*. Each context has its own glossary.

## The same word, different meanings

This table is the main reason Northlane has several bounded contexts instead of one. Each row was a source of confusion in the Event Storming session.

| Word people use | Employee Records | Leave | Time Tracking | Payroll Integration | Recruitment (ATS) |
|-----------------|------------------|-------|---------------|---------------------|-------------------|
| **Employee** | `Employee`: a person with personal details and one or more employments | Only an `EmployeeId` plus a local `WorkingPattern`, region and approver. No name, no address. | `EmployeeId` on a `Timesheet` | Provider's *worker* or *payee* number, mapped from `EmployeeId` | *Candidate* (becomes an employee only after hire) |
| **Manager** | The person above you in the `ReportingLine` | `Approver`: usually the line manager, but can be a delegate or HR, depending on policy | Timesheet approver | *(not used)* | *Hiring manager* (a different role) |
| **Day** | A calendar date (start date, effective date) | One **working day** according to the employee's working pattern. A part-timer working Mon/Tue/Wed taking a full week off uses 3 days. | A calendar date with hours recorded on it | A paid day in a pay period | *(not used)* |
| **Hours** | Contracted hours per week | Hours of leave, for policies that count in hours | Hours actually worked | Hours to be paid | *(not used)* |
| **Absence** | *(not used)* | Approved leave or sickness on specific dates | A working day with no time recorded | Unpaid absence that reduces pay | *(not used)* |
| **Balance** | *(not used)* | The sum of a `LeaveAccount`'s ledger entries | *(not used)* | *(not used)*, except the payout amount at termination | *(not used)* |
| **Terminate** | End an employment on a date | Settle the final balance: pay out or deduct | Close future timesheets | Send a final pay change | *(not used)* |

> Note the cells marked *(not used)*. Refusing to use a word inside a context is as important as defining it. Leave deliberately never stores a name or calls anyone "employee record"; it only needs an ID and a working pattern. That is also what keeps personal data in one place.

## Glossary per context

### Leave (core)

| Term | Definition | Code |
|------|------------|------|
| Leave type | A kind of leave with its own rules: annual leave, sick leave, parental leave, time off in lieu, unpaid leave. | `LeaveType` |
| Leave policy | The rules for one leave type in one country for one employer: entitlement, accrual, carry-over, expiry, approval, minimum balance. Combines statutory rules and company extras. Versioned. | `LeavePolicy` (aggregate root) |
| Statutory leave | The minimum the law requires (for example 20 days in Germany for a 5-day week). Cannot be reduced by company policy. | `StatutoryEntitlement` |
| Extra-statutory leave | Leave the employer gives on top of the legal minimum. May follow different expiry rules. | `ExtraStatutoryEntitlement` |
| Leave year | The 12-month period a policy's entitlement applies to. Usually the calendar year, not always. | `LeaveYear` (value object) |
| Entitlement | The amount of leave an employee is owed for a leave year under a policy, after pro-rating. | `Entitlement` (value object) |
| Pro-rating | Reducing an entitlement for someone who works part-time or joins or leaves mid-year. | `ProRating` (domain service) |
| Accrual | Entitlement building up over time (for example 2.08 days per month) instead of all at once. | `Accrual`, event `LeaveAccrued` |
| Leave account | One employee's account for one leave type and leave year. Its balance is the sum of its ledger. | `LeaveAccount` (aggregate root) |
| Ledger entry | One immutable movement in a leave account: grant, accrual, deduction, restoration, adjustment, carry-over, expiry, payout. | `LedgerEntry` (value object) |
| Balance | What is left in a leave account: the sum of its ledger entries. Never stored as a number that can be edited. | `LeaveAccount.balance()` |
| Working pattern | Which days of the week, and how many hours, the employee works. A local copy of the contract terms, effective-dated. | `WorkingPattern` (value object) |
| Leave request | An employee's request for leave on specific dates. | `LeaveRequest` (aggregate root) |
| Deduction | The number of working days (or hours) a request uses, after excluding non-working days and public holidays. | `Deduction` (value object) |
| Approver | Whoever must approve a request under the policy: line manager, delegate, or HR. | `Approver`, `ApprovalRoute` |
| Approve / reject | The approver's decision on a request. | `LeaveRequest.approve()`, `.reject(reason)`, events `LeaveRequestApproved`, `LeaveRequestRejected` |
| Restoration | Giving leave days back, for example after sickness during leave. | `LedgerEntry.restoration(...)` |
| Carry-over | Moving unused balance into the next leave year, within the policy's limit. | event `BalanceCarriedOver` |
| Expiry | Unused balance that is lost at a deadline set by the policy or the law. | event `BalanceExpired` |
| Time off in lieu (TOIL) | Leave earned by working overtime. | event `TimeOffInLieuAccrued` |
| Final settlement | When someone leaves: paying out unused leave or deducting overdrawn leave. | event `FinalLeaveBalanceSettled` |

### Employee Records

| Term | Definition | Code |
|------|------------|------|
| Employee | A person who has, or had, an employment with the customer. | `Employee` (aggregate root) |
| Employment | One contract between the employee and one legal entity, from a start date to (eventually) an end date. | `Employment` (entity) |
| Employment terms | The contract facts at a point in time: job, country and region, contracted hours, working pattern, salary. | `EmploymentTerms` (value object) |
| Effective date | The date from which a change applies, which may be in the past or the future. | `EffectiveDate` |
| Hire | Create an employee and their first employment. | event `EmployeeHired` |
| Change terms | Record new employment terms from an effective date. | event `EmploymentTermsChanged` |
| Transfer | Move an employee to a different org unit, position or manager. | event `EmployeeTransferred` |
| Terminate | End an employment on a date, with a reason. | event `EmploymentTerminated` |
| Org unit | A department or team in the organisation. | `OrgUnit` |
| Position | A seat in the organisation that a person fills. | `Position` |
| Reporting line | Who an employee reports to. | `ReportingLine` |

### Time Tracking

| Term | Definition | Code |
|------|------------|------|
| Timesheet | One employee's recorded hours for one week. | `Timesheet` (aggregate root) |
| Time entry | Hours worked on one date. | `TimeEntry` |
| Overtime | Hours worked beyond the contracted hours, which need approval. | `Overtime`, event `OvertimeApproved` |
| Submit / approve | The employee submits a timesheet; the team lead approves it. | events `TimesheetSubmitted`, `TimesheetApproved` |

### Payroll Integration

| Term | Definition | Code |
|------|------------|------|
| Pay period | The period one payroll run covers, usually a month. | `PayPeriod` |
| Cut-off | The last moment a change can be included in a pay period. | `CutOff` |
| Pay change | One pay-relevant fact to send to the provider: new salary, unpaid days, overtime hours, leave payout. | `PayChange` |
| Export batch | All pay changes for one period sent to one provider. | `ExportBatch` |

## Words we banned

| Avoid | Why | Say instead |
|-------|-----|-------------|
| "Update the employee" | Which part? Address, contract, manager? Each has different rules and consequences | Hire, change terms, transfer, terminate |
| "Status" on its own | Every context has a different one | Name the event that happened |
| "Holiday" | Means both a public holiday and annual leave | *Public holiday* or *annual leave* |
| "Set the balance" | Balances are calculated from a ledger, never set | Grant, adjust, restore (each a ledger entry) |
| "Data", "info", "manager", "handler" in domain class names | Says nothing about the domain (and "manager" is a real person in HR) | A domain noun or verb |
| Vendor terms (requisition, `U07`, observance) outside the ACL | Leaks a vendor's model into ours | Our own terms |

## How to keep the language alive

- **Code review checks language.** A pull request that adds `EmployeeService.update(EmployeeDto)` should be questioned: what actually happened in the business? A transfer? A contract change?
- **Tests read like the glossary.** `shouldRestoreAnnualLeaveWhenSicknessIsReportedDuringApprovedLeave()` documents a rule better than any wiki.
- **When the language changes, rename the code.** If policy owners start saying "allowance" instead of "entitlement", rename the class. A gap between spoken and coded language is technical debt that grows.

Next: [design decisions and trade-offs](06-design-decisions.md).
