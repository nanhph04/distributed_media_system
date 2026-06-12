# Media Service

**Source repo:** https://github.com/nanhph04/media-serivce-da.git

Media Service là service trung tâm của hệ thống media, chịu trách nhiệm quản lý kênh, video, danh mục, gói hội viên, quyền xem video và các nghiệp vụ liên quan đến nội dung.

## Vai trò chính

- Tạo và quản lý channel của người dùng.
- Tải video lên Object Storage theo cơ chế multipart upload.
- Lưu metadata video, trạng thái xử lý, trạng thái kiểm duyệt và trạng thái xuất bản.
- Gửi job xử lý video sang Media Processing Service.
- Gửi yêu cầu kiểm duyệt sang Moderation Service.
- Cung cấp dữ liệu phát video HLS cho người xem đủ quyền.
- Quản lý danh mục, gói hội viên, mua hội viên và mở khóa video.
- Nhận event từ Finance Service, Media Processing Service và Moderation Service để cập nhật dữ liệu media.

## Tài liệu

| Tài liệu | Mục đích |
| --- | --- |
| [`docs/API.md`](docs/API.md) | Danh sách API Media dành cho Frontend/Mobile. |
| [`docs/DATABASE.md`](docs/DATABASE.md) | Thiết kế dữ liệu media, channel, video, membership và quyền truy cập. |
| [`docs/ENV.md`](docs/ENV.md) | Biến môi trường quan trọng. |
| [`docs/EVENTS.md`](docs/EVENTS.md) | Event publish/consume của Media Service. |
| [`docs/FLOWS.md`](docs/FLOWS.md) | Các luồng nghiệp vụ media chính. |
| [`docs/SEQUENCE_DIAGRAMS.md`](docs/SEQUENCE_DIAGRAMS.md) | Mục lục sequence diagram. |
| [`docs/ACTIVITY_DIAGRAMS.md`](docs/ACTIVITY_DIAGRAMS.md) | Mục lục activity diagram. |

## Biểu đồ tiêu biểu

- Upload video.
- Xem video.
- Mở khóa video.
- Mua gói hội viên.
- Quản lý gói hội viên.
- Tạo/cập nhật danh mục.
- Cập nhật metadata video.
- Ẩn video.
- Quản trị viên duyệt video.
- Quản trị viên quản lý channel.
