# Hướng dẫn cài đặt Distributed Media System

Tài liệu này hướng dẫn chuẩn bị môi trường, cấu hình `.env`, chạy hạ tầng và khởi động các service trong môi trường local.

> Repo tài liệu nằm tại `distributed_media_system/`. Source code thực tế của các service nằm ở thư mục cha `E:\doan\distributed_media_system`.

## 1. Yêu cầu môi trường

Nên cài đặt trước các công cụ sau:

| Công cụ | Phiên bản khuyến nghị | Dùng cho |
| --- | --- | --- |
| Git | Mới nhất | Clone source code. |
| Docker Desktop | Mới nhất | Chạy PostgreSQL, Kafka, Redis, MinIO và các service bằng container. |
| Node.js | 22.x | Chạy các service NestJS và Frontend khi chạy ngoài Docker. |
| npm | Theo Node.js | Cài package JavaScript/TypeScript. |
| Python | 3.11+ | Chạy Moderation Service khi chạy ngoài Docker. |
| PowerShell | 5.1+ hoặc PowerShell 7 | Chạy script local trên Windows. |
| FFmpeg | Mới nhất | Xử lý video khi chạy Media Processing Service ngoài Docker. |

Nếu chỉ chạy bằng Docker, Node.js/Python/FFmpeg trên máy host chủ yếu dùng để phát triển hoặc test riêng từng service.

## 2. Cấu trúc thư mục mong muốn

Sau khi clone đầy đủ, thư mục làm việc nên có dạng:

```text
E:\doan\distributed_media_system
  api_gateway/
  identity-service/
  media_service/
  media_processing_service/
  moderation-service/
  finance-service/
  fe/
  infrastructure/
  distributed_media_system/   # repo tài liệu tổng quát
  start-local.ps1
```

## 3. Clone source code

Nếu chưa có source code, clone các repo service vào cùng thư mục cha:

```powershell
cd E:\doan\distributed_media_system

git clone https://github.com/nanhph04/api_gateway_da.git api_gateway
git clone https://github.com/nanhph04/identity-service-da.git identity-service
git clone https://github.com/nanhph04/media-serivce-da.git media_service
git clone https://github.com/nanhph04/media-prc-service-da.git media_processing_service
git clone https://github.com/nanhph04/moderation-service-da.git moderation-service
git clone https://github.com/nanhph04/finance-service-da.git finance-service
```

Frontend và thư mục `infrastructure/` cần đặt cùng cấp với các service. Nếu đang nằm trong repo riêng, clone hoặc copy về đúng cấu trúc trên.

## 4. Chuẩn bị file môi trường

Mỗi service có file `.env.example`. Tạo `.env` từ file mẫu trước khi chạy:

```powershell
cd E:\doan\distributed_media_system

Copy-Item .\api_gateway\.env.example .\api_gateway\.env -ErrorAction SilentlyContinue
Copy-Item .\identity-service\.env.example .\identity-service\.env -ErrorAction SilentlyContinue
Copy-Item .\media_service\.env.example .\media_service\.env -ErrorAction SilentlyContinue
Copy-Item .\media_processing_service\.env.example .\media_processing_service\.env -ErrorAction SilentlyContinue
Copy-Item .\moderation-service\.env.example .\moderation-service\.env -ErrorAction SilentlyContinue
Copy-Item .\finance-service\.env.example .\finance-service\.env -ErrorAction SilentlyContinue
Copy-Item .\fe\.env.example .\fe\.env -ErrorAction SilentlyContinue
```

Sau đó mở từng file `.env` để kiểm tra các giá trị quan trọng:

- Database: host, port, username, password, database name.
- Redis: host và port.
- Kafka: broker URL.
- MinIO: endpoint, access key, secret key, bucket.
- JWT/refresh token secret.
- Internal gateway secret.
- Cấu hình email nếu dùng quên mật khẩu.
- Cấu hình PayOS hoặc cổng thanh toán nếu dùng nạp tiền thật.

