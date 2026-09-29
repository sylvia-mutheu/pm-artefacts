# User stories and acceptance criteria: wallet top-up

**Status** Illustrative example written for this repository. Not a client deliverable.

A story is only finished when done stops being a matter of opinion. Every story below is written so that a QA engineer who was not in the refinement session can test it without asking anyone a question.

---

## US-01 Top up the wallet from mobile money

**As a** verified user  
**I want to** move funds from my mobile money account into my wallet  
**So that** I can spend from the wallet without leaving the app

### Acceptance criteria

- **Given** I am verified and within my daily limit, **when** I submit a valid amount, **then** a transaction is created in PENDING and I am shown a screen telling me to authorise on my handset
- **Given** I submit an amount above my per-transaction limit, **when** I confirm, **then** I see the limit and my remaining allowance, and no transaction record is created and the rail is not called
- **Given** I authorise successfully on my handset, **when** the rail confirms, **then** my wallet balance reflects the new amount within the defined confirmation window and I receive a receipt showing the amount, the rail reference and the timestamp
- **Given** the rail declines, **when** the decline is returned, **then** I see the reason in plain language, not a rail error code, and my balance is unchanged

### Out of scope for this story

Reconciliation of unresolved transactions. See US-03.

---

## US-02 Be protected from double debits

**As a** user with a slow or unreliable connection  
**I want** the app to stop me spending twice by accident  
**So that** I do not lose money to my own retrying

### Acceptance criteria

- **Given** I have a transaction in PENDING or UNKNOWN, **when** I attempt a second top-up, **then** the attempt is blocked and I am shown the status of the existing transaction
- **Given** my client retries the same request with the same idempotency key, **when** the server receives it, **then** the server returns the original transaction rather than creating a new one, and no second debit is attempted
- **Given** a top-up has timed out, **when** I look for a retry button, **then** there is none, and the screen explains that the top-up is being confirmed and I will be notified

### Notes for QA

The third criterion is deliberately a negative test. The absence of the retry affordance is the requirement, not an oversight.

---

## US-03 Know what happened to a top-up that did not resolve

**As a** user whose top-up timed out  
**I want to** be told the outcome once it is known  
**So that** I am not left guessing whether my money moved

### Acceptance criteria

- **Given** my transaction is in UNKNOWN, **when** I view my transaction history, **then** it appears with a status that says it is being confirmed, and it is not labelled as failed
- **Given** reconciliation resolves my transaction to COMPLETED, **when** resolution occurs, **then** my balance updates and I receive a notification stating the outcome and the original amount
- **Given** reconciliation resolves my transaction to FAILED, **when** resolution occurs, **then** I receive a notification confirming no funds left my account, and my balance is unchanged
- **Given** my transaction cannot be resolved after the defined number of cycles, **when** the final cycle completes, **then** it is escalated to support with the full state history attached, and I am told that a person is looking into it

---

## US-04 Support can reconstruct any transaction

**As a** support agent  
**I want** the complete history of a transaction  
**So that** I can answer a user without guessing

### Acceptance criteria

- **Given** any transaction reference, **when** I look it up, **then** I see every state transition with a timestamp, the actor or system that caused it, and the rail's response payload
- **Given** a transaction in REVERSED, **when** I look it up, **then** I can see both the original debit and the refund, linked to each other

---

## A note on how these are written

Three things I insist on and will argue for:

1. **Every story names its out of scope.** A story without a boundary expands until the sprint ends.
2. **Failure paths get their own criteria, not a footnote.** In payments the failure path is the product.
3. **Negative requirements are stated explicitly.** There is no retry button is a requirement. If it is not written down, someone helpfully adds one.
