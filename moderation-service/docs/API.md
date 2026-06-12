# Moderation Service API

Moderation Service không cung cấp API nghiệp vụ trực tiếp cho Frontend. Service này chủ yếu hoạt động dưới dạng worker Kafka để xử lý yêu cầu kiểm duyệt video.

Các endpoint HTTP hiện có phục vụ kiểm tra trạng thái service và worker.

## 1. Base URL

Khi chạy local, service mặc định lắng nghe:

```text
http://localhost:8000
```

Port có thể thay đổi theo cấu hình lúc chạy service.

## 2. Endpoint

### 2.1 GET `/`

Mục đích: kiểm tra thông tin cơ bản của service.

Response HTTP 200:

```json
{
  "service": "moderation-service",
  "version": "0.1.0"
}
```

### 2.2 GET `/health/live`

Mục đích: kiểm tra process FastAPI còn sống.

Response HTTP 200:

```json
{
  "status": "ok"
}
```

### 2.3 GET `/health/ready`

Mục đích: kiểm tra service đã sẵn sàng xử lý moderation job hay chưa.

Response khi sẵn sàng:

```json
{
  "status": "ok",
  "worker": {
    "started": true,
    "ready": true,
    "threadAlive": true,
    "workerCount": 1,
    "parallelEnabled": false,
    "lastError": null,
    "lastMessageAt": "2026-06-12T10:00:00Z",
    "processedCount": 10
  }
}
```

Response khi chưa sẵn sàng trả HTTP 503:

```json
{
  "status": "not_ready",
  "worker": {
    "started": false,
    "ready": false,
    "threadAlive": false,
    "workerCount": 1,
    "parallelEnabled": false,
    "lastError": "Kafka connection failed",
    "lastMessageAt": null,
    "processedCount": 0
  }
}
```

## 3. Luồng tích hợp chính

Moderation Service nhận và trả dữ liệu qua Kafka thay vì HTTP API nghiệp vụ:

| Hướng | Event | Ý nghĩa |
| --- | --- | --- |
| Consume | `video.moderation.requested` | Media Service yêu cầu kiểm duyệt video sau khi có file cần kiểm tra. |
| Publish | `video.moderation.completed` | Moderation Service trả kết quả kiểm duyệt cho Media Service. |

Chi tiết event xem thêm tại [`EVENTS.md`](EVENTS.md).
