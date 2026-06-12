# Moderation Service Database

Moderation Service hiện không sở hữu database nghiệp vụ riêng trong phạm vi tài liệu đồ án này.

## 1. Phạm vi dữ liệu

Service hoạt động chủ yếu theo mô hình worker bất đồng bộ:

- Nhận dữ liệu đầu vào từ event `video.moderation.requested`.
- Tải video gốc từ Object Storage.
- Xử lý frame tạm thời trong thư mục local/temp.
- Phát kết quả qua event `video.moderation.completed`.

Dữ liệu nghiệp vụ lâu dài như video, trạng thái kiểm duyệt, lý do từ chối và lịch sử hiển thị cho người dùng được lưu ở Media Service.

## 2. Dữ liệu tạm trong quá trình xử lý

| Nhóm dữ liệu | Nơi lưu | Ghi chú |
| --- | --- | --- |
| Video gốc | Object Storage | Được tải về theo `rawBucket` và `rawFileKey` trong event. |
| Frame trích xuất | Thư mục tạm của worker | Chỉ dùng trong lúc phân tích NSFW. |
| Điểm số frame | Bộ nhớ tiến trình | Dùng để tổng hợp quyết định kiểm duyệt. |
| Kết quả kiểm duyệt | Kafka event | Gửi về Media Service để lưu bền vững. |

## 3. Lý do không tách database riêng

- Service không cần truy vấn nghiệp vụ trực tiếp từ Frontend.
- Kết quả kiểm duyệt cuối cùng thuộc về vòng đời của video nên được lưu cùng Media Service.
- Việc không có database riêng giúp worker kiểm duyệt đơn giản hơn và dễ scale theo số partition Kafka.

## 4. Liên kết liên quan

- Event contract: [`EVENTS.md`](EVENTS.md)
- Luồng xử lý: [`FLOWS.md`](FLOWS.md)
- Thiết kế dữ liệu video: [`../../media-service/docs/DATABASE.md`](../../media-service/docs/DATABASE.md)
