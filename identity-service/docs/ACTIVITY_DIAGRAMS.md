# Identity Service Activity Diagrams

Last updated: 2026-06-07

Tai lieu nay la muc luc activity diagram cho identity-service. Moi bieu do
duoc tach thanh mot file rieng trong thu muc `docs/activity-diagrams/`.

Sequence diagram chi tiet nam trong `docs/sequence-diagrams/`.

Direct service prefix hien tai: `/api/identity`.

Gateway public path hien tai:

- Auth: `/api/auth/*`
- Profile: `/api/user/*`

## Danh sach bieu do

| STT | Chuc nang | File |
| --- | --- | --- |
| 1 | Dang nhap | `docs/activity-diagrams/login.puml` |
| 2 | Tao tai khoan | `docs/activity-diagrams/register-account.puml` |
| 3 | Dang xuat | `docs/activity-diagrams/logout.puml` |
| 4 | Quen mat khau | `docs/activity-diagrams/forgot-password.puml` |
| 5 | Cap nhat profile | `docs/activity-diagrams/update-profile.puml` |
| 6 | Khóa tài khoản (Admin) | `docs/activity-diagrams/manage-admin-users/manage-admin-users-overview.puml` |
