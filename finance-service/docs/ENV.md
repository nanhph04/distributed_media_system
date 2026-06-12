# FINANCE SERVICE ENV

Kafka topics used by payment and membership auto-renew:

```text
KAFKA_VIDEO_PAYMENT_SUCCESS_TOPIC=video.payment.success
KAFKA_MEMBERSHIP_PAYMENT_SUCCESS_TOPIC=membership.payment.success
KAFKA_MEMBERSHIP_AUTO_RENEW_REQUESTED_TOPIC=membership.auto_renew.requested
KAFKA_MEMBERSHIP_AUTO_RENEW_FAILED_TOPIC=membership.auto_renew.failed
```

When `KAFKA_AUTO_CREATE_TOPICS=true`, include these topics in
`KAFKA_TOPICS_TO_CREATE`.

Internal service-to-service payment charge:

```text
INTERNAL_GATEWAY_SECRET=finance-gateway-secret
```

- `INTERNAL_GATEWAY_SECRET`: finance-service internal gateway secret expected in
  the `x-internal-secret` header when other services call
  `POST /api/internal/payments/charge`.

Withdrawal configuration:

```text
WITHDRAWAL_EXCHANGE_RATE=100
WITHDRAWAL_FEE_PERCENT=5
```

- `WITHDRAWAL_EXCHANGE_RATE`: VND amount per 1 coin when a user withdraws.
- `WITHDRAWAL_FEE_PERCENT`: system withdrawal fee percent, applied after coin
  is converted to VND.

Withdrawal amount formula:

```text
grossMoneyAmount = coinAmount * WITHDRAWAL_EXCHANGE_RATE
feeAmount = floor(grossMoneyAmount * WITHDRAWAL_FEE_PERCENT / 100)
moneyAmount = grossMoneyAmount - feeAmount
```
