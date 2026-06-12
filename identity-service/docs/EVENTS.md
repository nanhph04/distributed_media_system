# IDENTITY SERVICE EVENTS

All outbound events use the shared integration event envelope:

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

1. user.registered

---

Purpose:

- Published after a user completes email verification and identity-service has
  created both account and profile.
- finance-service should consume this event to create the user's wallet.

Topic:

```text
user.registered
```

Key:

```text
<userId>
```

Envelope:

```json
{
  "eventId": "uuid-v4",
  "eventType": "user.registered",
  "aggregateId": "user-id",
  "timestamp": "2026-05-28T00:00:00.000Z",
  "version": 1,
  "traceId": "trace-id",
  "sourceService": "identity-service",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "createdAt": "2026-05-28T00:00:00.000Z"
  }
}
```

Producer reliability:

- identity-service stores this event in `outbox_messages` in the same database
  transaction that creates the account and profile.
- `OutboxPublisherService` publishes pending outbox messages to Kafka and marks
  them as published after Kafka send succeeds.
- If Kafka publish fails, the outbox row is marked `failed` with retry metadata
  and retried later.

Consumer rules:

- Consumers must implement idempotency by `eventId`.
- finance-service should create one wallet per `data.userId`.
- Wallet creation must tolerate duplicate delivery because an outbox publisher
  can publish successfully and crash before marking the row as published.

2. user.status.changed

---

Purpose:

- Published after an admin changes a normal user's account status.
- Consumers such as media-service and finance-service should use this event to
  disable/restore user-dependent actions.

Topic:

```text
user.status.changed
```

Key:

```text
<userId>
```

Envelope:

```json
{
  "eventId": "uuid-v4",
  "eventType": "user.status.changed",
  "aggregateId": "user-id",
  "timestamp": "2026-05-16T00:00:00.000Z",
  "version": 1,
  "traceId": "trace-id",
  "sourceService": "identity-service",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "previousStatus": "active",
    "currentStatus": "suspended",
    "changedByAdminId": "admin-id",
    "reason": "Violation of platform policy",
    "changedAt": "2026-05-16T00:00:00.000Z"
  }
}
```

Consumer rules:

- Consumers must implement idempotency by `eventId`.
- Treat `suspended` as the locked state.
- Treat `active` as restored access.
- When `currentStatus = suspended`, identity-service also revokes refresh tokens
  and emits the per-user SSE event `session.revoked`.

Producer reliability:

- identity-service stores `user.status.changed` in `outbox_messages` in the same
  database transaction that updates the account status.
- `OutboxPublisherService` publishes the pending outbox row to Kafka and retries
  later if Kafka is temporarily unavailable.

## SSE) session.revoked

Purpose:

- Emitted by identity-service to the authenticated user's SSE connection when an
  admin suspends the account.
- FE uses it to clear accessToken from localStorage, clear user state, close the
  stream, and redirect to login/account-disabled UI.

Endpoint:

```text
GET /api/auth/session/events
```

Event:

```text
event: session.revoked
data: {"reason":"ACCOUNT_SUSPENDED","message":"Tài khoản đã bị vô hiệu hóa. Vui lòng kiểm tra email để biết lý do.","revokedAt":"2026-05-17T00:00:00.000Z"}
```

Rules:

- This is not a Kafka event.
- It is scoped to the authenticated user only.
- FE must keep 401/403 cleanup fallback because SSE can disconnect.

3. membership.auto_renew.reminder_requested

---

Purpose:

- Consumed by identity-service.
- Sends an email reminder before media-service asks finance-service to renew a membership.

Topic:

```text
membership.auto_renew.reminder_requested
```

Envelope:

```json
{
  "eventId": "uuid-v4",
  "eventType": "membership.auto_renew.reminder_requested",
  "aggregateId": "membership-record-id",
  "timestamp": "2026-05-17T00:00:00.000Z",
  "version": 1,
  "traceId": "trace-id",
  "sourceService": "media-service",
  "data": {
    "membershipRecordId": "membership-record-id",
    "userId": "user-id",
    "channelId": "channel-id",
    "channelName": "Creator Channel",
    "membershipTierId": "tier-id",
    "tierName": "Premium",
    "coinAmount": 100,
    "renewalDate": "2026-05-18T00:00:00.000Z"
  }
}
```

Consumer rules:

- Lookup account by `data.userId`.
- Send email only when account exists and `isEmailVerified = true`.
- Use cache idempotency key
  `identity:email:membership-renew-reminder:{membershipRecordId}:{renewalDate}`
  to avoid duplicate email on Kafka retries.
- Email must include channel name, tier name, coin amount, renewal date, and a reminder that user can top up coin or turn off auto-renew before renewal.
