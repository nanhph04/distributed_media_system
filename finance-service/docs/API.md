# Finance Service API

Tai lieu nay viet theo goc nhin FE/BFF: goi API nao, gui gi, nhan gi, va backend dang xu ly ra sao theo code hien tai.

## 1. Tong quan

- Base path: `/api`
- Swagger: `/api/docs`
- Validation global:
  - `transform: true`
  - `whitelist: true`
  - `forbidNonWhitelisted: true`
- Hau het route di qua internal gateway guard.
- 4 route duoc skip internal gateway guard:
  - `GET /api`
  - `GET /api/health`
  - `POST /api/deposits/webhooks/payos`
  - `POST /api/deposits/:depositId/webhook/success`

## 2. Response format

Success response luon duoc boc theo format:

```json
{
  "success": true,
  "code": 200,
  "data": {},
  "mess": "optional"
}
```

Voi endpoint tao moi tra HTTP 201, envelope se co `code: 201`.

Error response:

```json
{
  "success": false,
  "code": 400,
  "mess": "Error message",
  "data": null,
  "errors": ["Error message"],
  "requestId": "1744712345-abc1234",
  "timestamp": "2026-04-15T10:19:05.123Z",
  "path": "/api/..."
}
```

## 3. Header chung

Phan lon API dang nam sau internal gateway, nen request thuong can:

- `x-internal-secret`: bat buoc cho moi route khong duoc skip guard
- `x-user-id`: bat buoc cho route user-specific va admin route
- `x-user-role`: bat buoc cho route admin, gia tri backend dang check la `admin`
- `idempotency-key`: bat buoc cho `POST /api/internal/payments/charge`
- `x-request-id`: optional cho `POST /api/internal/payments/charge`, dung lam trace id cho integration event

Webhook internal deposit success dung header rieng:

- `x-webhook-timestamp`
- `x-webhook-signature`

Neu FE goi thong qua API Gateway/BFF thi gateway thuong se tu gan `x-internal-secret`, `x-user-id`, `x-user-role`.

## 3.1 Health Check

### GET `/api`

- Muc dich: kiem tra finance-service dang chay
- Route dac biet:
  - skip internal gateway guard
- Input:
  - khong co body/query/path param
- Output `data`:

```json
"Hello World!"
```

### GET `/api/health`

- Muc dich: health check nhe cho gateway/load balancer hoac man hinh admin hien thi service con song.
- Route dac biet:
  - skip internal gateway guard
- Input:
  - khong co body/query/path param
- Output `data`:

```json
{
  "status": "ok",
  "service": "finance-service",
  "timestamp": "2026-05-28T00:19:26.000Z"
}
```

- Ghi chu:
  - endpoint nay chi bao Nest app con response HTTP duoc.
  - hien chua kiem tra DB/Kafka/Redis deep health.

## 4. Wallet APIs

### 4.1 GET `/api/wallets/me`

- Muc dich: lay vi cua user dang dang nhap
- Headers:
  - `x-user-id` bat buoc
- Input:
  - khong co body
- Output `data`:

```json
{
  "id": "wallet-id",
  "userId": "user-id",
  "type": "user",
  "balance": 1000,
  "frozenBalance": 200,
  "status": "active"
}
```

- Backend xu ly:
  - doc `x-user-id`
  - tim vi theo `userId`
  - neu khong co vi thi tra `404 Wallet not found`

### 4.2 GET `/api/wallets/user/:userId`

- Muc dich: lay vi theo `userId`
- Input:
  - path param: `userId`
- Output: giong `GET /api/wallets/me`
- Backend xu ly:
  - tim vi theo `userId`
  - khong check ownership
  - neu khong co vi thi tra `404`

### 4.3 GET `/api/studio/wallet/me`

- Muc dich: lay wallet read model cho Creator Studio.
- Dung khi FE can hien thi so du vi studio hien tai cua creator/channel owner.
- Gateway public contract:
  - `GET /api/studio/wallet/me`
- Headers:
  - `x-user-id` bat buoc
- Input:
  - khong co body/query/path param
- Output `data`:

```json
{
  "id": "wallet-id",
  "userId": "user-id",
  "type": "STUDIO",
  "balance": 1000,
  "frozenBalance": 200,
  "status": "ACTIVE",
  "createdAt": "2026-05-15T00:00:00.000Z",
  "updatedAt": "2026-05-15T00:00:00.000Z",
  "totalEarnings": 1200,
  "videoCount": 0,
  "totalViews": 0,
  "totalRevenue": 1200,
  "revenueThisMonth": 300,
  "subscribersCount": 0,
  "currency": "AC"
}
```

- Backend xu ly:
  - tim wallet theo `x-user-id`
  - tinh `totalEarnings` tu transaction `channel_revenue`
  - tinh `revenueThisMonth` tu transaction `channel_revenue` trong thang hien tai
  - `balance`: coin da kha dung, co the rut/chi tieu theo rule cua wallet
  - `frozenBalance`: coin revenue dang bi giu trong vi, thuong la phan chua release ve available balance
  - cac chi so media chua co trong finance DB (`videoCount`, `totalViews`, `subscribersCount`) tam thoi tra `0`
  - neu khong co wallet thi tra `404 Wallet not found`
- Luu y:
  - Neu can dung duy nhat so coin revenue dang pending cua channel/studio, uu tien
    `GET /api/studio/earnings/summary` va doc `pendingEarnings`.

### 4.4 GET `/api/studio/wallet/stats`

