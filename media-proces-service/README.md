# Media Processing Service

**Source repo:** https://github.com/nanhph04/media-prc-service-da.git

Media Processing Service xử lý video bất đồng bộ sau khi người dùng upload video. Service này tập trung vào pipeline kỹ thuật media như tải file gốc, phân tích metadata, transcode HLS, tạo thumbnail và phát event kết quả cho Media Service.

> Tên thư mục tài liệu hiện tại là `media-proces-service` để giữ tương thích với cấu trúc repo đang có.

## Vai trò chính

- Nhận job xử lý video qua hàng đợi BullMQ/Redis.
- Tải video gốc từ Object Storage.
- Dùng FFmpeg để đọc metadata và transcode sang HLS.
- Tạo thumbnail tự động cho video.
- Upload HLS output và thumbnail lên Object Storage.
- Phát event `video.processed.success`, `video.processed.failed`, `video.thumbnail.generated`, `video.thumbnail.failed`.

## Tài liệu

| Tài liệu | Mục đích |
| --- | --- |
| [`docs/FLOWS.md`](docs/FLOWS.md) | Luồng xử lý transcode và tạo thumbnail. |
| [`docs/EVENTS.md`](docs/EVENTS.md) | Event do Media Processing Service phát ra. |
| [`docs/ENV.md`](docs/ENV.md) | Biến môi trường cho Kafka, MinIO, FFmpeg và concurrency. |

## Ghi chú phạm vi

- Service này chủ yếu chạy dạng worker, không phải service API nghiệp vụ cho Frontend.
- Database nghiệp vụ chính của video nằm ở Media Service; Media Processing Service trả kết quả qua event.
- Khi viết báo cáo, nên mô tả service này như một thành phần xử lý nền trong luồng upload video.
