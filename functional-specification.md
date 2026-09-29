# Functional specification: wallet top-up via mobile money

**Status** Illustrative example written for this repository. Not a client deliverable.  
**Owner** Sylvia Mutheu  
**Version** 1.0

---

## 1. Purpose

Allow a user to move funds from their mobile money account into their in-app wallet balance.

## 2. Out of scope

Withdrawals from wallet to mobile money. Card top-ups. Top-ups initiated by a third party on the user's behalf. Each of these has its own compliance surface and belongs in its own specification.

## 3. Actors

| Actor | Role |
|---|---|
| User | Initiates the top-up, holds the mobile money account and the wallet |
| Mobile money provider | External rail. Authoritative on whether funds left the user's account |
| Ledger service | Internal. Authoritative on the wallet balance |
| Reconciliation job | Internal, scheduled. Resolves transactions left in an unknown state |

## 4. Preconditions

- User is authenticated
- User's account has completed identity verification to at least tier 1
- The mobile money number on file is verified and matches the initiating account
- Requested amount is within the per-transaction and daily limits for the user's tier

## 5. Main flow

1. User enters an amount and confirms
2. System validates the amount against limits and returns an error without contacting the rail if validation fails
3. System creates a transaction record in state PENDING with an idempotency key
4. System calls the mobile money provider to request a debit
5. Provider prompts the user to authorise on their handset
6. Provider returns a result
7. On success, the ledger service credits the wallet and the transaction moves to COMPLETED
8. User sees the updated balance and a receipt

## 6. States

This is the section most specifications get wrong by leaving it implicit.

| State | Meaning | Exit conditions |
|---|---|---|
| PENDING | Request created, rail not yet responded | To COMPLETED, FAILED or UNKNOWN |
| COMPLETED | Funds confirmed debited and wallet credited | Terminal |
| FAILED | Rail confirmed no debit occurred | Terminal |
| UNKNOWN | Rail did not respond within the timeout, or responded ambiguously | To COMPLETED or FAILED via reconciliation only |
| REVERSED | Debit occurred but the wallet credit could not be applied, and the debit has been refunded | Terminal |

UNKNOWN is not an error state. It is a legitimate outcome and the product must be designed for it. A transaction in UNKNOWN has not failed and must never be presented to the user as failed, because the user's money may well have moved.

## 7. Failure handling

| Failure | Behaviour |
|---|---|
| Validation failure before the rail is called | Immediate error to user. No transaction record created |
| Rail returns explicit decline | Transaction to FAILED. User told the reason the rail gave, in plain language |
| Rail times out | Transaction to UNKNOWN. User told that the top-up is being confirmed and that they will be notified. No retry is offered |
| Rail confirms debit, ledger credit fails | Transaction to REVERSED. Automatic refund initiated. Incident raised, because this should not happen |
| User retries while a transaction is PENDING or UNKNOWN | Blocked. The idempotency key prevents a second debit. User sees the status of the existing transaction |

**Retry policy.** The system does not automatically retry a top-up after a timeout. A retry against a rail that may have already debited the user risks a double debit, which is a materially worse outcome for the user than a delay. Resolution happens through reconciliation, not retry.

## 8. Reconciliation

The reconciliation job runs on a schedule and queries the provider for the authoritative status of every transaction in UNKNOWN older than the timeout window. It moves each one to COMPLETED or FAILED and notifies the user of the resolution. A transaction that cannot be resolved after the defined number of cycles is escalated to support with the full transaction history attached.

## 9. Non-functional requirements

- Every transaction carries an idempotency key, generated client side and honoured server side
- The full state history of a transaction is retained and queryable for audit, including who or what caused each transition
- Limit checks are enforced server side. Client-side checks are a courtesy, not a control
- No transaction state may be inferred from the user interface. The ledger is authoritative

## 10. Open questions

1. What is the timeout window before a PENDING transaction becomes UNKNOWN? Needs the rail's own latency distribution, not a guess
2. How many reconciliation cycles before escalation to support?
3. Does a REVERSED transaction count against the user's daily limit? This is a compliance question, not a product one
