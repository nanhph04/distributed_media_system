# Finance Service Database Notes

## Recommended database design

PlantUML ERD đề xuất cho finance-service:

```text
docs/database-design.puml
```

Các hình chi tiết được tách nhỏ để đưa vào Word dễ đọc:

```text
docs/database-design-wallet-ledger-detail.puml
docs/database-design-deposit-withdrawal-detail.puml
docs/database-design-payment-settlement-detail.puml
docs/database-design-event-detail.puml
```

Thiết kế này lược lại DB theo hướng tài chính rõ ràng hơn: `wallets` giữ số dư
hiện tại, `ledger_transactions` là sổ cái ghi nhận mọi biến động coin/tiền,
`deposits` và `withdrawals` là bảng nghiệp vụ, còn settlement, payout hold,
outbox và idempotency được tách riêng để phục vụ xử lý bất đồng bộ an toàn.