Không commit `.env` thật hoặc khóa bí mật lên Git.

## 5. Khởi tạo Docker network và volume

Các file compose đang dùng network và volume external, vì vậy cần tạo trước:

```powershell
docker network create app-network

docker volume create infrastructure_postgres_data
docker volume create infrastructure_kafka-data
docker volume create infrastructure_redis_data
docker volume create infrastructure_minio_data
```

Nếu network hoặc volume đã tồn tại, Docker sẽ báo đã có sẵn; có thể bỏ qua.

## 6. Chạy hạ tầng dùng chung

Chạy PostgreSQL, Redis, Kafka và MinIO:

```powershell
cd E:\doan\distributed_media_system

docker compose -f .\infrastructure\postgres.yml up -d
docker compose -f .\infrastructure\redis.yml up -d
docker compose -f .\infrastructure\kafka.yml up -d
docker compose -f .\infrastructure\minio.yml up -d
```

PostgreSQL sẽ tạo các database ban đầu qua script init:

- `identity_service_db`
- `media_service_db`
- `finance_service_db`

Các cổng hạ tầng local thường dùng:

| Thành phần | URL/Cổng |
| --- | --- |
| PostgreSQL | `localhost:15432` |
| Redis | `localhost:16379` |
| Kafka | `localhost:19092` |
| Kafka UI | `http://localhost:18080` |
| MinIO API | `http://localhost:19000` |
| MinIO Console | `http://localhost:19001` |

MinIO mặc định dùng tài khoản local:

```text
Username: admin
Password: admin123
```

## 7. Chạy toàn bộ hệ thống bằng Docker Compose

Sau khi hạ tầng đã chạy, khởi động các service:

```powershell
cd E:\doan\distributed_media_system

docker compose -f .\identity-service\docker-compose.yml up -d --build
docker compose -f .\media_service\docker-compose.yml up -d --build
docker compose -f .\media_processing_service\docker-compose.yml up -d --build
docker compose -f .\moderation-service\docker-compose.yml up -d --build
docker compose -f .\finance-service\docker-compose.yml up -d --build
docker compose -f .\api_gateway\docker-compose.yml up -d --build
docker compose -f .\fe\docker-compose.yml up -d --build
```

Thứ tự khuyến nghị:

1. Hạ tầng: PostgreSQL, Redis, Kafka, MinIO.
2. Service nền: Media Processing Service, Moderation Service.
3. Service nghiệp vụ: Identity Service, Media Service, Finance Service.
4. API Gateway.
5. Frontend.

## 8. Script chạy nhanh

Trong thư mục cha có script:

```powershell
.\start-local.ps1
```

Script này hỗ trợ khởi động nhanh một số thành phần và tự set `PUBLIC_HOST` theo IP LAN. Trước khi dùng script, vẫn nên đảm bảo Docker network, volume, hạ tầng chính và file `.env` đã sẵn sàng.

Nếu muốn tự chỉ định host public cho link MinIO/media:

```powershell
$env:PUBLIC_HOST="localhost"
.\start-local.ps1
```

## 9. Chạy từng service ngoài Docker

### 9.1 Service NestJS

Áp dụng cho:

- `api_gateway`
- `identity-service`
- `media_service`
- `media_processing_service`
- `finance-service`

Ví dụ:

```powershell
cd E:\doan\distributed_media_system\identity-service
npm install
npm run start:dev
```

Các lệnh thường dùng:

```powershell
npm run build
npm run start:dev
npm run test
npm run lint
```

Một số service có migration riêng:

```powershell
# Identity Service
npm run migration:run

# Media Service
npm run migration:run

# Finance Service
npm run migration:run
```

### 9.2 Moderation Service

```powershell
cd E:\doan\distributed_media_system\moderation-service

python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e ".[runtime,test]"
.\.venv\Scripts\python.exe -m moderation_service --host 0.0.0.0 --port 8000
```

