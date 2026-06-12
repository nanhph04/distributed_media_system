IDENTITY SERVICE API LIST - FRONTEND CONTRACT

Last updated: 2026-06-06

# Muc dich

Tai lieu nay danh cho Frontend/Mobile khi ghep API identity-service qua
api_gateway.

# Nguyen tac ghep API cho FE

1. FE nen goi qua api_gateway, khong goi truc tiep identity-service noi bo.

   Vi du local:
   - Gateway base URL: http://localhost:<GATEWAY_PORT>
   - FE endpoint: http://localhost:<GATEWAY_PORT>/api/auth/login

2. Public path FE dung qua gateway:
   - Auth: /api/auth/\*
   - Profile: /api/user/\*

3. Direct path neu test thang vao identity-service:

   identity-service dang set global prefix: /api/identity
   - Gateway path: /api/auth/login
   - Direct path: /api/identity/auth/login

   - Gateway path: /api/user/users/profile
   - Direct path: /api/identity/user/users/profile

4. Auth token strategy hien tai:
   - accessToken nam trong response body cua login/refresh.
   - refresh_token nam trong httpOnly cookie, do backend set.
   - FE khong doc duoc httpOnly cookie bang JavaScript.
   - Khi goi login/refresh/logout, FE can bat credentials de trinh duyet gui/nhan cookie.

5. HTTP client rule:
   - KHONG duoc su dung axios.
   - FE su dung native fetch hoac wrapper noi bo build tren fetch.
   - Moi request can ghep URL tu gateway base URL + public path ben duoi.

   Vi du wrapper fetch:

   ```ts
   const gatewayBaseUrl = "http://localhost:<GATEWAY_PORT>";

   await fetch(`${gatewayBaseUrl}/api/auth/refresh`, {
     method: "POST",
     credentials: "include",
   });

   await fetch(`${gatewayBaseUrl}/api/auth/session/profile`, {
     method: "GET",
     credentials: "include",
   });
   ```

6. API protected can header:

   Authorization: Bearer <accessToken>

7. Response success chung:

   ```json
   {
     "success": true,
     "code": 200,
     "data": {},
     "mess": "Message"
   }
   ```

8. Response error chung:

   ```json
   {
     "success": false,
     "code": 401,
     "mess": "Invalid access token",
     "data": null,
     "errors": ["Invalid access token"],
     "requestId": "some-request-id",
     "timestamp": "2026-04-17T10:00:00.000Z",
     "path": "/api/auth/change-password"
   }
   ```

9. Validation:

   Backend dung ValidationPipe voi whitelist va forbidNonWhitelisted.
   FE khong nen gui field thua ngoai DTO, neu khong request co the bi 400.

==================================================

1. # AUTH APIs

1.1) POST /api/auth/register

Muc dich:

- Buoc 1 dang ky tai khoan.
- Backend tao account chua verify email, tao OTP, gui OTP qua email.
- API nay CHUA tao accessToken/refreshToken.

Auth:

- Public.
- Khong can Authorization header.

Body:

```json
{
  "email": "user@example.com",
  "password": "123456"
}
```

Field:

- email: string, required, email.
- password: string, required, min 6.

Response:

- HTTP status: 201
- Body:

```json
{
  "success": true,
  "code": 201,
  "mess": "Registration successful",
  "data": {
    "message": "OTP sent to your email. Please verify to complete registration."
  }
}
```

FE flow:

- Sau khi register thanh cong, chuyen user sang man verify-email.
- Chi can giu lai email de gui tiep len verify-email. Khong gui password o
  buoc verify.

  1.2) POST /api/auth/verify-email

Muc dich:

- Buoc 2 xac minh OTP.
- Backend verify account va tao profile sau khi OTP dung.
- API nay hien tai CHUA tra token, FE can goi login sau khi verify thanh cong.

Auth:

- Public.
- Khong can Authorization header.

Body:

