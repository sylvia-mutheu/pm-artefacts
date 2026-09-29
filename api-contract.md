# API contract: wallet top-up

**Status** Illustrative example written for this repository. Not a client deliverable.

I specify contracts rather than describing them, and I validate them in Postman and Swagger before engineering starts. An ambiguous contract is a defect that has not happened yet.

---

## POST /v1/wallet/topups

Creates a top-up request. Idempotent on Idempotency-Key.

### Headers

| Header | Required | Notes |
|---|---|---|
| Authorization | Yes | Bearer token |
| Idempotency-Key | Yes | Client-generated UUID v4. Replaying the same key returns the original transaction, never a new one |
| Content-Type | Yes | application/json |

### Request

```json
{
  "amount": { "value": "500.00", "currency": "KES" },
  "source": { "type": "mobile_money", "msisdn": "+254700000000" }
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| amount.value | string | Yes | Decimal string, two places. String not float, to avoid binary rounding on money |
| amount.currency | string | Yes | ISO 4217. Must match the wallet's currency |
| source.type | enum | Yes | mobile_money only in this version |
| source.msisdn | string | Yes | E.164. Must match a verified number on the account |

### Responses

**202 Accepted** Request created, awaiting user authorisation on the handset.

```json
{
  "transaction_id": "txn_01HQ3M9Z4K",
  "status": "PENDING",
  "amount": { "value": "500.00", "currency": "KES" },
  "created_at": "2026-09-29T09:14:02Z",
  "expires_at": "2026-09-29T09:16:02Z"
}
```

**200 OK** The Idempotency-Key has been seen before. Returns the existing transaction unchanged. No new debit is attempted.

**400 Bad Request** Malformed request.

**409 Conflict** The user already has a transaction in PENDING or UNKNOWN.

```json
{
  "error": "concurrent_transaction",
  "message": "A top-up is already in progress.",
  "existing_transaction_id": "txn_01HQ3M9Z4K"
}
```

**422 Unprocessable Entity** Valid request, rejected by a business rule.

```json
{
  "error": "limit_exceeded",
  "message": "Amount exceeds your daily limit.",
  "details": { "daily_limit": "10000.00", "remaining": "250.00", "currency": "KES" }
}
```

The details object is required on 422. An error the client cannot explain to the user is an error the user will call support about.

---

## GET /v1/wallet/topups/{transaction_id}

### Responses

**200 OK**

```json
{
  "transaction_id": "txn_01HQ3M9Z4K",
  "status": "UNKNOWN",
  "amount": { "value": "500.00", "currency": "KES" },
  "created_at": "2026-09-29T09:14:02Z",
  "resolved_at": null,
  "rail_reference": null,
  "history": [
    { "status": "PENDING", "at": "2026-09-29T09:14:02Z", "actor": "system" },
    { "status": "UNKNOWN", "at": "2026-09-29T09:16:02Z", "actor": "system", "reason": "rail_timeout" }
  ]
}
```

Status is one of PENDING, COMPLETED, FAILED, UNKNOWN, REVERSED. Clients must handle all five. A client that treats anything other than COMPLETED as a failure will tell users their money is gone when it is not.

---

## Contract rules I hold teams to

1. **Money is a string with an explicit currency, never a float.** This is not negotiable and it is not a preference.
2. **Every error carries a machine-readable code and a human-readable message.** The client displays the message. The client branches on the code. Neither substitutes for the other.
3. **Enumerations are closed and documented.** A status the client has never heard of is a production incident.
4. **Idempotency is specified at the contract level**, not left to the client to be careful.
5. **Null and absent mean the same thing, or the difference is documented.** Usually they should mean the same thing.
