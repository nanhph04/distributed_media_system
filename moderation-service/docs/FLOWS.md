# Moderation Service Flows

Tài liệu này mô tả các luồng xử lý chính của Moderation Service trong hệ thống Distributed Media System.

## 1. Tổng quan luồng kiểm duyệt video

```text
Media Service
    |
    | publish video.moderation.requested
    v
Kafka
    |
    v
Moderation Service Worker
    |
    +--> Tải video gốc từ Object Storage
    +--> Trích xuất frame theo khoảng thời gian cấu hình
    +--> Chạy NSFW classifier trên từng frame
    +--> Tổng hợp điểm số và bằng chứng
    +--> Quyết định kết quả kiểm duyệt
    |
    | publish video.moderation.completed
    v
Media Service cập nhật trạng thái video
```

## 2. Các bước xử lý chính

1. Media Service phát event `video.moderation.requested` sau khi video cần được kiểm duyệt.
2. Moderation Service consume event từ Kafka.
3. Worker tải file video gốc từ bucket/object key được gửi trong event.
4. Service trích xuất frame theo `FRAME_INTERVAL_SECONDS` và giới hạn xử lý theo cấu hình.
5. Mỗi frame được phân tích bằng model NSFW chính.
6. Nếu frame có dấu hiệu nhạy cảm, service có thể dùng detector phụ để xác nhận thêm.
7. Service tổng hợp kết quả theo chính sách `NSFW_POLICY` và các threshold.
8. Service phát event `video.moderation.completed` với trạng thái cuối cùng.
9. Media Service nhận event và cập nhật trạng thái kiểm duyệt của video.

## 3. Trạng thái kết quả

| Trạng thái | Ý nghĩa |
| --- | --- |
| `SAFE` | Video được xem là an toàn và có thể tiếp tục quy trình xuất bản. |
| `PENDING_MANUAL_REVIEW` | Video có dấu hiệu cần quản trị viên kiểm tra thủ công. |
| `REJECTED` | Video bị từ chối do phát hiện nội dung vi phạm rõ ràng. |
| `ERROR` | Có lỗi trong quá trình tải, trích xuất frame, phân tích hoặc publish kết quả. |

## 4. Chính sách kiểm duyệt mặc định

Chính sách mặc định là `explicit_only`:

- Chỉ tự động từ chối khi có bằng chứng rõ ràng về nội dung explicit/porn.
- Cần đủ số frame nghi ngờ gần nhau theo `NSFW_REJECT_MIN_FRAMES` và `NSFW_REJECT_WINDOW_SECONDS`.
- Detector phụ như OpenNSFW2 được dùng để xác nhận trước khi tự động reject.
- Nội dung gợi cảm hoặc nhạy cảm nhẹ nên chuyển sang `PENDING_MANUAL_REVIEW` thay vì tự động từ chối.

## 5. Luồng lỗi

- Nếu không tải được video: trả kết quả lỗi hoặc ghi nhận lỗi worker tùy thời điểm xử lý.
- Nếu trích xuất frame thất bại: event hoàn tất có thể mang trạng thái `ERROR`.
- Nếu Kafka hoặc Object Storage lỗi tạm thời: worker dựa vào cơ chế consumer/retry của hạ tầng để xử lý lại.
- Health endpoint `/health/ready` trả `503` khi worker chưa sẵn sàng hoặc có lỗi khởi động.

## 6. Tài liệu liên quan

- API health check: [`API.md`](API.md)
- Event kiểm duyệt: [`EVENTS.md`](EVENTS.md)
- Biến môi trường: [`ENV.md`](ENV.md)