```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

Field:

- email: string, required, email.
- otp: string, required, 6 ky tu.

Response:

- HTTP status: 200
- Body:

```json
{
  "success": true,
  "code": 200,
  "mess": "Email verified successfully",
  "data": {
    "message": "Registration successful. Your email has been verified."
  }
}
```

FE flow:

- Sau khi verify thanh cong, goi POST /api/auth/login de lay accessToken va
  set refresh_token cookie.

  1.3) POST /api/auth/login

Muc dich:

- Dang nhap tai khoan da verify email.
- Backend tra accessToken trong body.
- Backend set refresh_token vao httpOnly cookie.

Auth:

- Public.
- Khong can Authorization header.

Body:

```json
{
  "email": "user@example.com",
  "password": "123456"
}
```

Field:

- email: string, required, email.
- password: string, required.

Luu y:

- Controller lay IP tu request bang @Ip().
- FE khong can gui field ip.
- Neu FE gui field ip, backend hien tai khong dung gia tri nay; IP thuc te van
  lay tu request.
- Neu account bi admin khoa voi status `suspended`, API tra 403 voi mess
  `Your account has been suspended`.

Cookie:

- Set-Cookie: refresh_token=<token>
- httpOnly: true
- secure: true
- sameSite: strict
- maxAge: 3 ngay
- path: lay tu AUTH_COOKIE_PATH.
- .env.example de AUTH_COOKIE_PATH=/ trong identity-service.
- Khi di qua api_gateway, gateway rewrite Set-Cookie Path tu /api/identity/auth sang public path /api/auth.

Response:

- HTTP status: 200
- Body KHONG co refreshToken.

```json
{
  "success": true,
  "code": 200,
  "mess": "Login successful",
  "data": {
    "accessToken": "jwt-access-token",
    "expiresIn": 900
  }
}
```

FE flow:

- Luu accessToken o state/memory hoac storage theo policy cua app.
- Khong tim refreshToken trong body.
- Bat withCredentials/credentials include de browser nhan cookie.

  1.4) POST /api/auth/refresh

Muc dich:

- Cap accessToken moi bang refresh_token cookie.
- Backend doc refresh token tu cookie `refresh_token`, khong doc body.
- Backend rotate refresh token va set lai cookie moi.

Auth:

- Public o gateway.
- Khong can Authorization header.

Body:

- Khong gui body.

Cookie request:

- Browser tu gui cookie `refresh_token` neu FE bat credentials.

Response:

- HTTP status: 200
- Set-Cookie: refresh_token=<new-token>
- Body KHONG co refreshToken.

```json
{
  "success": true,
  "code": 200,
  "mess": "Token refreshed successfully",
  "data": {
    "accessToken": "new-jwt-access-token",
    "expiresIn": 900
  }
}
```

FE flow:

- Khi accessToken het han hoac gap 401 do token expired, goi refresh.
- Neu refresh thanh cong, cap nhat accessToken va retry request truoc do.
- Neu refresh that bai, clear local auth state va dieu huong ve login.

  1.5) GET /api/auth/session/profile

Muc dich:

- Cho SSR guard kiem tra session hien tai bang refresh_token httpOnly cookie.
- Lay profile hien tai ma KHONG rotate refresh token.
- API nay KHONG cap accessToken moi va KHONG set lai refresh_token cookie.

Auth:

- Public o gateway.
- Khong can Authorization header.

Body:

- Khong gui body.

Cookie request:

- Browser/SSR fetch gui cookie `refresh_token` neu bat credentials/include cookie.

Response:

- HTTP status: 200
- KHONG co Set-Cookie moi.
- KHONG co accessToken/refreshToken trong body.

```json
{
  "success": true,
  "code": 200,
  "mess": "Session profile fetched successfully",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "role": "user",
    "displayName": "User",
    "avatarUrl": "",
    "bio": "",
    "phone": 0,
    "gender": null,
    "birthday": null,
    "isCreator": false,
    "createdAt": "2026-01-01T00:00:00.000Z",
    "updatedAt": "2026-01-01T00:00:00.000Z"
  }
}
```

FE/SSR flow:

- Dung cho SSR middleware/guard can biet user dang login hay chua.
- Neu success, SSR co the dung profile/role de render hoac redirect.
- Neu fail 401, xem nhu chua co session hop le.
- Khong dung endpoint nay de refresh accessToken; khi can accessToken moi van goi
  `POST /api/auth/refresh`.

  1.5.1) GET /api/auth/session/events

Muc dich:

- Protected SSE endpoint cho realtime session control.
- FE mo stream nay sau login qua gateway path `/api/auth/session/events`.
- Khi admin suspend account hien tai, backend emit `session.revoked` de FE xoa
  accessToken trong localStorage va thoat authenticated UI ngay.

Auth:

- Protected, can `Authorization: Bearer <accessToken>`.

SSE event:

```text
event: session.revoked
data: {"reason":"ACCOUNT_SUSPENDED","message":"Tài khoản đã bị vô hiệu hóa. Vui lòng kiểm tra email để biết lý do.","revokedAt":"2026-05-17T00:00:00.000Z"}
```

Ghi chu:

- SSE la best-effort, FE van phai clear auth state khi gap 401/403.
- Stream chi gui event cua user dang authenticated.

  1.6) POST /api/auth/resend-otp

Muc dich:

- Gui lai OTP cho luong register hoac forgot password.

Auth:

- Public.
- Khong can Authorization header.

Body:

```json
{
  "email": "user@example.com",
  "type": "register|forgot"
}
```

Field:

- email: string, required, email.
- type: "register" | "forgot", required.

Response:

- HTTP status: 200

```json
{
  "success": true,
  "code": 200,
  "mess": "OTP sent successfully",
  "data": {
    "message": "OTP sent successfully"
  }
}
```

Luu y:

- Gia tri `data.message` phu thuoc use case.
- FE nen hien `mess` hoac `data.message`.
- Voi `type = "forgot"`, backend chi resend neu dang co forgot-password OTP
  session con active trong DB. Neu session da het han hoac chua tung request
  forgot-password, API tra 400 voi message
  `Forgot password session expired. Please request a new OTP.`
- Resend OTP bi rate limit theo email: toi da 3 lan trong 1 gio cho tung action
  resend.
- OTP moi se thay the OTP cu trong session hien tai va reset verify attempts ve
  0.

  1.7) POST /api/auth/forgot-password

Muc dich:

- Yeu cau reset mat khau.
- Backend gui OTP reset password qua email.
- API nay khong tao reset link/token rieng; reset password dung email + OTP +
  newPassword.

Auth:

- Public.
- Khong can Authorization header.

Body:

```json
{
  "email": "user@example.com"
}
```

Field:

- email: string, required, email.

Response:

- HTTP status: 200
- Backend luon tra message chung ben duoi cho ca truong hop email khong ton
  tai, de tranh lo thong tin email nao da dang ky.

```json
{
  "success": true,
  "code": 200,
  "mess": "Password reset instructions sent",
  "data": {
    "message": "If the email exists, a reset OTP will be sent"
  }
}
```

Backend behavior:

- Lowercase email truoc khi query DB va tao session/cache key.
- Neu email khong ton tai, backend khong gui mail nhung van tra response
  success/message chung.
- Neu email ton tai:
  - Rate limit action `forgot` theo email: toi da 3 lan trong 1 gio.
  - Generate OTP theo `OTP_LENGTH`, mac dinh 6.
  - Hash OTP va luu vao `auth_otp_sessions` voi `purpose = "forgot"`.
  - Consume forgot-password OTP session cu cua email do neu co.
  - Mirror raw OTP vao Redis best-effort voi TTL bang `OTP_EXPIRY_SECONDS`.
  - Gui OTP qua email.
- PostgreSQL la source of truth cho OTP session. Redis loi khong lam fail
  request.
- OTP het han theo `OTP_EXPIRY_SECONDS`, mac dinh 300 giay.

1.8) POST /api/auth/reset-password

Muc dich:

- Dat lai mat khau bang OTP.

Auth:

- Public.
- Khong can Authorization header.

Body:

```json
{
  "email": "user@example.com",
  "otp": "123456",
  "newPassword": "new-password"
}
```

Field:

- email: string, required, email.
- otp: string, required, 6 chu so. Backend remove whitespace truoc khi validate
  va so sanh.
- newPassword: string, required, min 6.

Response:

- HTTP status: 200

```json
{
  "success": true,
  "code": 200,
  "mess": "Password reset successful",
  "data": {
    "message": "Password reset successfully. Please login with your new password."
  }
}
```

Backend behavior:

- Lowercase email truoc khi query DB.
- Tim account theo email. Neu khong ton tai, tra 404 `User not found`.
- Tim active OTP session trong `auth_otp_sessions` voi `purpose = "forgot"`.
  Neu khong co hoac da het han, tra 400 `OTP expired or not found`.
- So sanh OTP bang hash, khong so sanh raw OTP trong DB.
- Neu OTP sai:
  - Tang `verifyAttempts` trong DB.
  - Sai du 3 lan thi consume OTP session, xoa Redis mirror best-effort, va tra
    429 `Too many attempts. Please try again later.`
  - Chua du 3 lan thi tra 400 `Invalid OTP`.
- Neu OTP dung:
  - Hash `newPassword` bang bcrypt va update account.
  - Revoke tat ca refresh token cua user, buoc user login lai o moi thiet bi.
  - Mark OTP session consumed.
  - Xoa Redis keys `otp:forgot:<email>` va `forgot:<email>` best-effort.

Error cases:

- Validation failed:
  - HTTP: `400`
  - Vi du: email sai format, OTP khong phai 6 chu so, newPassword ngan hon 6.
- Account khong ton tai:
  - HTTP: `404`
  - Message: `User not found`
- OTP khong co hoac het han:
  - HTTP: `400`
  - Message: `OTP expired or not found`
- OTP sai:
  - HTTP: `400`
  - Message: `Invalid OTP`
- OTP sai 3 lan:
  - HTTP: `429`
  - Message: `Too many attempts. Please try again later.`

1.9) POST /api/auth/change-password

Muc dich:

- Doi mat khau khi user da dang nhap.

Auth:

- Protected.
- Required header:

```http
Authorization: Bearer <accessToken>
```

Body:

```json
{
  "oldPassword": "old-password",
  "newPassword": "new-password"
}
```

Field:

- oldPassword: string, required, min 6.
- newPassword: string, required, min 6.

System fields:

- userId lay tu accessToken da decode, FE khong gui userId.

Response:

- HTTP status: 200

```json
{
  "success": true,
  "code": 200,
  "mess": "Password changed successfully",
  "data": {
    "message": "Password changed successfully"
  }
}
```

1.10) POST /api/auth/logout

Muc dich:

- Dang xuat.
- Backend doc refresh_token tu cookie va revoke session.
- Backend clear cookie refresh_token.

Auth:

- Protected.
- Required header:

```http
Authorization: Bearer <accessToken>
```

Body:

- Khong gui body.

Cookie request:

- Browser tu gui cookie `refresh_token` neu FE bat credentials.

Response:

- HTTP status: 200
- Clear-Cookie: refresh_token voi path lay tu AUTH_COOKIE_PATH.

```json
{
  "success": true,
  "code": 200,
  "mess": "Logout successful",
  "data": {
    "message": "Logout successful"
  }
}
```

FE flow:

- Goi logout voi Authorization header va credentials include.
- Sau do clear accessToken local va dieu huong ve login.

# ================================================== 2. PROFILE APIs

2.1) GET /api/user/users/profile

Muc dich:

- Lay ho so cua user hien tai.

Auth:

- Protected.
- Required header:

```http
Authorization: Bearer <accessToken>
```

Body:

- Khong gui body.

System fields:

- userId lay tu accessToken qua CurrentUserId, FE khong gui userId.

Response:

- HTTP status: 200
- Body CO boc ApiResponse.

```json
{
  "success": true,
  "code": 200,
  "mess": "Profile fetched successfully",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "role": "user",
    "displayName": "Nguyen Van A",
    "avatarUrl": "https://cdn.example.com/avatars/user-1.png",
    "bio": "I create short-form media content.",
    "phone": 84901234567,
    "gender": "male",
    "birthday": "1998-05-20T00:00:00.000Z",
    "isCreator": false,
    "createdAt": "2026-04-23T00:00:00.000Z",
    "updatedAt": "2026-04-23T00:00:00.000Z"
  }
}
```

2.2) POST /api/user/users/profile/avatar/upload-url

Muc dich:

- Upload avatar qua identity-service.
- Client khong upload truc tiep len MinIO, nen khong phu thuoc MinIO CORS.
- Backend validate file, upload len MinIO bang internal client, cap nhat
  `profile.avatarUrl`, va xoa avatar object cu neu user da co avatar truoc do.

Auth:

- Protected.
- Required header:

```http
Authorization: Bearer <accessToken>
```

Body:

```http
Content-Type: multipart/form-data
```

Field:

- avatar: file, required.
- Supported MIME type: `image/jpeg`, `image/png`, `image/webp`.
- Max size mac dinh: 5242880 bytes, cau hinh bang
  `MINIO_AVATAR_MAX_FILE_SIZE`.

FE example:

```ts
const formData = new FormData();
formData.append("avatar", file);

