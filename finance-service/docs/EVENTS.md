FINANCE SERVICE EVENTS
======================

All integration events use the shared envelope:

```ts
interface IIntegrationEvent<T = unknown> {
  eventId: string;
  eventType: string;
  aggregateId: string;
  timestamp: string;
  version: number;
  traceId: string;
  spanId?: string;
  sourceService: string;
  data: T;
}
```

1) membership.auto_renew.requested
----------------------------------

Purpose:

- Consumed by finance-service.
- Requests charging a user's wallet for a due membership renewal.

Topic:

```text
membership.auto_renew.requested
```

Data:

```json
{
  "membershipRecordId": "membership-record-id",
  "userId": "user-id",
  "channelId": "channel-id",
  "channelOwnerId": "creator-user-id",
  "membershipTierId": "tier-id",
  "coinAmount": 100,
  "currentExpiryDate": "2026-05-18T00:00:00.000Z",
  "paymentType": "renew",
  "idempotencyKey": "membership-renew:membership-record-id:2026-05-18T00:00:00.000Z"
}
```

2) membership.payment.success
-----------------------------

Purpose:

- Published by finance-service after a video or membership payment succeeds.
- For membership auto-renew, `paymentType` is `renew`.
- For new purchase flows, media-service may unlock synchronously after the
  internal finance charge response and still consume this event for idempotent
  reconciliation.

Membership data:

```json
{
  "userId": "user-id",
  "channelId": "channel-id",
  "membershipTierId": "tier-id",
  "channelOwnerId": "creator-user-id",
  "coinAmount": 100,
  "paymentTransactionId": "txn-id",
  "paymentType": "renew",
  "chargedCoinAmount": 100,
  "ledgerReferenceId": "txn-id",
  "membershipRecordId": "membership-record-id",
  "currentExpiryDate": "2026-05-18T00:00:00.000Z",
  "expiryDate": null
}
```

3) membership.auto_renew.failed
-------------------------------

Purpose:

- Published by finance-service when it cannot charge an auto-renew request.
- media-service consumes it to retry or disable auto-renew.

Topic:

```text
membership.auto_renew.failed
```

Data:

```json
{
  "membershipRecordId": "membership-record-id",
  "userId": "user-id",
  "channelId": "channel-id",
  "membershipTierId": "tier-id",
  "reasonCode": "INSUFFICIENT_BALANCE",
  "retryable": true,
  "idempotencyKey": "membership-renew:membership-record-id:2026-05-18T00:00:00.000Z"
}
```

4) channel.status.changed
-------------------------

Purpose:

- Consumed by finance-service from media-service.
- When a channel is suspended, finance-service stores an active payout hold for
  `channelId/channelOwnerId`.
- While the hold is active, owner withdrawals are rejected and due revenue
  settlements for that owner are skipped instead of moving from frozen to
  available balance.
- When a channel is restored to `active`, the hold is deactivated.

Topic:

```text
channel.status.changed
```

Data:

```json
{
  "channelId": "channel-id",
  "channelOwnerId": "creator-user-id",
  "previousStatus": "active",
  "currentStatus": "suspended",
  "changedByAdminId": "admin-id",
  "reason": "DMCA violation",
  "changedAt": "2026-05-17T00:00:00.000Z"
}
```

Consumer rules:

- Consumers must be idempotent by `eventId`.
- Channel ban does not freeze the whole user wallet, so the user can still spend
  normally.

5) user.status.changed
----------------------

Purpose:

- Consumed by finance-service from identity-service.
- When `currentStatus = suspended`, finance-service freezes the user's wallet.

Topic:

```text
user.status.changed
```

Data fields used:

```json
{
  "userId": "user-id",
  "previousStatus": "active",
  "currentStatus": "suspended",
  "changedByAdminId": "admin-id",
  "reason": "Policy violation",
  "changedAt": "2026-05-17T00:00:00.000Z"
}
```

Consumer rules:

- Consumers must be idempotent by `eventId`.
- Account ban freezes the mixed user wallet; user spending and withdrawals are
  blocked through wallet inactive checks.
