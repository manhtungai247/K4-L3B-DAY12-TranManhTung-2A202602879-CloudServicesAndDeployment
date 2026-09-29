# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Mạnh Tùng  Mã học viên: 2A202602879

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu deploy một bản mới lên Render mà quên khai báo `AGENT_API_KEY`, cấu hình bắt buộc làm app dừng ngay khi khởi động. Nhờ vậy mình phát hiện thiếu secret trong lúc deploy, thay vì để service chạy với khóa công khai `changeme` mà người lạ có thể đoán và dùng để gọi `/ask`, làm tiêu tốn ngân sách.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng JSON lấy từ Render Logs sau request `/ask`:
>
> ```json
> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T04:19:26.407732+00:00", "user_id": "sv01", "tokens_in": 84, "tokens_out": 45, "cost_usd": 3.96e-05}
> ```
>
> Từ `user_id` và `timestamp`, mình có thể lọc hoặc truy vết request của một user theo thời gian. Từ `tokens_in`, `tokens_out` và `cost_usd`, mình có thể tổng hợp mức dùng token/chi phí để theo dõi ngân sách. `print("đã trả lời xong")` không có các trường cấu trúc này nên khó lọc và thống kê tự động.

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
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 274 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage dùng base `python:3.11` đầy đủ và giữ toàn bộ môi trường cài đặt trong image cuối. Bản multi-stage dùng `python:3.11-slim` cho runtime, cài dependency ở stage `builder` rồi chỉ copy phần đã cài sang stage cuối. Vì vậy image cuối không mang theo các thành phần dư của base image đầy đủ và giảm từ 1.73 GB xuống 274 MB.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Theo Dockerfile trong repo, stage `builder` chỉ phụ thuộc `requirements.txt`, nên khi chỉ sửa `app/main.py`, layer cài dependency vẫn có thể dùng cache. Các layer copy source ở stage `runtime` và những lệnh sau layer bị đổi phải chạy lại. Nếu đưa `COPY . .` lên trước `pip install`, thay đổi ở bất kỳ file source nào cũng làm cache của bước cài dependency mất hiệu lực.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu lỗ hổng cho phép chạy lệnh, tiến trình bị chiếm có quyền của user chạy app. Chạy container bằng root khiến tiến trình đó có quyền cao trong container; nếu đồng thời có lỗi cấu hình hoặc lỗ hổng runtime cho phép thoát container thì rủi ro có thể lan tới host. Dockerfile của mình chuyển sang `USER appuser`, nên xâm nhập ứng dụng không tự động nhận quyền root trong container. Đây là giảm thiểu tác động, không phải bảo đảm tuyệt đối chống container escape.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng 2 giây: 10 request sát cuối một phút và 10 request ngay đầu phút tiếp theo. Fixed window đếm riêng mỗi phút nên cả hai đợt đều nằm trong hạn mức, dù tổng cộng có 20 request trong thời gian rất ngắn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong thời gian ngắn; cost guard giới hạn tiền đã dùng trong tháng. Một request đắt vẫn có thể nằm trong quota request nhưng bị cost guard chặn vì gần vượt ngân sách. Ngược lại, user còn ngân sách tháng nhưng gửi quá 10 request trong một phút sẽ bị rate limit trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis lỗi làm probe chung trả trạng thái unhealthy cho cả ba container, dù process ứng dụng vẫn chạy. Orchestrator có thể restart các container vì tưởng process chết; Redis vẫn lỗi nên probe tiếp tục fail và có thể gây restart lặp lại, đồng thời giảm số instance phục vụ. Tách `/health` khỏi `/ready` giúp health phản ánh process, còn readiness báo Redis mất kết nối để ngừng gửi traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Lần chạy `docker compose up -d --scale agent=3` của mình chưa tạo đủ replica: Compose báo `port is already allocated` khi replica thứ hai cùng bind host port `8000`, nên mình chưa có số đo `history_length` từ cụm ba instance. Theo code, history lưu ở Redis nên các instance dùng chung dữ liệu. Nếu thay bằng dict Python, mỗi process giữ dict riêng; request tới instance khác có thể không thấy các lượt trước và `history_length` sẽ thiếu hoặc thay đổi theo instance.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Deploy trên Render đã lên `Live`. Khi thử `POST /ask` từ PowerShell, mình nhận `422 Unprocessable Entity` với thông báo JSON `Expecting property name enclosed in double quotes`. Nội dung lỗi chỉ ra body gửi lên không phải JSON hợp lệ; đây là lỗi định dạng request khi kiểm tra service, không phải lỗi build hay khởi động Render. Sau đó bộ test CP5 cho request hợp lệ pass, xác nhận endpoint hoạt động.