await fetch("/api/user/users/profile/avatar/upload-url", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${accessToken}`,
  },
  body: formData,
});
```

Luu y:

- Khong set thu cong header `Content-Type`; browser se tu them multipart
  boundary.
- `avatarUrl` tra ve la public URL da duoc luu vao DB va co the dung truc tiep
  cho `<img src="...">`.
- Neu upload avatar moi thanh cong, backend xoa avatar object cu sau khi save
  profile thanh cong.

Response:

- HTTP status: 200
- Body CO boc ApiResponse.

```json
{
  "success": true,
  "code": 200,
  "mess": "Avatar uploaded successfully",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "displayName": "Nguyen Van A",
    "avatarUrl": "http://localhost:9000/identity-avatars/identity-service/avatars/user-id/550e8400-e29b-41d4-a716-446655440000.png",
    "bio": "I create short-form media content.",
    "phone": 84901234567,
    "gender": "male",
    "birthday": "1998-05-20T00:00:00.000Z",
    "isCreator": false,
    "createdAt": "2026-04-23T00:00:00.000Z",
    "updatedAt": "2026-04-23T00:00:00.000Z"
  }
}
```

Error cases:

- Missing file:
  - HTTP: `400`
  - Message: `Avatar file is required`
- Unsupported MIME type:
  - HTTP: `400`
  - Message: `Unsupported avatar content type`
- File size <= 0:
  - HTTP: `400`
  - Message: `Avatar content length must be a positive integer`
- File qua lon:
  - HTTP: `400`
  - Message: `Avatar file size exceeds the allowed limit`
- Profile khong ton tai:
  - HTTP: `404`
  - Message: `Profile not found`

  2.3) PATCH /api/user/users/profile