- Muc dich: lay thong ke tong hop cho Studio Wallet dashboard.
- Dung cho card/tong quan vi, bao gom available balance va withdrawal dang cho xu ly.
- Gateway public contract:
  - `GET /api/studio/wallet/stats`
- Headers:
  - `x-user-id` bat buoc
- Input:
  - khong co body/query/path param
- Output `data`:

```json
{
  "totalBalance": 1200,
  "availableBalance": 1000,
  "pendingPayouts": 100,
  "totalWithdrawn": 500,
  "avgRevenuePerVideo": 0,
  "totalViews": 0,
  "totalLikes": 0,
  "monthlyEarnings": 300,
  "monthlyGrowth": 25,
  "topPerformingVideo": null
}
```

- Backend xu ly:
  - `totalBalance = balance + frozenBalance`
  - `availableBalance = balance`
  - `pendingPayouts`: tong withdrawal dang `pending`
  - `totalWithdrawn`: tong withdrawal `completed`
  - `monthlyEarnings`: tong transaction `channel_revenue` trong thang hien tai
  - `monthlyGrowth`: so sanh `monthlyEarnings` voi thang truoc
  - cac chi so media/top video chua co metadata media trong finance DB nen tam thoi tra `0`/`null`
- Luu y:
  - `pendingPayouts` la coin dang nam trong luong rut tien, khong phai revenue pending cua channel.
  - De hien thi coin revenue dang cho release cua channel/studio, dung `pendingEarnings`
    tu `GET /api/studio/earnings/summary`.

## 4.5 Studio Earnings GET APIs

### 4.5.1 GET `/api/studio/earnings/summary`

- Muc dich: lay tong quan earnings cho Creator Studio.
- Dung khi FE can so coin revenue dang pending cua channel/studio.
- Gateway public contract:
  - `GET /api/studio/earnings/summary`
- Headers:
  - `x-user-id` bat buoc
- Query:
  - `startDate` optional, ISO date/date-time
  - `endDate` optional, ISO date/date-time
  - `period` neu FE gui thi backend hien bo qua; dung `startDate/endDate` khi can range cu the
- Output `data`:

```json
{
  "totalEarnings": 1200,
  "totalViews": 0,
  "totalWatchTime": 0,
  "totalVideos": 0,
  "pendingEarnings": 200,
  "confirmedEarnings": 1000,
  "paidEarnings": 500,
  "averageEarningsPerVideo": 0,
  "averageEarningsPerView": 0,
  "growth": {
    "percentage": 25,
    "comparedToPrevious": 960,
    "absoluteDifference": 240
  },
  "nextPayoutDate": "2026-05-20T00:00:00.000Z",
  "minimumPayoutThreshold": 100
}
```

- Backend xu ly:
  - `totalEarnings`: tong transaction `channel_revenue`
  - `pendingEarnings`: tong settlement `pending`
  - `confirmedEarnings`: tong settlement `released`
  - `paidEarnings`: tong withdrawal `completed`
  - `nextPayoutDate`: `MIN(releaseAfter)` cua settlement `pending`
  - cac chi so view/watch/video tam thoi tra `0`
- Field quan trong:
  - `pendingEarnings`: so coin revenue cua channel/studio dang pending, chua release ve available balance.
  - `confirmedEarnings`: revenue da release.
  - `paidEarnings`: coin da rut thanh cong.
- Luu y:
  - `pendingEarnings` lay tu bang `payment_revenue_settlements` voi status `pending`.
  - Neu co `startDate/endDate`, `totalEarnings` duoc filter theo range; cac tong settlement
    `pendingEarnings`/`confirmedEarnings` hien tai la tong theo wallet.

### 4.5.2 GET `/api/studio/earnings/monthly`

- Muc dich: lay earnings trong mot thang.
- Gateway public contract:
  - `GET /api/studio/earnings/monthly?year=2026&month=5`
- Headers:
  - `x-user-id` bat buoc
- Query:
  - `year` optional, mac dinh nam hien tai
  - `month` optional, mac dinh thang hien tai, gia tri 1-12
- Output `data`:

```json
{
  "month": "05",
  "year": 2026,
  "earnings": 300,
  "views": 0,
  "watchTime": 0,
  "videoCount": 0,
  "payoutAmount": 1000,
  "status": "confirmed",
  "growth": 25
}
```

- Backend xu ly:
  - tinh `earnings` tu transaction `channel_revenue` trong thang query
  - `growth`: so sanh voi thang lien truoc
  - `payoutAmount`: balance kha dung hien tai
  - `status`: `confirmed` neu balance >= nguong payout toi thieu, nguoc lai `pending`

### 4.5.3 GET `/api/studio/earnings/top-videos`

- Muc dich: lay top video theo revenue tu settlement.
- Gateway public contract:
  - `GET /api/studio/earnings/top-videos?period=monthly&limit=3`
- Headers:
  - `x-user-id` bat buoc
- Query:
  - `limit` optional, mac dinh `3`, min `1`, max `20`
  - `startDate` optional, ISO date/date-time
  - `endDate` optional, ISO date/date-time
  - `period` hien FE co gui nhung backend chua dung; dung `startDate/endDate` khi can range cu the
- Output `data`:

