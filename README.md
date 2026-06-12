# Distributed Media System

Kho lưu trữ này là **repo tài liệu tổng quát** cho đồ án Distributed Media System. Nội dung trong repo dùng để tổng hợp kiến trúc, API, cơ sở dữ liệu, biến môi trường, event và các biểu đồ nghiệp vụ của toàn bộ hệ thống.

Repo này không phải là nơi triển khai source code chính của từng service. Source code của từng service được quản lý ở các repo/thư mục riêng và được liên kết trong từng thư mục tài liệu bên dưới.

## 1. Mục tiêu hệ thống

Distributed Media System là hệ thống nền tảng media/video theo kiến trúc microservices, hỗ trợ các nhóm nghiệp vụ chính:

- Đăng ký, đăng nhập, quản lý hồ sơ và phân quyền người dùng.
- Tạo kênh, tải video, xử lý video, kiểm duyệt nội dung và phát video.
- Quản lý danh mục, gói hội viên, mở khóa video và quyền xem nội dung trả phí.
- Nạp tiền, thanh toán nội bộ, quản lý ví, yêu cầu rút tiền và đối soát giao dịch.
- Xử lý bất đồng bộ qua message broker để giảm phụ thuộc trực tiếp giữa các service.

## 2. Kiến trúc tổng quan

```text
Frontend / Mobile
       |
       v
API Gateway
       |
       +--> Identity Service
       +--> Media Service
       +--> Finance Service
       +--> Media Processing Service
       +--> Moderation Service

Hạ tầng dùng chung:
- PostgreSQL: lưu dữ liệu nghiệp vụ của từng service.
- Kafka: truyền integration event giữa các service.
- Redis/BullMQ: hàng đợi xử lý video.
- MinIO/Object Storage: lưu video gốc, HLS output và thumbnail.
- FFmpeg/AI model: xử lý media và kiểm duyệt nội dung.
```

## 3. Các service trong đồ án

| Service | Vai trò chính | Tài liệu |
| --- | --- | --- |
| Frontend | Giao diện người dùng cho người xem, chủ kênh và quản trị viên; tích hợp backend qua API Gateway. | [`frontend/README.md`](frontend/README.md) |
| API Gateway | Cổng vào chung cho Frontend/Mobile, định tuyến request và gắn thông tin xác thực cho service nội bộ. | [`api-gateway/README.md`](api-gateway/README.md) |
| Identity Service | Quản lý tài khoản, đăng nhập, refresh token, hồ sơ người dùng và khóa/mở khóa người dùng. | [`identity-service/README.md`](identity-service/README.md) |
| Media Service | Quản lý kênh, video, danh mục, gói hội viên, quyền xem video và các nghiệp vụ media chính. | [`media-service/README.md`](media-service/README.md) |
| Media Processing Service | Xử lý video bất đồng bộ: tải video gốc, transcode HLS, tạo thumbnail và phát event kết quả. | [`media-proces-service/README.md`](media-proces-service/README.md) |
| Moderation Service | Kiểm duyệt nội dung video bằng AI/NSFW model và phát kết quả kiểm duyệt. | [`moderation-service/README.md`](moderation-service/README.md) |
| Finance Service | Quản lý ví, nạp tiền, thanh toán, giao dịch nội bộ, doanh thu và rút tiền. | [`finance-service/README.md`](finance-service/README.md) |

## 4. Luồng nghiệp vụ chính

Các luồng quan trọng đã được tách thành tài liệu và biểu đồ trong từng service:

- **Xác thực người dùng:** đăng ký, đăng nhập, đăng xuất, quên mật khẩu, cập nhật hồ sơ.
- **Quản trị người dùng:** quản trị viên khóa/mở khóa người dùng và đồng bộ trạng thái sang các service liên quan.
- **Quản lý media:** tạo kênh, tạo danh mục, cập nhật metadata video, ẩn/xuất bản video.
- **Tải và xử lý video:** upload multipart, xác nhận upload, đưa video vào hàng đợi xử lý, nhận kết quả xử lý.
- **Kiểm duyệt video:** gửi yêu cầu kiểm duyệt, phân tích frame, trả kết quả an toàn/chờ duyệt/từ chối.
- **Xem video:** kiểm tra quyền xem, lấy dữ liệu phát HLS, ghi nhận lượt xem và tiến độ xem.
- **Nội dung trả phí:** mua gói hội viên, mở khóa video, kiểm tra quyền truy cập.
- **Tài chính:** nạp tiền qua webhook, thanh toán nội bộ, ghi nhận giao dịch, yêu cầu rút tiền và duyệt rút tiền.

## 5. Quy ước tài liệu

Mỗi service ưu tiên có cùng cấu trúc tài liệu:

```text
<service>/
  README.md              # Tổng quan service và liên kết tài liệu
  docs/
    API.md               # Danh sách API và contract cho FE/BFF
    DATABASE.md          # Thiết kế dữ liệu hoặc ERD
    ENV.md               # Biến môi trường quan trọng
    EVENTS.md            # Integration event publish/consume
    FLOWS.md             # Luồng nghiệp vụ chính
    SEQUENCE_DIAGRAMS.md # Mục lục sequence diagram, nếu có
    ACTIVITY_DIAGRAMS.md # Mục lục activity diagram, nếu có
```

Một số service không có database hoặc không expose API nghiệp vụ sẽ ghi rõ phạm vi trong README thay vì tạo tài liệu giả.

## 6. Hướng dẫn đọc tài liệu

1. Đọc README này để nắm kiến trúc tổng quan.
2. Đọc [`INSTALLATION.md`](INSTALLATION.md) nếu cần cài đặt và chạy hệ thống local.
3. Vào README của từng service để biết repo source, trách nhiệm và danh sách tài liệu liên quan.
4. Khi cần ghép Frontend, ưu tiên đọc `API.md` của service tương ứng.
5. Khi cần viết báo cáo, ưu tiên dùng `FLOWS.md`, `SEQUENCE_DIAGRAMS.md`, `ACTIVITY_DIAGRAMS.md` và các file PlantUML.
6. Khi cần giải thích tích hợp giữa service, đọc `EVENTS.md` của các service có liên quan.

## 7. Liên kết source code

| Thành phần | Repo/source tham khảo |
| --- | --- |
| API Gateway | https://github.com/nanhph04/api_gateway_da.git |
| Identity Service | https://github.com/nanhph04/identity-service-da.git |
| Media Service | https://github.com/nanhph04/media-serivce-da.git |
| Media Processing Service | https://github.com/nanhph04/media-prc-service-da.git |
| Moderation Service | https://github.com/nanhph04/moderation-service-da.git |
| Finance Service | https://github.com/nanhph04/finance-service-da.git |

## 8. Trạng thái tài liệu

- Đã có tài liệu API, database, event, env và flow cho các service nghiệp vụ chính.
- Đã có nhiều sequence diagram và activity diagram cho Identity Service, Media Service và Finance Service.
- Media Processing Service và Moderation Service tập trung vào worker/event nên tài liệu nhấn mạnh luồng xử lý bất đồng bộ thay vì API nghiệp vụ.
- API Gateway hiện được mô tả ở mức tổng quan vì vai trò chính là routing và bảo vệ đường vào hệ thống.

## 9. Cài đặt và chạy local

Xem hướng dẫn chi tiết tại [`INSTALLATION.md`](INSTALLATION.md).