Muc dich:

- Cap nhat ho so cua user hien tai.

Auth:

- Protected.
- Required header:

```http
Authorization: Bearer <accessToken>
```

Body:

Tat ca field deu optional, nhung FE chi nen gui field can update.

```json
{
  "displayName": "Nguyen Van A",
  "bio": "I create short-form media content.",
  "phone": 84901234567,
  "gender": "male",
  "birthday": "1998-05-20"
}
```

Field:

- displayName: string, optional, maxLength 100.
- bio: string, optional.
- phone: number, optional, integer, min 1.
- gender: "male" | "women" | "female", optional.
- birthday: string, optional, ISO date/date-time string.

Luu y:

- `avatarUrl` khong con duoc cap nhat qua `PATCH /profile`.
- Avatar duoc doi qua `POST /profile/avatar/upload-url` voi
  `multipart/form-data`.

System fields:

- userId lay tu accessToken qua CurrentUserId, FE khong gui userId.
- role lay tu auth context cua access token, FE khong gui role.
- birthday duoc controller convert tu string sang Date.

Response:

- HTTP status: 200
- Body CO boc ApiResponse.

```json
{
  "success": true,
  "code": 200,
  "mess": "Profile updated successfully",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "displayName": "Nguyen Van A",
    "avatarUrl": "https://cdn.example.com/avatars/user-1.png",
    "bio": "I create short-form media content.",
    "phone": 84901234567,
    "gender": "male",
    "birthday": "1998-05-20T00:00:00.000Z",
    "isCreator": false,
    "createdAt": "2026-04-23T00:00:00.000Z",
    "updatedAt": "2026-04-23T00:00:00.000Z"
  }
}
```

