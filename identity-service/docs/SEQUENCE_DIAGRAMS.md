# Identity Service Sequence Diagrams

Last updated: 2026-06-07

Tai lieu nay la muc luc sequence diagram cho identity-service. Moi bieu do
duoc tach thanh mot file rieng trong thu muc `docs/sequence-diagrams/`.

Direct service prefix hien tai: `/api/identity`.

Gateway public path hien tai:

- Auth: `/api/auth/*`
- Profile: `/api/user/*`

## Quy uoc

- `->`: Goi dong bo
- `-->`: Tra ket qua
- `->>`: Gui su kien hoac callback bat dong bo

## Danh sach bieu do

| STT | Chuc nang | File |
| --- | --- | --- |
| 1 | Dang nhap | `docs/sequence-diagrams/login.puml` |
| 2 | Tao tai khoan | `docs/sequence-diagrams/register-account.puml` |
| 3 | Dang xuat | `docs/sequence-diagrams/logout.puml` |
| 4 | Quen mat khau | `docs/sequence-diagrams/forgot-password.puml` |
| 5 | Cap nhat profile | `docs/sequence-diagrams/update-profile.puml` |
| 6 | Khóa tài khoản (Admin) - tổng quát | `docs/sequence-diagrams/manage-admin-users/manage-admin-users-overview.puml` |
| 7 | Khóa tài khoản (Admin) - Identity Service khóa tài khoản | `docs/sequence-diagrams/manage-admin-users/manage-admin-users-detail-01-identity-lock.puml` |
| 8 | Khóa tài khoản (Admin) - Media Service xử lý sự kiện | `docs/sequence-diagrams/manage-admin-users/manage-admin-users-detail-02-media-event.puml` |
| 9 | Khóa tài khoản (Admin) - Finance Service xử lý sự kiện | `docs/sequence-diagrams/manage-admin-users/manage-admin-users-detail-03-finance-event.puml` |
