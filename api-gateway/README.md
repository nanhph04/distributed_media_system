# API Gateway

**Source repo:** https://github.com/nanhph04/api_gateway_da.git

API Gateway là cổng vào chung của Distributed Media System. Frontend/Mobile nên gọi API qua gateway thay vì gọi trực tiếp vào các service nội bộ.

## Vai trò chính

- Định tuyến request từ client đến đúng service nội bộ.
- Chuẩn hóa public path cho Frontend/Mobile.
- Gắn các header nội bộ như `x-user-id`, `x-user-role`, `x-internal-secret` khi chuyển tiếp request.
- Che giấu địa chỉ nội bộ của các service phía sau.
- Là điểm tập trung để áp dụng xác thực, phân quyền, CORS và logging request.

## Các nhóm route chính

| Nhóm route public | Service xử lý | Mục đích |
| --- | --- | --- |
| `/api/auth/*` | Identity Service | Đăng ký, đăng nhập, refresh token, đăng xuất, quên mật khẩu. |
| `/api/user/*` | Identity Service | Hồ sơ người dùng và quản trị người dùng. |
| `/api/media/*` | Media Service | Kênh, video, danh mục, quyền xem, hội viên và nội dung media. |
| `/api/finance/*` hoặc `/api/*` theo cấu hình gateway | Finance Service | Ví, nạp tiền, thanh toán và rút tiền. |
| Webhook/payment callback | Finance Service | Nhận callback từ cổng thanh toán hoặc luồng nội bộ. |

## Quy tắc tích hợp Frontend

- Frontend chỉ cần biết base URL của API Gateway.
- Token truy cập gửi qua header `Authorization: Bearer <accessToken>` với các API protected.
- Các API dùng refresh token/cookie cần bật `credentials: "include"` khi gọi bằng `fetch`.
- Không tự set các header nội bộ như `x-internal-secret`; gateway chịu trách nhiệm gắn các header này.

## Tài liệu liên quan

- Identity API: [`../identity-service/docs/API.md`](../identity-service/docs/API.md)
- Media API: [`../media-service/docs/API.md`](../media-service/docs/API.md)
- Finance API: [`../finance-service/docs/API.md`](../finance-service/docs/API.md)