# ================================================== 3. ADMIN APIs

3.1) GET /api/user/admin/users/summary

Muc dich:

- Lay tong quan user cho admin dashboard.
- API nay chi tra du lieu co domain/schema that trong identity-service.

Auth:

- Protected.
- Required header:

```http
Authorization: Bearer <accessToken>
```

System fields:

- userId lay tu accessToken hoac gateway header `x-user-id`.
- role lay tu accessToken hoac gateway header `x-user-role`.
- Chi cho role `admin`; FE khong tu gui role thu cong.

Response:

- HTTP status: 200
- Body CO boc ApiResponse.

```json
{
  "success": true,
  "code": 200,
  "mess": "Admin users summary fetched successfully",
  "data": {
    "totalUsers": 1240000,
    "activeUsers30d": 980000,
    "newUsers30d": 25000,
    "growth30dPercent": 12.4,
    "flaggedUsers": 0,
    "lockedUsers": 120
  }
}
```

Field tinh toan:

- totalUsers: dem account co role `user`.
- activeUsers30d: dem distinct user co refresh token chua revoke, chua het han,
  va created/updated trong 30 ngay gan nhat.
- newUsers30d: dem account role `user` tao trong 30 ngay gan nhat.
- growth30dPercent: so sanh newUsers30d voi 30 ngay truoc do; `null` neu ky
  truoc bang 0.