Chạy test:

```powershell
.\.venv\Scripts\python.exe -m pytest
```

### 9.3 Frontend

```powershell
cd E:\doan\distributed_media_system\fe
npm install
npm run dev
```

Frontend mặc định chạy tại:

```text
http://localhost:3000
```

## 10. Cổng service local

| Thành phần | Cổng mặc định | Ghi chú |
| --- | --- | --- |
| Frontend | `3000` | Giao diện người dùng. |
| API Gateway | `4000` | Client nên gọi qua cổng này. |
| Identity Service | `4001` | Service nội bộ. |
| Media Service | `4002` | Service nội bộ. |
| Media Processing Service | `4003` | Worker/API health. |
| Finance Service | `4004` | Service nội bộ. |
| Moderation Service | `4005` trên host, `8000` trong container | Health check và worker moderation. |

## 11. Kiểm tra sau khi chạy

Kiểm tra container:

```powershell
docker ps
```

Kiểm tra log một service:

```powershell
docker logs -f api-gateway
docker logs -f identity-service
docker logs -f media-service
docker logs -f finance-service
docker logs -f media-processing-service
docker logs -f moderation-service
docker logs -f frontend
```

Một số endpoint kiểm tra nhanh:

```text
Frontend:             http://localhost:3000
API Gateway:          http://localhost:4000
Identity Service:     http://localhost:4001
Media Service:        http://localhost:4002/api/media/
Finance Service:      http://localhost:4004/api/health
Media Processing:     http://localhost:4003/health/ready
Moderation Service:   http://localhost:4005/health/live
Kafka UI:             http://localhost:18080
MinIO Console:        http://localhost:19001
```

## 12. Dừng hệ thống

Dừng từng nhóm container:

```powershell
cd E:\doan\distributed_media_system

docker compose -f .\fe\docker-compose.yml down
docker compose -f .\api_gateway\docker-compose.yml down
docker compose -f .\identity-service\docker-compose.yml down
docker compose -f .\media_service\docker-compose.yml down
docker compose -f .\media_processing_service\docker-compose.yml down
docker compose -f .\moderation-service\docker-compose.yml down
docker compose -f .\finance-service\docker-compose.yml down

docker compose -f .\infrastructure\minio.yml down
docker compose -f .\infrastructure\kafka.yml down
docker compose -f .\infrastructure\redis.yml down
docker compose -f .\infrastructure\postgres.yml down
```

Không xóa volume nếu muốn giữ dữ liệu local.

## 13. Lỗi thường gặp

### 13.1 Compose báo thiếu `app-network`

Chạy:

```powershell
docker network create app-network
```

### 13.2 Compose báo thiếu external volume

Tạo lại các volume:

```powershell
docker volume create infrastructure_postgres_data
docker volume create infrastructure_kafka-data
docker volume create infrastructure_redis_data
docker volume create infrastructure_minio_data
```

### 13.3 Service không kết nối được database/Kafka/Redis

Kiểm tra:

- Hạ tầng đã chạy chưa bằng `docker ps`.
- Service chạy trong Docker phải dùng host nội bộ: `postgres`, `kafka`, `redis`, `minio`.
- Service chạy ngoài Docker phải dùng host local: `localhost` và port publish như `15432`, `19092`, `16379`, `19000`.

### 13.4 Link ảnh/video MinIO không mở được từ máy khác trong LAN

Set `PUBLIC_HOST` bằng IP LAN trước khi build/chạy container:

```powershell
$env:PUBLIC_HOST="192.168.1.10"
```

Sau đó recreate các service có dùng public URL như Media Service, Media Processing Service và Frontend.

### 13.5 Moderation Service tải model lâu hoặc tốn tài nguyên

Lần đầu chạy service có thể tải AI model và dùng nhiều CPU/RAM. Nên chạy trên máy có đủ tài nguyên hoặc tạm giảm concurrency trong `.env`.
