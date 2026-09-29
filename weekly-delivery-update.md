# Weekly delivery update

**Status** Illustrative example written for this repository. Not a client deliverable.

This is the format I send every week to founders and client stakeholders. It is short on purpose. The test is whether someone who missed every meeting can read it in two minutes and know exactly where things stand and what they owe me.

I write it every week, not most weeks. A cadence that slips becomes a cadence nobody relies on, and then the only signal anyone gets is the bad news, late.

---

## Week 38 · Wallet top-up · 22 to 26 September

### Where we are

Mobile money top-up is feature complete in the test environment. The remaining work before UAT is reconciliation monitoring, which is two days.

We are on track for the 10 October date. That has not changed since last week.

### Shipped this week

- Idempotency enforced server side, replay returns the original transaction (US-02)
- UNKNOWN state implemented end to end, including the user-facing copy
- Reconciliation job running against the rail sandbox

### In progress

- Reconciliation monitoring and escalation path. Due Tuesday
- UAT environment seeding with tiered test accounts. Due Monday

### Decisions taken

- **No automatic retry after a rail timeout.** Recorded as DL-001. A retry against a rail that may already have debited risks a double debit, which is worse for the user than a delay. Reconciliation resolves these instead.

### Risks

| Risk | Impact if it lands | What I am doing |
|---|---|---|
| Rail sandbox timeout simulation is not identical to production behaviour | T-11 may pass in UAT and fail in production | Requested the provider's documented timeout behaviour. Chasing since Wednesday, no response yet |
| Reconciliation cycle count not yet agreed | Blocks the escalation path and therefore UAT exit | Need a decision from you this week. See below |

### What I need from you

1. **How many reconciliation cycles before we escalate to support?** My recommendation is three. I need this by Wednesday or the UAT exit criteria cannot be finalised.
2. **Confirmation that a REVERSED transaction does not count against the daily limit.** This is a compliance question rather than a product one, so I would rather you answer it than me assume it.

### Not being worked on, so you know

Card top-ups remain deferred, per DL-005. Withdrawals have not been scoped.

---

## Why the format is shaped this way

- **The date claim comes first**, and I say whether it has changed. If a stakeholder reads one line, that is the line.
- **Decisions are recorded in the update as well as the log**, because a decision nobody saw is a decision that gets reopened.
- **Risks name the impact, not just the risk**, and say what I am already doing. A risk register with no owner is a list of worries.
- **What I need from you carries a deadline.** Requests without deadlines get read and not actioned.
- **Not being worked on prevents the most common misunderstanding in remote delivery**, which is a stakeholder assuming something is quietly in hand.