- flaggedUsers: tam thoi `0` trong v1 vi chua co domain/schema user flag.
- lockedUsers: dem account role `user` co status `suspended`.

Luu y:

- Creator verification API chua co trong v1 vi identity-service chua co
  workflow/table verification that.
- Dashboard nen giu unavailable state cho:
  - GET /api/user/admin/creator-verifications/summary
  - GET /api/user/admin/creator-verifications?status=pending&page=1&limit=5

    3.2) GET /api/user/admin/users

Muc dich:

- Lay danh sach user cho man hinh admin user management.
- Chi tra account role `user`; khong tra account role `admin`.

Auth:

- Protected, chi role `admin`.
- Required header:

```http
Authorization: Bearer <accessToken>
```

Query optional:

- page: number, default 1.
- limit: number, default 20, toi da 100.
- search: string, tim theo email hoac displayName.
- status: "active" | "suspended".
- emailVerified: boolean.
- isCreator: boolean.
- createdFrom: ISO date/date-time string.
- createdTo: ISO date/date-time string.
- sortBy: "createdAt" | "updatedAt" | "email", default "createdAt".
- sortOrder: "asc" | "desc", default "desc".

Response:

- HTTP status: 200
- Body CO boc ApiResponse va pagination.

```json
{
  "success": true,
  "code": 200,
  "mess": "Admin users fetched successfully",
  "data": [
    {
      "userId": "user-id",
      "email": "user@example.com",
      "role": "user",
      "status": "active",
      "isEmailVerified": true,
      "displayName": "Nguyen Van A",
      "avatarUrl": "",
      "isCreator": false,
      "createdAt": "2026-01-01T00:00:00.000Z",
      "updatedAt": "2026-01-02T00:00:00.000Z",
      "lastActiveAt": "2026-01-03T00:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1,
    "totalPages": 1
  }
}
```

3.3) GET /api/user/admin/users/:userId

Muc dich:

- Lay chi tiet account, profile va session summary cua mot user.
- Chi danh cho admin dashboard/detail drawer.

Auth:

- Protected, chi role `admin`.

Response:

- HTTP status: 200

```json
{
  "success": true,
  "code": 200,
  "mess": "Admin user detail fetched successfully",
  "data": {
    "userId": "user-id",
    "email": "user@example.com",
    "role": "user",
    "status": "active",
    "isEmailVerified": true,
    "createdAt": "2026-01-01T00:00:00.000Z",
    "updatedAt": "2026-01-02T00:00:00.000Z",
    "profile": {
      "userId": "user-id",
      "email": "user@example.com",
      "displayName": "Nguyen Van A",
      "avatarUrl": "",
      "bio": "",
      "phone": 0,
      "gender": null,
      "birthday": null,
      "isCreator": false,
      "createdAt": "2026-01-01T00:00:00.000Z",
      "updatedAt": "2026-01-02T00:00:00.000Z"
    },
    "sessions": {
      "activeSessionCount": 1,
      "lastActiveAt": "2026-01-03T00:00:00.000Z"
    }
  }
}
```