```json
[
  {
    "id": "video-id",
    "videoId": "video-id",
    "videoTitle": "Video",
    "categoryId": "",
    "categoryName": "",
    "views": 0,
    "watchTime": 0,
    "revenue": 300,
    "estimatedRevenue": 300,
    "currency": "AC",
    "status": "confirmed",
    "period": {
      "startDate": "2026-05-01T00:00:00.000Z",
      "endDate": "2026-05-31T23:59:59.999Z"
    },
    "metrics": {
      "completionRate": 0,
      "engagementRate": 0,
      "averageViewDuration": 0
    }
  }
]
```

- Backend xu ly:
  - group `payment_revenue_settlements` theo `serviceId` voi `serviceType = video`
  - loc theo `createdAt` neu co `startDate/endDate`
  - sort theo tong `amount` giam dan
  - finance-service chua co title/category/view/watchtime tu media-service, nen metadata media tam thoi la placeholder/`0`

### 4.5.4 GET `/api/studio/earnings/history`

- Muc dich: lay lich su earnings cua creator tu transaction `channel_revenue`.
- Headers:
  - `x-user-id` bat buoc
- Query:
  - `status` optional, loc theo transaction status
  - `page` optional, mac dinh `1`
  - `limit` optional, mac dinh `20`, toi da `100`
  - `startDate` optional, ISO date/date-time
  - `endDate` optional, ISO date/date-time
- Output `data`:

```json
{
  "items": [
    {
      "id": "transaction-id",
      "amount": 80,
      "status": "pending",
      "referenceId": "video-id",
      "description": "Creator revenue",
      "createdAt": "2026-05-15T00:00:00.000Z"
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

## 5. Transaction APIs

### 5.1 GET `/api/transactions/me`

- Muc dich: lay danh sach giao dich do user hien tai khoi tao
- Headers:
  - `x-user-id` bat buoc
- Output `data`: mang transaction
- Luu y voi transaction nap tien:
  - `type = deposit`
  - `assetType = coin`
  - `amount` la so coin duoc cong vao vi
  - so tien user da thanh toan nam trong `metadata.moneyAmount`

```json
[
  {
    "id": "txn-1",
    "type": "deposit",
    "assetType": "coin",
    "amount": 550,
    "status": "completed",
    "fromWalletId": null,
    "toWalletId": "wallet-1",
    "initiatedByUserId": "user-1",
    "referenceId": "deposit-1",
    "description": "Completed PayOS deposit TOPUP_50K",
    "failureReason": null,
    "metadata": {
      "packageId": "package-1",
      "packageCode": "TOPUP_50K",
      "packageName": "Goi 50.000 VND",
      "moneyAmount": 50000,
      "baseCoinAmount": 500,
      "bonusCoinAmount": 50,
      "totalCoinAmount": 550,
      "gateway": "payos",
      "gatewayTransactionId": "PAYOS-REF-1",
      "paymentCode": "123456789",
      "payosOrderCode": 123456789,
      "payosReference": "PAYOS-REF-1",
      "paymentLinkId": "link-123456789"
    },
    "completedAt": "2026-05-06T10:00:00.000Z",
    "failedAt": null,
    "cancelledAt": null,
    "createdAt": "2026-05-06T09:55:00.000Z",
    "updatedAt": "2026-05-06T10:00:00.000Z"
  }
]
```

- Backend xu ly:
  - lay `x-user-id`
  - query transaction theo `initiatedByUserId`

### 5.2 GET `/api/transactions/reference/:referenceId`

- Muc dich: lay tat ca transaction lien quan den mot `referenceId`
- Input:
  - path param: `referenceId`
- Output: mang transaction
- Backend xu ly:
  - query theo `referenceId`
  - khong check ownership
- Goi y cho FE:
  - dung khi can xem cac but toan sinh ra tu cung 1 nghiep vu nhu payment, deposit, withdrawal

### 5.3 GET `/api/transactions/:transactionId`

- Muc dich: lay chi tiet 1 transaction
- Headers:
  - `x-user-id` bat buoc
- Input:
  - path param: `transactionId`
- Output: 1 transaction
- Backend xu ly:
  - tim transaction theo id
  - kiem tra `initiatedByUserId === x-user-id`
  - neu khong dung owner thi tra `403 You do not own this transaction`

## 6. Deposit APIs

### 6.1 GET `/api/deposits/packages`

- Muc dich: lay danh sach goi nap dang active de FE render man nap tien
- Headers:
  - `x-internal-secret` bat buoc neu goi truc tiep vao service
- Input:
  - khong co body
- Output `data`: mang package

```json
[
  {
    "id": "package-1",
    "code": "TOPUP_500K",
    "name": "Goi 500.000 VND",
    "moneyAmount": 500000,
    "baseCoinAmount": 5000,
    "bonusCoinAmount": 500,
    "totalCoinAmount": 5500,
    "sortOrder": 1,
    "description": "Tang coin"
  }
]
```

- Backend xu ly:
  - chi lay package `isActive = true`
  - khong can login user

### 6.2 POST `/api/deposits`

- Muc dich: tao yeu cau nap tien theo package
- Headers:
  - `x-user-id` bat buoc
- Input body:

```json
{
  "packageId": "package-id"
}
```

- Output `data`:

```json
{
  "id": "deposit-id",
  "walletId": "wallet-id",
  "userId": "user-id",
  "packageId": "package-id",
  "packageCode": "TOPUP_500K",
  "packageName": "Goi 500.000 VND",
  "moneyAmount": 500000,
  "baseCoinAmount": 5000,
  "bonusCoinAmount": 500,
  "totalCoinAmount": 5500,
  "gateway": "payos",
  "status": "pending",
  "paymentCode": "123456789",
  "transferContent": "optional",
  "gatewayTransactionId": null,
  "checkoutUrl": "https://pay.payos.vn/...",
  "description": "Top up via package TOPUP_500K",
  "requestedAt": "2026-05-06T10:00:00.000Z",
  "completedAt": null,
  "failedAt": null,
  "cancelledAt": null
}
```

- Backend xu ly:
  - kiem tra package co ton tai va dang active khong
  - tim vi user, neu chua co thi tu tao vi
  - tao ban ghi deposit `pending`
  - sinh `paymentCode` duy nhat
  - goi PayOS de tao payment link
  - tra `checkoutUrl` cho FE de redirect/mo link thanh toan

### 6.3 POST `/api/deposits/:depositId/webhook/success`

- Muc dich: webhook noi bo de xac nhan deposit thanh cong
- FE: khong goi API nay
- Route dac biet:
  - skip internal gateway guard
  - khong can `x-internal-secret`
  - verify HMAC qua `x-webhook-timestamp` va `x-webhook-signature`
- Input body:

```json
{
  "gatewayTransactionId": "VNPAY_TX_123456",
  "gateway": "vnpay",
  "moneyAmount": 10000,
  "description": "Payment confirmed by webhook"
}
```

- Output: `DepositResponseDto`
- Backend xu ly:
  - verify HMAC signature
  - kiem tra gateway, so tien, transaction id co match deposit khong
  - cong coin vao vi
  - tao transaction `deposit` dang ledger coin: `assetType=coin`, `amount=totalCoinAmount`
  - luu so tien da thanh toan trong transaction metadata: `moneyAmount`
  - update deposit sang `completed`

### 6.4 POST `/api/deposits/webhooks/payos`

- Muc dich: webhook PayOS callback truc tiep
- FE: khong goi API nay
- Route dac biet:
  - skip internal gateway guard
  - khong can `x-internal-secret`
- Input body:

```json
{
  "code": "00",
  "desc": "success",
  "success": true,
  "data": {},
  "signature": "sha256-signature"
}
```

- Output `data`:

```json
{
  "received": true,
  "status": "completed",
  "depositId": "deposit-id"
}
```

- Backend xu ly:
  - verify chu ky webhook PayOS trong payment gateway service
  - neu `success !== true` thi khong throw, tra:

```json
{
  "received": true,
  "status": "ignored"
}
```

- chi xu ly complete khi:
  - `success === true`
  - `code === "00"`
  - neu co `dataCode` thi `dataCode === "00"`
- tim deposit qua `paymentCode`/`orderCode`
- check gateway va amount
- complete deposit se cong `totalCoinAmount` vao vi va tao transaction `deposit` voi `assetType=coin`
- transaction metadata luu `moneyAmount`, `baseCoinAmount`, `bonusCoinAmount`, `totalCoinAmount`
- neu webhook duplicate cho deposit da `completed` thi tra thanh cong va khong cong coin lan 2

### 6.5 GET `/api/deposits/admin/packages`

- Muc dich: admin xem tat ca package ke ca inactive
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Output `data`: mang admin package, co them:
  - `isActive`
  - `createdAt`
  - `updatedAt`

- Backend xu ly:
  - check role phai la `admin`
  - neu khong phai admin tra `403 Admin role is required`

### 6.6 POST `/api/deposits/admin/packages`

- Muc dich: admin tao package nap
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input body:

```json
{
  "code": "TOPUP_500K",
  "name": "Goi 500.000 VND",
  "moneyAmount": 500000,
  "baseCoinAmount": 5000,
  "bonusCoinAmount": 500,
  "sortOrder": 4,
  "isActive": true,
  "description": "Tang them coin cho nguoi dung nap lon"
}
```

- Output: admin package object
- Backend xu ly:
  - check admin role
  - check `code` chua ton tai
  - tao package moi

### 6.7 PATCH `/api/deposits/admin/packages/:packageId`

- Muc dich: admin cap nhat package
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input:
  - path param: `packageId`
  - body: cac field can update, tat ca optional

```json
{
  "code": "TOPUP_500K_NEW",
  "name": "Goi 500.000 VND moi",
  "moneyAmount": 500000,
  "baseCoinAmount": 5000,
  "bonusCoinAmount": 500,
  "sortOrder": 4,
  "isActive": false,
  "description": "Tam khoa de doi campaign moi"
}
```

- Output: admin package object sau update
- Backend xu ly:
  - check admin role
  - tim package theo id
  - neu doi `code` thi check `code` chua ton tai tren package khac
  - update field truyen len

### 6.8 POST `/api/deposits/admin/:depositId/reconcile`

- Muc dich: admin ep backend kiem tra lai trang thai payment PayOS cho 1 deposit pending
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input:
  - path param: `depositId`
- Output: `DepositResponseDto`
- Backend xu ly:
  - check admin role
  - lock chong reconcile trung
  - neu deposit da `completed` hoac `failed` thi tra trang thai hien tai
  - neu PayOS bao `PAID` thi complete deposit, cong coin
  - neu PayOS bao `EXPIRED` hoac `CANCELLED` thi mark deposit failed

## 7. Withdrawal APIs

### 7.1 POST `/api/withdrawals`

- Muc dich: user tao yeu cau rut coin ra tien mat
- Headers:
  - `x-user-id` bat buoc
- Input body:

```json
{
  "coinAmount": 100,
  "bankInfo": {
    "bankCode": "VCB",
    "bankName": "Vietcombank",
    "accountNumber": "0123456789",
    "accountHolderName": "Nguyen Van A",
    "qrCode": "https://example.com/qr.png"
  },
  "description": "Rut tien thang 4"
}
```

- Output `data`:

```json
{
  "id": "withdrawal-id",
  "walletId": "wallet-id",
  "userId": "user-id",
  "coinAmount": 100,
  "grossMoneyAmount": 10000,
  "feePercent": 5,
  "feeAmount": 500,
  "moneyAmount": 9500,
  "exchangeRate": 100,
  "bankInfo": {
    "bankCode": "VCB",
    "bankName": "Vietcombank",
    "accountNumber": "0123456789",
    "accountHolderName": "Nguyen Van A",
    "qrCode": "https://example.com/qr.png"
  },
  "status": "pending",
  "adminNote": null,
  "processedByAdminId": null,
  "transferReference": null,
  "description": "Rut tien thang 4",
  "rejectionReason": null,
  "requestedAt": "2026-05-06T10:00:00.000Z",
  "completedAt": null,
  "rejectedAt": null,
  "cancelledAt": null
}
```

- Backend xu ly:
  - check vi user ton tai
  - lay `exchangeRate` tu server config `WITHDRAWAL_EXCHANGE_RATE`
  - lay `feePercent` tu server config `WITHDRAWAL_FEE_PERCENT`
  - tinh `grossMoneyAmount = coinAmount * exchangeRate`
  - tinh `feeAmount = floor(grossMoneyAmount * feePercent / 100)`
  - tinh `moneyAmount = grossMoneyAmount - feeAmount`
  - reject neu user dang co payout hold do channel bi suspended
  - tru coin kha dung khoi vi theo co che freeze/rut
  - tao withdrawal `pending`
  - tao transaction `withdrawal`

- FE luu y:
  - khong gui `grossMoneyAmount`, `feePercent`, `feeAmount`, `moneyAmount` hoac `exchangeRate`
  - request se bi reject neu gui field ngoai DTO do `forbidNonWhitelisted`

### 7.2 GET `/api/withdrawals/me`

- Muc dich: user xem danh sach lenh rut cua minh
- Headers:
  - `x-user-id` bat buoc
- Query:
  - `status` optional, mot trong: `pending`, `completed`, `rejected`, `cancelled`
  - `page` optional, mac dinh `1`
  - `limit` optional, mac dinh `20`, toi da `100`
  - `startDate` optional, ISO date/date-time, loc theo `requestedAt >= startDate`
  - `endDate` optional, ISO date/date-time, loc theo `requestedAt <= endDate`
- Output: object co `items` va `pagination`
- Backend xu ly:
  - lay theo `userId`
  - loc theo query neu co
  - fallback `page/limit` neu query khong hop le

```json
{
  "items": [
    {
      "id": "withdrawal-id",
      "walletId": "wallet-id",
      "userId": "user-id",
      "coinAmount": 100,
      "grossMoneyAmount": 10000,
      "feePercent": 5,
      "feeAmount": 500,
      "moneyAmount": 9500,
      "exchangeRate": 100,
      "bankInfo": {
        "bankCode": "VCB",
        "bankName": "Vietcombank",
        "accountNumber": "0123456789",
        "accountHolderName": "Nguyen Van A"
      },
      "status": "pending",
      "requestedAt": "2026-05-06T10:00:00.000Z"
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

### 7.2.1 GET `/api/withdrawals/summary`

- Muc dich: user xem tong quan withdrawal cua minh.
- Headers:
  - `x-user-id` bat buoc
- Output `data`:

```json
{
  "pendingCount": 1,
  "pendingCoinAmount": 100,
  "pendingMoneyAmount": 10000,
  "completedCount": 2,
  "completedCoinAmount": 300,
  "completedMoneyAmount": 30000,
  "rejectedCount": 0,
  "cancelledCount": 1
}
```

### 7.2.2 GET `/api/withdrawals/amount-limits`

- Muc dich: FE lay rule hien tai de render form rut tien.
- Headers:
  - `x-user-id` bat buoc
- Output `data`:

```json
{
  "minCoinAmount": 1,
  "maxCoinAmount": 1000,
  "availableBalance": 1000,
  "exchangeRate": 100,
  "feePercent": 5,
  "minFeeAmount": 5,
  "maxFeeAmount": 5000,
  "minGrossMoneyAmount": 100,
  "maxGrossMoneyAmount": 100000,
  "minMoneyAmount": 95,
  "maxMoneyAmount": 95000,
  "currency": "VND"
}
```

- Backend xu ly:
  - lay wallet user hien tai
  - `maxCoinAmount = availableBalance`
  - `exchangeRate` lay tu config `WITHDRAWAL_EXCHANGE_RATE`
  - `feePercent` lay tu config `WITHDRAWAL_FEE_PERCENT`
  - `moneyAmount` la tien user thuc nhan sau phi
  - khong nhan `grossMoneyAmount`, `feePercent`, `feeAmount`, `moneyAmount`, `exchangeRate`, hay `methodId` tu FE

### 7.3 GET `/api/withdrawals/:withdrawalId`

- Muc dich: user xem chi tiet lenh rut
- Headers:
  - `x-user-id` bat buoc
- Input:
  - path param: `withdrawalId`
- Output: 1 withdrawal
- Backend xu ly:
  - tim withdrawal
  - chi owner moi xem duoc

### 7.4 POST `/api/withdrawals/:withdrawalId/cancel`

- Muc dich: user huy lenh rut
- Headers:
  - `x-user-id` bat buoc
- Input:
  - path param: `withdrawalId`
- Output: withdrawal sau khi huy
- Backend xu ly:
  - check owner
  - doi status withdrawal sang `cancelled`
  - hoan coin ve vi
  - update transaction sang cancelled

### 7.5 POST `/api/withdrawals/:withdrawalId/reject`

- Muc dich: admin tu choi lenh rut
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input body:

```json
{
  "reason": "Thong tin tai khoan khong hop le",
  "adminNote": "Can cap nhat lai ten chu tai khoan"
}
```

- Output: withdrawal sau khi reject
- Backend xu ly:
  - check role `admin`
  - set status `rejected`
  - tra coin ve vi
  - update transaction sang failed

### 7.6 POST `/api/withdrawals/:withdrawalId/complete`

- Muc dich: admin xac nhan da chuyen khoan xong
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input body:

```json
{
  "transferReference": "BANK-TXN-123",
  "adminNote": "Da chuyen khoan thanh cong"
}
```

- Output: withdrawal sau khi complete
- Backend xu ly:
  - check role `admin`
  - set status `completed`
  - finalize coin deduction trong vi
  - update transaction completed

### 7.8 GET `/api/withdrawals/admin/summary`

- Muc dich: admin lay so lieu tong quan de render dashboard xu ly rut tien.
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input:
  - khong co body/query/path param
- Output `data`:

```json
{
  "pendingCount": 3,
  "pendingCoinAmount": 1200,
  "pendingMoneyAmount": 120000,
  "completed30dMoneyAmount": 500000
}
```

- Backend xu ly:
  - check role `admin`
  - dem/tong hop withdrawal theo cac status can hien thi
  - `completed30dMoneyAmount`: tong tien withdrawal `completed` trong 30 ngay gan nhat

### 7.9 GET `/api/withdrawals/admin`

- Muc dich: admin lay danh sach lenh rut co phan trang va loc theo status.
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Query:
  - `status` optional, mot trong: `pending`, `completed`, `rejected`, `cancelled`
  - `userId` optional, loc withdrawal theo user
  - `page` optional, mac dinh `1`, neu khong phai so nguyen duong thi backend fallback ve `1`
  - `limit` optional, mac dinh `20`, toi da `100`, neu khong phai so nguyen duong thi backend fallback ve `20`
  - `startDate` optional, ISO date/date-time, loc theo `requestedAt >= startDate`
  - `endDate` optional, ISO date/date-time, loc theo `requestedAt <= endDate`
- Output `data`:

```json
{
  "items": [
    {
      "id": "withdrawal-id",
      "walletId": "wallet-id",
      "userId": "user-id",
      "coinAmount": 100,
      "grossMoneyAmount": 10000,
      "feePercent": 5,
      "feeAmount": 500,
      "moneyAmount": 9500,
      "exchangeRate": 100,
      "bankInfo": {
        "bankCode": "VCB",
        "bankName": "Vietcombank",
        "accountNumber": "0123456789",
        "accountHolderName": "Nguyen Van A",
        "qrCode": "https://example.com/qr.png"
      },
      "status": "pending",
      "adminNote": null,
      "processedByAdminId": null,
      "transferReference": null,
      "description": "Rut tien thang 4",
      "rejectionReason": null,
      "requestedAt": "2026-05-06T10:00:00.000Z",
      "completedAt": null,
      "rejectedAt": null,
      "cancelledAt": null
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

- Backend xu ly:
  - check role `admin`
  - validate `status` neu co; status khong hop le tra `400 Invalid withdrawal status`
  - query withdrawal theo status/user/date range neu co
  - tra `items` va `pagination`

### 7.10 GET `/api/withdrawals/admin/:withdrawalId`

- Muc dich: admin xem chi tiet bat ky lenh rut.
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Input:
  - path param: `withdrawalId`
- Output: 1 withdrawal
- Backend xu ly:
  - check role `admin`
  - tim withdrawal theo id
  - khong check ownership user vi day la route admin

## 8. Admin Dashboard APIs

### 8.1 GET `/api/admin/dashboard/overview`

- Muc dich: admin lay so lieu tong the de render dashboard finance trong 1 request.
- Headers:
  - `x-user-id` bat buoc
  - `x-user-role=admin` bat buoc
- Query optional:
  - `startDate`: ISO date-time, loc tu thoi diem nay
  - `endDate`: ISO date-time, loc den thoi diem nay
- Neu khong truyen `startDate/endDate`, backend tinh tong toan bo du lieu.
- Output `data`:

```json
{
  "generatedAt": "2026-05-28T00:19:26.000Z",
  "range": {
    "startDate": "2026-05-01T00:00:00.000Z",
    "endDate": "2026-05-28T23:59:59.999Z"
  },
  "deposits": {
    "totalCount": 120,
    "pendingCount": 5,
    "processingCount": 1,
    "completedCount": 110,
    "failedCount": 2,
    "cancelledCount": 2,
    "completedMoneyAmount": 25000000,
    "completedCoinAmount": 250000
  },
  "withdrawals": {
    "totalCount": 35,
    "pendingCount": 4,
    "completedCount": 28,
    "rejectedCount": 2,
    "cancelledCount": 1,
    "pendingMoneyAmount": 1200000,
    "completedMoneyAmount": 8400000,
    "pendingCoinAmount": 4000,
    "completedCoinAmount": 28000,
    "totalFeeAmount": 500000,
    "pendingFeeAmount": 60000,
    "completedFeeAmount": 420000
  },
  "revenue": {
    "totalPaymentCoins": 30000,
    "creatorRevenueCoins": 24000,
    "systemRevenueCoins": 6000,
    "videoSystemRevenueCoins": 4000,
    "membershipSystemRevenueCoins": 2000,
    "pendingSystemRevenueCoins": 1500,
    "releasedSystemRevenueCoins": 4500
  },
  "wallets": {
    "userWalletCount": 1000,
    "activeUserWalletCount": 980,
    "totalUserAvailableBalance": 900000,
    "totalUserFrozenBalance": 50000,
    "systemRevenueAvailableBalance": 4500,
    "systemRevenueFrozenBalance": 1500
  },
  "transactions": {
    "totalCount": 500,
    "pendingCount": 20,
    "completedCount": 470,
    "failedCount": 5,
    "cancelledCount": 5,
    "depositCoinAmount": 250000,
    "withdrawalCoinAmount": 90000,
    "videoPurchaseCoins": 20000,
    "membershipCoins": 10000
  }
}
```

- Y nghia cac nhom data:
  - `deposits`: tong hop lenh nap theo `requestedAt`; so tien/coin chi cong vao cac deposit `completed`.
  - `withdrawals`: tong hop lenh rut theo `requestedAt`; `pendingCoinAmount`/`completedCoinAmount` la tong coin rut theo trang thai; `totalFeeAmount` la tong phi rut tien backend da tinh tren cac lenh rut trong range.
  - `revenue.systemRevenueCoins`: tong coin hoa hong web thu duoc tu transaction `system_revenue`.
  - `revenue.creatorRevenueCoins`: tong coin chia cho creator tu transaction `channel_revenue`.
  - `revenue.pendingSystemRevenueCoins`: hoa hong web dang nam trong settlement `pending`, chua release ve available balance.
  - `revenue.releasedSystemRevenueCoins`: hoa hong web da release trong settlement.
  - `wallets.systemRevenueAvailableBalance`: balance hien tai cua vi `system_revenue`, khong loc theo date range.
  - `wallets.systemRevenueFrozenBalance`: frozen balance hien tai cua vi `system_revenue`, khong loc theo date range.
  - `transactions`: tong hop ledger theo `createdAt`.

- Backend xu ly:
  - check `x-user-role` phai la `admin`
  - validate `startDate/endDate` neu co
  - neu `startDate > endDate` tra `400 startDate must be before endDate`
  - query truc tiep bang TypeORM read model, khong thay doi du lieu

- Luu y:
  - Cac metric money dang la VND integer.
  - Cac metric coin dang la coin integer.
  - Metric wallet la snapshot hien tai, khong phai snapshot lich su theo range.

## 9. Payment APIs

### 9.1 Payment entrypoint

- Finance-service khong expose public payment creation endpoint.
- Client mua video/membership phai goi media-service:
  - `POST /api/media/videos/:id/purchase`
  - `POST /api/media/channels/:channelId/memberships/:tierId/purchase`
- Finance-service chi nhan charge video/membership qua internal API ben duoi
  hoac qua luong Kafka auto-renew.

### 9.2 POST `/api/internal/payments/charge`

- Muc dich: API noi bo de media-service charge coin sau khi da validate video
  hoac membership authoritative.
- Khong phai public gateway contract. Khong cho client goi truc tiep.
- Headers:
  - `x-internal-secret: <FINANCE_INTERNAL_GATEWAY_SECRET>`
  - `idempotency-key` bat buoc
  - `x-request-id` optional

- Input body:

```json
{
  "payerUserId": "user-1",
  "serviceType": "video",
  "serviceId": "video-123",
  "channelId": "channel-123",
  "channelOwnerId": "channel-owner-1",
  "coinAmount": 100,
  "metadata": {
    "videoTitle": "How to Build a Media System",
    "channelName": "Distributed Media Lab",
    "thumbnailUrl": "https://cdn.example.com/thumbnails/video-123.jpg"
  }
}
```

- Backend xu ly:
  - verify `x-internal-secret` bang internal gateway secret cua finance-service
  - map `payerUserId` thanh payer `userId`
  - reuse payment logic hien co: tru coin, chia revenue, tao transaction,
    tao settlement va day outbox event
  - idempotency theo `payerUserId + idempotency-key`

- Media-service contract:
  - client chi goi media-service purchase endpoint
  - media-service tu lay gia, owner, channel, trang thai video/tier tu DB cua
    media-service
  - media-service tao entitlement sync sau khi charge thanh cong
  - media-service van consume `video.payment.success` /
    `membership.payment.success` de reconcile idempotent

### 9.3 Membership auto-renew charge

- Muc dich: finance-service tu dong tru coin khi media-service phat event den han gia han membership.
- Khong co public HTTP API rieng. Luong nay chay qua Kafka de tranh client gia mao yeu cau charge.
- Input event:
  - topic `membership.auto_renew.requested`
  - `data.membershipRecordId`
  - `data.userId`
  - `data.channelId`
  - `data.channelOwnerId`
  - `data.membershipTierId`
  - `data.coinAmount`
  - `data.currentExpiryDate`
  - `data.paymentType = renew`
  - `data.idempotencyKey`
- Backend xu ly:
  - reuse payment membership logic hien co
  - idempotency theo `userId + idempotencyKey`
  - tru coin user, chia revenue creator/system, tao transaction va settlement
  - neu thanh cong, day `membership.payment.success`
  - neu that bai, day `membership.auto_renew.failed`
- `membership.payment.success` cho membership co them cac field:
  - `paymentType` (`new` | `renew` | `upgrade`)
  - `chargedCoinAmount`
  - `ledgerReferenceId`
  - `membershipRecordId` optional
  - `currentExpiryDate` optional
  - `expiryDate` optional
- `membership.auto_renew.failed` data:

```json
{
  "membershipRecordId": "membership-record-id",
  "userId": "user-id",
  "channelId": "channel-id",
  "membershipTierId": "tier-id",
  "reasonCode": "INSUFFICIENT_BALANCE",
  "retryable": true,
  "idempotencyKey": "membership-renew:membership-record-id:2026-05-18T00:00:00.000Z"
}
```

## 10. Mapping trang thai nghiep vu cho FE

### Deposit status

- `pending`: da tao lenh nap, cho thanh toan hoac cho webhook
- `processing`: PayOS/webhook da ghi nhan dang xu ly
- `completed`: nap thanh cong, coin da cong vi
- `failed`: thanh toan loi, het han, hoac bi huy
- `cancelled`: trang thai huy neu co xu ly huy rieng

### Withdrawal status

- `pending`: user vua tao lenh rut, cho admin xu ly
- `rejected`: admin tu choi, coin da hoan vi
- `completed`: admin da chuyen tien xong
- `cancelled`: user tu huy, coin da hoan vi

### Transaction status

- `pending`: da sinh ledger nhung chua hoan tat, vi du revenue cho creator/system dang bi freeze cho settlement
- `completed`: hoan tat
- `failed`: that bai
- `cancelled`: da huy

### Wallet status/type

- Wallet `status`: `active`, `frozen`, `closed`
- Wallet `type`: `user`, `system_revenue`

### Transaction type/assetType

- `assetType`: `money`, `coin`
- `type` pho bien: `deposit`, `withdrawal`, `video_purchase`, `member_subscription`, `channel_revenue`, `system_revenue`, `refund`, `system_adjustment`
- Voi `type=deposit`, transaction la ledger cong coin vao vi:
  - `assetType=coin`
  - `amount=totalCoinAmount`
  - `metadata.moneyAmount` la so tien user da thanh toan

## 11. Loi FE se hay gap

- `400 Bad Request`
  - thieu field body
  - sai kieu du lieu
  - gui field khong nam trong DTO
  - reconcile dang chay do
  - amount hoac gateway khong khop webhook

- `401 Unauthorized`
  - thieu `x-user-id`
  - thieu hoac sai `x-internal-secret`
  - webhook thieu signature
  - webhook timestamp khong hop le

- `403 Forbidden`
  - khong phai owner cua transaction hoac withdrawal
  - API admin nhung role khong phai `admin`

- `404 Not Found`
  - khong tim thay wallet, deposit, withdrawal, transaction, package

- `409 Conflict`
  - `idempotency-key` bi reuse voi payload khac
  - `deposit package code` bi trung

## 12. Goi y tich hop FE

- Man vi:
  - goi `GET /api/wallets/me`

- Man Studio Wallet:
  - goi `GET /api/studio/wallet/me`
  - goi `GET /api/studio/wallet/stats`

- Man Studio Earnings:
  - goi `GET /api/studio/earnings/summary`
  - goi `GET /api/studio/earnings/monthly?year=YYYY&month=M`
  - goi `GET /api/studio/earnings/top-videos?limit=3`

- Man lich su giao dich:
  - goi `GET /api/transactions/me`
  - can drill down thi goi `GET /api/transactions/:transactionId`
  - voi transaction nap tien, hien thi dang `Nap {metadata.moneyAmount} VND, nhan +{amount} coin`

- Man nap tien:
  - goi `GET /api/deposits/packages`
  - user chon goi xong goi `POST /api/deposits`
  - redirect user sang `checkoutUrl`

- Man rut tien:
  - goi `POST /api/withdrawals`
  - goi `GET /api/withdrawals/me` de load lich su
  - cho phep user huy bang `POST /api/withdrawals/:withdrawalId/cancel` khi con pending

- Man admin dashboard:
  - goi `GET /api/admin/dashboard/overview`
  - truyen `startDate/endDate` khi can loc theo ky bao cao
  - dung `revenue.systemRevenueCoins` de hien thi hoa hong web thu duoc bang coin
  - dung `withdrawals.totalFeeAmount` hoac `withdrawals.completedFeeAmount` de hien thi phi rut tien

- Health check:
  - goi `GET /api/health`
  - dung cho gateway/load balancer hoac UI trang thai service

- Man thanh toan video/membership:
  - khong goi finance-service truc tiep
  - mua video: goi `POST /api/media/videos/:id/purchase`
  - mua membership: goi `POST /api/media/channels/:channelId/memberships/:tierId/purchase`
  - FE nen gui/reuse `idempotency-key` theo tung action mua de retry khong tru coin hai lan

## 13. Ghi chu quan trong

- Cac API admin cua `deposits` dang check role `admin`.
- Cac API admin cua `withdrawals` cung dang check role `admin` trong use case.
- API admin dashboard dang check role `admin` trong use case.
- `GET /api/transactions/reference/:referenceId` hien khong check ownership, nen khong nen expose truc tiep cho FE public neu chua co tang kiem soat phia gateway/BFF.
- Webhook PayOS co the tra `status: "ignored"` thay vi loi khi payload khong phai success event.
