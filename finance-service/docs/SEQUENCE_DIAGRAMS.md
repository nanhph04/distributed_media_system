# Finance Service Sequence Diagrams

Last updated: 2026-06-07

Tài liệu này là mục lục sequence diagram cho finance-service. Mỗi biểu đồ
được tách thành một file riêng trong thư mục `docs/sequence-diagrams/`.

Gateway public path hiện tại:

- Finance: `/api/finance/*`

## Quy ước

- `->`: Gọi đồng bộ
- `-->`: Trả kết quả
- `->>`: Gửi sự kiện hoặc callback bất đồng bộ

## Danh sách biểu đồ

| STT | Chức năng | File |
| --- | --- | --- |
| 1 | Nạp tiền - tổng quát | `docs/sequence-diagrams/deposit-money/deposit-money-overview.puml` |
| 2 | Nạp tiền - chi tiết tạo link thanh toán | `docs/sequence-diagrams/deposit-money/deposit-money-detail-01-create-payment-link.puml` |
| 3 | Nạp tiền - chi tiết webhook và cộng coin | `docs/sequence-diagrams/deposit-money/deposit-money-detail-02-webhook-settlement.puml` |
| 4 | Nạp tiền - chi tiết đối soát pending | `docs/sequence-diagrams/deposit-money/deposit-money-detail-03-reconcile-pending.puml` |
| 5 | Rút tiền - tổng quát | `docs/sequence-diagrams/withdraw-money/withdraw-money-overview.puml` |
| 6 | Rút tiền - chi tiết kiểm tra điều kiện | `docs/sequence-diagrams/withdraw-money/withdraw-money-detail-01-validation.puml` |
| 7 | Rút tiền - chi tiết tạo yêu cầu chờ duyệt | `docs/sequence-diagrams/withdraw-money/withdraw-money-detail-02-create-pending-request.puml` |
| 8 | Duyệt yêu cầu rút tiền - tổng quát (Admin) | `docs/sequence-diagrams/review-withdrawal-request/review-withdrawal-request-overview.puml` |
| 9 | Duyệt yêu cầu rút tiền - xem danh sách và chi tiết | `docs/sequence-diagrams/review-withdrawal-request/review-withdrawal-request-detail-01-list-and-detail.puml` |
| 10 | Duyệt yêu cầu rút tiền - hoàn tất yêu cầu | `docs/sequence-diagrams/review-withdrawal-request/review-withdrawal-request-detail-02-complete.puml` |
| 11 | Duyệt yêu cầu rút tiền - từ chối yêu cầu | `docs/sequence-diagrams/review-withdrawal-request/review-withdrawal-request-detail-03-reject.puml` |