3.4) PATCH /api/user/admin/users/:userId/status

Muc dich:

- Admin doi trang thai account user.
- Neu status moi la `suspended`, backend revoke tat ca refresh token cua user de
  dang xuat khoi moi session.
- Sau khi cap nhat thanh cong, backend luu Kafka event `user.status.changed`
  vao outbox de media-service va finance-service dong bo quyen hoat dong cua
  user. Outbox worker se publish event len Kafka va retry neu Kafka tam loi.
- API nay khong cho thao tac account role `admin`.

Auth:

- Protected, chi role `admin`.

Body:

```json
{
  "status": "suspended",
  "reason": "Violation of platform policy"
}
```

Field:

- status: "active" | "suspended", required.
- reason: string, required khi status la `suspended`, maxLength 500.

Response:

- HTTP status: 200
- Body tra ve detail moi nhat, cung shape voi GET detail.

Side effects:

- Khi status la `suspended`, backend gui email cho user voi ly do bi khoa.
- Khi status la `suspended`, backend revoke refresh tokens va emit SSE
  `session.revoked` cho user dang online.
- Backend publish Kafka event `user.status.changed` thong qua outbox; payload co
  field `data.reason`.

# ================================================== 4. PUBLIC ROUTE SUMMARY FOR FE

Public, khong can Authorization:

- POST /api/auth/register
- POST /api/auth/verify-email
- POST /api/auth/login
- POST /api/auth/refresh
- GET /api/auth/session/profile
- POST /api/auth/resend-otp
- POST /api/auth/forgot-password
- POST /api/auth/reset-password

Protected, can Authorization Bearer:

- POST /api/auth/change-password
- POST /api/auth/logout
- GET /api/auth/session/events
- GET /api/user/users/profile
- POST /api/user/users/profile/avatar/upload-url
- PATCH /api/user/users/profile
- GET /api/user/admin/users/summary
- GET /api/user/admin/users
- GET /api/user/admin/users/:userId
- PATCH /api/user/admin/users/:userId/status

# ================================================== 5. FRONTEND AUTH FLOW GOI Y

Register flow:

1. POST /api/auth/register
2. User nhap OTP tu email
3. POST /api/auth/verify-email voi email + otp
4. POST /api/auth/login
5. Luu accessToken tu body, refresh_token tu cookie se do browser quan ly

Login flow:

1. POST /api/auth/login voi credentials include
2. Luu data.accessToken
3. Goi protected API voi Authorization Bearer

Refresh flow:

1. POST /api/auth/refresh voi credentials include, khong body
2. Neu success, cap nhat data.accessToken
3. Neu fail, clear auth state va ve login

Forgot password flow:

1. POST /api/auth/forgot-password voi email.
2. Backend gui OTP neu email ton tai; FE van hien message chung va chuyen user
   sang man nhap OTP + mat khau moi.
3. User nhap OTP tu email va mat khau moi.
4. POST /api/auth/reset-password voi email + otp + newPassword.
5. Neu success, clear auth state local neu co va dieu huong ve login.
6. User dang nhap lai bang mat khau moi; refresh token cu da bi revoke.
7. Neu can gui lai OTP, goi POST /api/auth/resend-otp voi `type = "forgot"`
   khi forgot session con active.

SSR session guard flow:

1. GET /api/auth/session/profile voi credentials include/cookie forwarded tu SSR
   request
2. Neu success, dung data profile + role cho SSR guard/render
3. Neu 401, redirect ve login hoac render unauthenticated state
4. Khong cap nhat accessToken va khong mong doi Set-Cookie moi

Logout flow:

1. POST /api/auth/logout voi Authorization Bearer va credentials include
2. Clear accessToken local
