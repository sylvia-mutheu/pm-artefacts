# UAT test plan: wallet top-up

**Status** Illustrative example written for this repository. Not a client deliverable.

I came up through QA and I run this pass myself before a client sees the build. The purpose of UAT is not to confirm that the feature works. It is to find the states where it does not, before someone's money is involved.

---

## 1. Entry criteria

UAT does not begin until all of the following are true. This list exists so that is it ready for UAT is not a negotiation.

- All stories in scope are code complete and merged to the test environment
- The rail sandbox is available and its failure simulation modes are confirmed working
- Test accounts exist at each verification tier, with known limits
- The reconciliation job can be triggered manually in the test environment
- Known defects are documented, with a severity assigned

## 2. Exit criteria

- No open severity 1 or severity 2 defects
- Every scenario below has been executed and its result recorded
- Severity 3 defects are documented and explicitly accepted by the product owner, in writing

## 3. Severity definitions

| Severity | Definition |
|---|---|
| 1 | Money is lost, duplicated, or misreported to the user. Release blocked, no discussion |
| 2 | A user is left unable to complete or unable to understand the outcome. Release blocked |
| 3 | Cosmetic, or a workaround exists that a user would plausibly find |

Severity 1 is defined tightly on purpose. In a payments product, incorrect information about money is the same class of failure as losing it.

## 4. Scenarios

### Happy path

| ID | Scenario | Expected |
|---|---|---|
| T-01 | Valid amount, user authorises promptly | Wallet credited, receipt shows amount, rail reference, timestamp |
| T-02 | Minimum permitted amount | Accepted |
| T-03 | Exactly the per-transaction limit | Accepted |

### Validation

| ID | Scenario | Expected |
|---|---|---|
| T-04 | One unit above the per-transaction limit | Rejected before the rail is called. Limit and remaining allowance both shown |
| T-05 | Amount that would exceed the daily limit | Rejected with remaining allowance shown |
| T-06 | Unverified mobile number | Rejected. Message explains what to do about it |
| T-07 | Currency mismatch with wallet | Rejected |

### Failure paths, which is where the real work is

| ID | Scenario | Expected |
|---|---|---|
| T-08 | Rail declines explicitly | FAILED. Plain language reason. Balance unchanged |
| T-09 | Rail times out | UNKNOWN. User told it is being confirmed. **Not** labelled failed. No retry offered |
| T-10 | User declines on handset | FAILED. Balance unchanged |
| T-11 | User does not respond before expiry | UNKNOWN or FAILED per rail behaviour. Verify which, do not assume |
| T-12 | Rail confirms debit, ledger credit fails | REVERSED. Refund initiated. Incident raised |
| T-13 | Connection drops after submit, before response | Transaction visible and correct on reopening the app |

### Idempotency and concurrency

| ID | Scenario | Expected |
|---|---|---|
| T-14 | Same idempotency key replayed | Original transaction returned. No second debit |
| T-15 | Second top-up attempted while one is PENDING | Blocked with 409. Existing transaction shown |
| T-16 | Two requests submitted simultaneously, different keys | Exactly one succeeds. Verify no double debit at the rail |

### Reconciliation

| ID | Scenario | Expected |
|---|---|---|
| T-17 | UNKNOWN resolves to success | Balance updates. User notified with the original amount |
| T-18 | UNKNOWN resolves to failure | User notified that no funds left the account. Balance unchanged |
| T-19 | Unresolvable after the defined cycles | Escalated to support with full state history attached |

### Audit

| ID | Scenario | Expected |
|---|---|---|
| T-20 | Look up any transaction from the above | Full state history, timestamps, actor per transition, rail payload all present |

## 5. What I do with the results

Every executed scenario gets a result and, where it failed, a reproduction path with the exact inputs. A defect report that says top-up failed is not a defect report.

The output of the pass is a single page: what was tested, what passed, what failed with severity, and a recommendation to release or not. The recommendation is mine to make and the product owner's to overrule, in writing.
