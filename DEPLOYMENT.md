# Thông Tin Deploy — Checkpoint 5

> Bài nộp này dùng phương án dự phòng cục bộ theo hướng dẫn của lab.
> Tài liệu chỉ ghi tên biến môi trường, không chứa giá trị API key.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Dương Dương |
| Mã học viên | 2A202602398 |
| Repo | https://github.com/duongduong1606/K4-L3A-DAY12-DuongDuong-2A202602398-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | Không có — dùng `http://localhost:8000` theo local fallback |
| Platform | Docker Compose local fallback; chưa deploy Railway/Render |
| Ngày kiểm tra | 2026-09-28 |

## Biến Môi Trường Đã Set

Chỉ liệt kê tên biến và nguồn cấp, không ghi giá trị secret:

| Biến | Đã set | Nguồn |
|------|--------|-------|
| `PORT` | ✅ | file `.env` cục bộ |
| `AGENT_API_KEY` | ✅ | file `.env` cục bộ, không commit |
| `REDIS_URL` | ✅ | Compose ghi đè thành `redis://redis:6379/0` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | file `.env` cục bộ |
| `MONTHLY_BUDGET_USD` | ✅ | file `.env` cục bộ |
| `LOG_LEVEL` | ✅ | file `.env` cục bộ |
| `LOCAL_FALLBACK` | ✅ | `true` trong file `.env` cục bộ |

## Lệnh Kiểm Tra

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/ready
curl -i -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
```

Request có xác thực và kiểm tra rate limit được chạy bằng API key lấy từ
`.env`; giá trị khóa không được in ra terminal hoặc ghi vào tài liệu này.

## Kết Quả Chạy Thật

Kiểm tra ngày 2026-09-28:

```text
docker compose ps
agent: running, healthy, 0.0.0.0:8000->8000/tcp
redis: running, healthy, 0.0.0.0:6379->6379/tcp

GET /health
HTTP 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP 200 {"status":"ready","redis":true}

POST /ask không có X-API-Key
HTTP 401

POST /ask có X-API-Key
HTTP 200, answer_present=true

15 request liên tiếp với cùng user
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

- `screenshots/health.png` — phản hồi thật của `/health` trong trình duyệt.
- `screenshots/ready.png` — phản hồi thật của `/ready`, xác nhận Redis hoạt động.

## Lý Do Dùng Phương Án Dự Phòng

Môi trường làm bài không có Railway/Render CLI và không có phiên trình duyệt
cloud đã đăng nhập. Vì không thể deploy công khai mà không yêu cầu thêm tài
khoản hoặc credential, bài dùng phương án Docker Compose local fallback theo
hướng dẫn chính thức. CP5 vì vậy bị giới hạn tối đa 9/15 điểm.
