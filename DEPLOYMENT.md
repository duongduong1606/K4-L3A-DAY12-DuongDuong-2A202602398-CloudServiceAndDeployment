# Thông Tin Deploy — Checkpoint 5

> Service đã được deploy công khai trên Render. Tài liệu này chỉ ghi tên biến
> môi trường và nguồn cấp, không chứa giá trị API key.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Dương Dương |
| Mã học viên | 2A202602398 |
| Repo | https://github.com/duongduong1606/K4-L3A-DAY12-DuongDuong-2A202602398-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-civg.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy và kiểm tra | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Đã set | Nguồn |
|------|--------|-------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Render secret, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value `day12-redis` cấp connection string |
| `RATE_LIMIT_PER_MINUTE` | ✅ | `render.yaml` |
| `MONTHLY_BUDGET_USD` | ✅ | `render.yaml` |
| `LOG_LEVEL` | ✅ | `render.yaml` |

## Lệnh Kiểm Tra

```bash
URL=https://day12-agent-civg.onrender.com

curl -i "$URL/health"
curl -i "$URL/ready"
curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
```

## Kết Quả Chạy Thật

Kiểm tra từ máy local ngày 2026-09-28:

```text
GET https://day12-agent-civg.onrender.com/health
HTTP 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}

GET https://day12-agent-civg.onrender.com/ready
HTTP 200 {"status":"ready","redis":true}

POST https://day12-agent-civg.onrender.com/ask không có X-API-Key
HTTP 401 {"detail":"invalid or missing API key"}
```

Kết quả chứng minh service HTTPS hoạt động, app kết nối được Render Key Value
và endpoint `/ask` không cho request chưa xác thực đi qua.

## Ảnh Chụp Màn Hình

- `screenshots/health.png` — phản hồi `/health` từ URL Render công khai.
- `screenshots/ready.png` — phản hồi `/ready` từ URL Render công khai.
