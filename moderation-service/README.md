# Moderation Service

**Source repo:** https://github.com/nanhph04/moderation-service-da.git

Moderation Service kiểm duyệt nội dung video bằng cách lấy mẫu frame, chạy mô hình phát hiện nội dung NSFW và phát event kết quả kiểm duyệt cho Media Service.

## Vai trò chính

- Consume event/yêu cầu `video.moderation.requested` từ Kafka.
- Tải video gốc từ Object Storage.
- Trích xuất frame theo khoảng thời gian cấu hình.
- Phân tích frame bằng mô hình AI/NSFW classifier.
- Quyết định video an toàn, cần duyệt thủ công, bị từ chối hoặc lỗi xử lý.
- Publish event `video.moderation.completed` để Media Service cập nhật trạng thái kiểm duyệt.
- Cung cấp health check cho hạ tầng giám sát.

## Tài liệu

| Tài liệu | Mục đích |
| --- | --- |
| [`docs/API.md`](docs/API.md) | Health API và phạm vi API của Moderation Service. |
| [`docs/DATABASE.md`](docs/DATABASE.md) | Ghi chú phạm vi dữ liệu của service. |
| [`docs/ENV.md`](docs/ENV.md) | Biến môi trường cho moderation policy, model và worker concurrency. |
| [`docs/EVENTS.md`](docs/EVENTS.md) | Event kiểm duyệt publish/consume. |
| [`docs/FLOWS.md`](docs/FLOWS.md) | Luồng kiểm duyệt nội dung video. |

## Ghi chú phạm vi

- Service này chủ yếu chạy dạng worker nền, không expose API nghiệp vụ cho Frontend.
- Kết quả kiểm duyệt được đồng bộ về Media Service bằng event.
- Các API hiện có chủ yếu phục vụ health check và trạng thái worker.
