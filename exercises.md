# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: viết nội dung phản ánh ngay dưới từng câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Dương Dương  Mã học viên: 2A202602398

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy mà quên set `AGENT_API_KEY`, fail fast làm container dừng ngay và
> log báo thiếu cấu hình. Nhờ vậy mình phát hiện lỗi trước khi service nhận
> traffic. Nếu mặc định là `"changeme"`, service vẫn chạy với một khóa dễ đoán;
> người lạ có thể gọi `/ask` và tiêu ngân sách trước khi mình nhận ra.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log mình thu được:
> `{"event":"ask_completed","level":"info","timestamp":"2026-09-28T09:34:39.057770+00:00","user_id":"rate-check-1790588078925","tokens_in":392,"tokens_out":43,"cost_usd":8.46e-05}`.
> Từ JSON này mình có thể lọc/tổng hợp chi phí theo `user_id`, đồng thời tính
> số request hoặc tỷ lệ lỗi theo khoảng thời gian nhờ `event`, `level` và
> `timestamp`. Chuỗi `print("đã trả lời xong")` không có các trường để máy
> truy vấn hay cảnh báo tự động.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (full `python:3.11`) | 427.7 MB |
| Multi-stage (`python:3.11-slim`) | 63.9 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Chênh lệch đo được khoảng 363.8 MB. Phần lớn đến từ base image
> `python:3.11` đầy đủ so với `python:3.11-slim`; bản runtime chỉ nhận thư viện
> đã cài và source cần chạy, không mang theo filesystem lớn của image đầy đủ
> hay các artifact chỉ phục vụ giai đoạn build.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer base image, `WORKDIR`, tạo `appuser`,
> `COPY requirements.txt` và `pip install` được lấy lại từ cache. Docker chỉ
> chạy lại layer copy `app/` và các layer sau nó. Nếu đặt `COPY . .` trước
> `pip install`, bất kỳ thay đổi source nào cũng làm layer copy đổi và buộc
> Docker cài lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng cho phép thực thi lệnh, kẻ tấn công sẽ chạy lệnh
> với UID của process trong container. Khi process là root, họ có quyền sửa
> filesystem container và một lỗi cấu hình/mount hoặc lỗ hổng runtime có thể
> mở đường tác động tới host với quyền cao. `USER appuser` làm process chạy với
> UID 10001, nên bước đầu tiên chỉ cho quyền của user hạn chế. Nó không loại bỏ
> mọi khả năng container escape, nhưng giảm đáng kể quyền và phạm vi thiệt hại.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa là 20 request trong 2 giây: gửi 10 request ở cuối phút, ví dụ
> 10:00:59, rồi gửi thêm 10 request ngay sau khi bộ đếm reset ở 10:01:00.
> Sliding window nhìn lại đúng 60 giây nên vẫn thấy cả hai nhóm và chặn nhóm
> thứ hai khi tổng vượt hạn mức.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ/số request trong cửa sổ ngắn; cost guard giới hạn
> tổng số tiền theo tháng. Một request rất dài có thể vẫn nằm trong 10
> request/phút nhưng làm tổng chi phí vượt ngân sách, nên cost guard chặn.
> Ngược lại, user còn gần như nguyên ngân sách nhưng gửi request thứ 11 trong
> một phút thì rate limiter chặn dù cost guard vẫn cho phép về mặt tiền.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm endpoint gộp trả 503 cho cả liveness. Orchestrator hiểu
> nhầm rằng ba process bị hỏng và restart cả ba container. Trong lúc restart,
> cụm không còn instance nhận request; nếu Redis vẫn chưa phục hồi, container
> mới lại fail probe và tiếp tục restart. Với hai endpoint riêng, `/health`
> vẫn 200 nên process không bị restart, còn `/ready` 503 chỉ rút các instance
> khỏi load balancer cho đến khi Redis hoạt động lại.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi kiểm tra hai request liên tiếp với cùng user trên Redis, mình thấy
> `history_length` lần lượt là 0 rồi 2 vì câu hỏi và câu trả lời của lượt đầu
> đã được lưu chung. Hai `ConversationStore` khác nhau trong test cũng đọc
> được cùng dữ liệu. Nếu dùng dict Python, mỗi container có một dict riêng;
> request rơi vào instance khác sẽ thấy history về 0 hoặc tăng không đều theo
> instance thay vì 0, 2, 4, ... một cách nhất quán.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi chạy stack lần đầu, container `agent` liên tục restart và log báo
> `NotImplementedError: TODO (CP4): cài đặt install` tại
> `lifecycle.install()`. Mình tìm nguyên nhân bằng `docker compose ps` và
> `docker compose logs agent`: lifespan luôn gọi hàm này khi Uvicorn startup.
> Mình cài đặt đăng ký `SIGTERM`/`SIGINT`, lưu handler cũ và chuyển tiếp tín
> hiệu trong `request_shutdown`. Sau khi rebuild, container chuyển sang
> `healthy`, `/health` và `/ready` đều trả 200. Vì môi trường không có phiên
> cloud đã đăng nhập, CP5 được nộp theo local fallback thay vì bịa URL public.
