# Identity Service

**Source repo:** https://github.com/nanhph04/identity-service-da.git

Identity Service quản lý định danh người dùng trong Distributed Media System, bao gồm xác thực, phiên đăng nhập, hồ sơ cá nhân và các nghiệp vụ quản trị người dùng.

## Vai trò chính

- Đăng ký tài khoản và đăng nhập.
- Phát hành `accessToken` và quản lý `refresh_token` qua httpOnly cookie.
- Cung cấp API lấy/cập nhật hồ sơ người dùng.
- Hỗ trợ quên mật khẩu và đăng xuất.
- Cho phép quản trị viên khóa/mở khóa người dùng.
- Phát event đồng bộ trạng thái người dùng sang Media Service và Finance Service khi cần.

## Tài liệu

| Tài liệu | Mục đích |
| --- | --- |
| [`docs/API.md`](docs/API.md) | Danh sách API Identity dành cho Frontend/Mobile khi đi qua API Gateway. |
| [`docs/DATABASE.md`](docs/DATABASE.md) | Thiết kế dữ liệu và các bảng chính của Identity Service. |
| [`docs/ENV.md`](docs/ENV.md) | Biến môi trường quan trọng. |
| [`docs/EVENTS.md`](docs/EVENTS.md) | Event publish/consume liên quan đến người dùng. |
| [`docs/FLOWS.md`](docs/FLOWS.md) | Mô tả các luồng nghiệp vụ chính. |
| [`docs/SEQUENCE_DIAGRAMS.md`](docs/SEQUENCE_DIAGRAMS.md) | Mục lục sequence diagram. |
| [`docs/ACTIVITY_DIAGRAMS.md`](docs/ACTIVITY_DIAGRAMS.md) | Mục lục activity diagram. |

## Biểu đồ tiêu biểu

- Đăng ký tài khoản.
- Đăng nhập.
- Đăng xuất.
- Quên mật khẩu.
- Cập nhật hồ sơ.
- Quản trị viên quản lý người dùng.
