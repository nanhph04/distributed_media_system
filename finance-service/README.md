# Finance Service

**Source repo:** https://github.com/nanhph04/finance-service-da.git

Finance Service quản lý các nghiệp vụ tài chính của Distributed Media System, bao gồm ví người dùng, nạp tiền, thanh toán nội bộ, giao dịch, doanh thu và yêu cầu rút tiền.

## Vai trò chính

- Quản lý ví và số dư của người dùng/chủ kênh.
- Tạo yêu cầu nạp tiền và xử lý webhook từ cổng thanh toán.
- Cung cấp API thanh toán nội bộ cho các nghiệp vụ như mở khóa video hoặc mua gói hội viên.
- Ghi nhận transaction, ledger và trạng thái thanh toán.
- Quản lý yêu cầu rút tiền của chủ kênh.
- Cho phép quản trị viên duyệt, hoàn tất hoặc từ chối yêu cầu rút tiền.
- Phát event tài chính để các service khác cập nhật quyền truy cập hoặc trạng thái liên quan.

## Tài liệu

| Tài liệu | Mục đích |
| --- | --- |
| [`docs/API.md`](docs/API.md) | Danh sách API Finance và contract tích hợp. |
| [`docs/DATABASE.md`](docs/DATABASE.md) | Thiết kế dữ liệu ví, giao dịch, ledger, deposit và withdrawal. |
| [`docs/ENV.md`](docs/ENV.md) | Biến môi trường quan trọng. |
| [`docs/EVENTS.md`](docs/EVENTS.md) | Event tài chính publish/consume. |
| [`docs/FLOWS.md`](docs/FLOWS.md) | Luồng nghiệp vụ tài chính chính. |
| [`docs/SEQUENCE_DIAGRAMS.md`](docs/SEQUENCE_DIAGRAMS.md) | Mục lục sequence diagram. |
| [`docs/ACTIVITY_DIAGRAMS.md`](docs/ACTIVITY_DIAGRAMS.md) | Mục lục activity diagram. |

## Biểu đồ tiêu biểu

- Rút tiền.
- Quản trị viên duyệt yêu cầu rút tiền.
- Thanh toán nội bộ cho các nghiệp vụ media trả phí.
