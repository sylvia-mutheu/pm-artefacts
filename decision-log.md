# Decision log: wallet top-up

**Status** Illustrative example written for this repository. Not a client deliverable.

A decision log exists so that a decision is made once. Without one, the same argument returns every few months with different people in the room and no memory of why it was settled.

Each entry records what was decided, what it was decided instead of, why, and what would make us revisit it. That last column is the one most teams omit, and it is the one that makes the log honest rather than defensive.

---

## DL-001 No automatic retry after a rail timeout

**Date** 2026-08-12  
**Status** Accepted  
**Decided by** Product, engineering lead, compliance

**Context.** When the mobile money rail times out we do not know whether the user was debited. The obvious move is to retry.

**Decision.** We do not retry. The transaction moves to UNKNOWN and is resolved by reconciliation.

**Alternatives considered.**
- *Retry once after a short delay.* Rejected. If the first attempt did debit, the retry double debits. A double debit is materially worse for the user than a delay, and materially worse for us than a support ticket.
- *Ask the user whether to retry.* Rejected. This pushes a decision onto the user that they have less information to make than we do.

**Consequences.** Users wait longer for resolution in the timeout case. We accept that cost. Reconciliation becomes load bearing, so it needs monitoring and an escalation path.

**Revisit if.** The rail exposes a reliable pre-check that tells us authoritatively whether a debit occurred, at which point retry becomes safe.

---

## DL-002 Money represented as decimal strings, not floats

**Date** 2026-08-12  
**Status** Accepted  
**Decided by** Product, engineering lead

**Context.** The mobile client team asked for numeric amounts to simplify parsing.

**Decision.** Amounts are decimal strings with an explicit currency code, everywhere, including internal services.

**Alternatives considered.**
- *Floats.* Rejected. Binary floating point cannot represent common decimal amounts exactly, and the resulting drift shows up in reconciliation as unexplained fractions.
- *Integer minor units.* A reasonable option, rejected only because not every currency we may add has two minor units and the ambiguity costs more than the parsing does.

**Consequences.** Clients do slightly more work. Reconciliation stops producing mystery rounding differences.

**Revisit if.** Never, realistically. This is recorded so it is not reopened.

---

## DL-003 Deterministic limit checks server side only

**Date** 2026-08-19  
**Status** Accepted  
**Decided by** Product, engineering lead, compliance

**Context.** Client-side limit checks give faster feedback.

**Decision.** Client-side checks are permitted as a courtesy. The server check is the control, and the server never trusts the client's assessment.

**Consequences.** A small amount of duplicated logic. Limits cannot be bypassed by an altered client.

**Revisit if.** Nothing. Recorded because someone always proposes removing the duplication.

---

## DL-004 Reconciliation notifies the user directly

**Date** 2026-09-02  
**Status** Accepted  
**Decided by** Product, support lead

**Context.** When reconciliation resolves an UNKNOWN transaction, someone has to tell the user. The options were a notification, or leaving it for the user to discover in their history.

**Decision.** The system notifies the user on resolution, in both the success and failure case.

**Alternatives considered.**
- *Silent resolution.* Rejected. The user was last told we are confirming this. Leaving that unanswered is what generates the support contact we were trying to avoid.
- *Notify only on success.* Rejected. A user who was told nothing after a failure assumes the money is gone.

**Consequences.** Additional notification volume. Support expects this to reduce inbound contacts, which is worth measuring after the first month rather than asserting now.

**Revisit if.** Notification volume becomes a fatigue problem, in which case batch rather than suppress.

---

## DL-005 Deferred: card top-ups

**Date** 2026-09-02  
**Status** Deferred, not rejected  
**Decided by** Product

**Context.** Card top-ups were requested for the first release.

**Decision.** Out of scope for this release.

**Reason.** Card brings chargebacks, a different compliance surface and a different failure model. Bolting it onto a specification written for a push-based rail would produce a worse version of both.

**Revisit if.** Mobile money top-up is stable in production and the card work can get its own specification.
