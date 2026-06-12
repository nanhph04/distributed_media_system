# Frontend

**Source repo:** https://github.com/nanhph04/fe-da

Frontend là ứng dụng giao diện người dùng của Distributed Media System, được xây dựng bằng Next.js. Ứng dụng là điểm tương tác chính của người xem, chủ kênh và quản trị viên với toàn bộ hệ thống thông qua API Gateway.

## Vai trò chính

- Cung cấp giao diện đăng ký, đăng nhập, đăng xuất và quản lý hồ sơ người dùng.
- Hiển thị danh sách video, chi tiết video và trình phát video HLS.
- Hỗ trợ người dùng xem video, theo dõi tiến độ xem và truy cập nội dung đã mua.
- Hỗ trợ chủ kênh tạo kênh, upload video, quản lý video, quản lý gói hội viên và theo dõi doanh thu.
- Hỗ trợ ví người dùng: nạp tiền, xem số dư, xem lịch sử giao dịch và rút tiền.
- Hỗ trợ quản trị viên quản lý người dùng, kênh, video, danh mục và yêu cầu rút tiền.
- Tích hợp với API Gateway để gọi Identity Service, Media Service và Finance Service.

## Công nghệ sử dụng

| Thành phần | Công nghệ |
| --- | --- |
| Framework | Next.js 16 |
| UI library | React 19 |
| Ngôn ngữ | TypeScript |
| Styling | Tailwind CSS, shadcn/base-ui theo cấu trúc hiện có |
| Form/validation | React Hook Form, Zod |
| Video player | video.js |
| Thanh toán | PayOS checkout |
| Kiểm thử | Jest |

## Cấu trúc chức năng chính

```text
fe/
  src/
    app/              # Next.js App Router
    features/         # Các module nghiệp vụ theo feature
    components/       # Component dùng chung
    shared/           # Tiện ích, API client, type dùng chung
    styles/           # Style toàn cục và cấu hình giao diện
```

Tùy phiên bản source thực tế, tên thư mục con có thể thay đổi nhưng nguyên tắc chính là tổ chức theo feature và ưu tiên tái sử dụng component dùng chung.

## Các nhóm màn hình chính

| Nhóm màn hình | Mục đích |
| --- | --- |
| Xác thực | Đăng ký, đăng nhập, refresh session, đăng xuất, quên mật khẩu. |
| Trang xem video | Duyệt video, xem chi tiết video, phát HLS, ghi nhận tiến độ xem. |
| Kênh/Studio | Quản lý channel, upload video, cập nhật metadata, quản lý nội dung của chủ kênh. |
| Hội viên/nội dung trả phí | Mua gói hội viên, mở khóa video, kiểm tra quyền xem. |
| Ví và thanh toán | Nạp tiền, xem số dư, xem lịch sử giao dịch, yêu cầu rút tiền. |
| Quản trị | Quản lý người dùng, kênh, video, danh mục và yêu cầu rút tiền. |

## Tích hợp API

Frontend nên gọi backend thông qua API Gateway:

```text
Frontend -> API Gateway -> Service nội bộ
```

Các biến môi trường quan trọng:

```env
NEXT_PUBLIC_GATEWAY_URL=http://localhost:4000
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_AVATAR_STORAGE_URL=http://localhost:19000
NEXT_PUBLIC_MEDIA_STORAGE_URL=http://localhost:19000
GATEWAY_INTERNAL_URL=http://api-gateway:4000
```

Quy tắc tích hợp:

- Không gọi trực tiếp Identity Service, Media Service hoặc Finance Service từ trình duyệt.
- Các request protected gửi `Authorization: Bearer <accessToken>`.
- Các API dùng refresh token/httpOnly cookie cần bật `credentials: "include"`.
- Không tự set các header nội bộ như `x-internal-secret`, `x-user-id`, `x-user-role`; API Gateway hoặc backend chịu trách nhiệm xử lý.

## Cài đặt và chạy local

### Chạy bằng Docker Compose

Từ thư mục cha của toàn bộ hệ thống:

```powershell
cd E:\doan\distributed_media_system
docker compose -f .\fe\docker-compose.yml up -d --build
```

Frontend mặc định chạy tại:

```text
http://localhost:3000
```

### Chạy ngoài Docker

```powershell
cd E:\doan\distributed_media_system\fe
npm install
npm run dev
```

Các lệnh thường dùng:

```powershell
npm run dev          # Chạy môi trường phát triển
npm run build        # Build production
npm run start        # Chạy bản production đã build
npm run lint         # Kiểm tra ESLint
npm run type-check   # Kiểm tra TypeScript
npm run test         # Chạy test
npm run validate     # Lint + type-check + build
```

## Phụ thuộc hệ thống khi chạy đầy đủ

Để frontend hoạt động đủ chức năng, cần chạy các thành phần sau:

- API Gateway: `http://localhost:4000`
- Identity Service: xử lý tài khoản và phiên đăng nhập.
- Media Service: xử lý video, channel, membership và quyền xem.
- Finance Service: xử lý ví, nạp tiền, thanh toán và rút tiền.
- MinIO/Object Storage: phục vụ avatar, thumbnail và nội dung media.
- Kafka/Redis/PostgreSQL: hạ tầng phía backend.

Hướng dẫn chạy toàn bộ hệ thống xem thêm: [`../INSTALLATION.md`](../INSTALLATION.md).

## Kiểm tra nhanh sau khi chạy

1. Mở `http://localhost:3000`.
2. Kiểm tra frontend gọi API Gateway tại `http://localhost:4000`.
3. Thử đăng ký/đăng nhập người dùng.
4. Kiểm tra các màn hình chính: danh sách video, chi tiết video, ví, studio và admin.
5. Nếu hình ảnh hoặc video không hiển thị, kiểm tra `NEXT_PUBLIC_AVATAR_STORAGE_URL`, `NEXT_PUBLIC_MEDIA_STORAGE_URL` và `PUBLIC_HOST` khi build Docker.

## Ghi chú cho báo cáo đồ án

Khi trình bày trong báo cáo, Frontend nên được mô tả là lớp giao diện và điều phối trải nghiệm người dùng, không xử lý trực tiếp nghiệp vụ lõi. Các nghiệp vụ quan trọng như xác thực, quyền xem, thanh toán, xử lý video và kiểm duyệt nội dung được thực hiện ở backend service tương ứng.
