# Identity Service Database Notes

## Recommended database design

PlantUML ERD đề xuất cho identity-service:

```text
docs/database-design.puml
```

Nếu cần hình có đầy đủ cột hơn, dùng thêm:

```text
docs/database-design-detail.puml
```

Thiết kế này lược bỏ các phần dư thừa của schema hiện tại: không lặp email ở
profile, không lưu refresh token gốc, đổi số điện thoại sang chuỗi, và tách rõ
nhóm bảng tài khoản, hồ sơ, phiên đăng nhập, OTP và outbox event.

## Auth OTP persistence

OTP flows use PostgreSQL as the source of truth so Redis outages do not block
registration, email verification, OTP resend, forgot password, or reset
password.

Tables:

- `auth_otp_sessions`: stores active/consumed OTP sessions by email and purpose.
  OTP values are stored as hashes, never raw codes.
  - Current purposes: `register`, `forgot`.
  - `forgot` sessions are used by forgot-password, resend forgot OTP, and
    reset-password.
- `auth_otp_rate_limits`: stores OTP send/resend rate-limit windows by action,
  scope, and key.
  - Current forgot-password actions:
    - `forgot`: request reset OTP by email, max 3 attempts per 3600 seconds.
    - `resend_forgot`: resend reset OTP by email, max 3 attempts per 3600
      seconds.

Redis is only a best-effort mirror/cache for OTP data. If Redis is unavailable,
the service continues through the database-backed flow.

Forgot-password Redis mirror keys:

```text
forgot:<email>
otp:forgot:<email>
```

Reset-password validates OTP against PostgreSQL, updates the account password,
revokes all refresh tokens for the account, consumes the OTP session, then
deletes the Redis mirror keys best-effort.
