# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.

> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).

> Họ và tên: Lê Công Dũng         Mã học viên: 2A202601649

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để giá trị mặc định là `"changeme"`, ứng dụng vẫn sẽ khởi động thành công và âm thầm chạy trên production, dẫn đến việc các request gọi đến LLM bị từ chối ngầm hoặc dùng sai khóa, khiến bạn chỉ phát hiện ra sự cố khi nhận được hóa đơn bất thường. Việc không có giá trị mặc định buộc app chết ngay từ lúc khởi động nếu thiếu cấu hình, giúp ta phát hiện sai sót từ sớm trước khi đưa lên cloud.
> 
> 

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log mẫu: `{"event": "ask_completed", "level": "info", "timestamp": "2026-08-01T09:30:00+00:00", "user_id": "sv01", "cost_usd": 0.0001}`.
> Hai việc làm được:
> 
> 
> 1. Trả lời được câu hỏi *"user nào tiêu nhiều tiền nhất hôm nay?"* bằng cách truy vấn và tổng hợp các trường dữ liệu cấu trúc.
> 
> 
> 2. Tính toán chính xác tỷ lệ lỗi hoặc thời gian phản hồi trong 5 phút qua mà lệnh `print()` thông thường không thể phân tách hay thống kê tự động.
> 
> 
> 

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent

```

| Bản | Dung lượng |
| --- | --- |
| 1 stage (bản đầu) | ~1000 MB

 |
| Multi-stage | ~200 MB

 |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch chính là các công cụ biên dịch (`build-essential`), tệp cache của trình quản lý gói, và các thư viện phát triển chỉ cần thiết trong quá trình cài đặt nhưng bị loại bỏ ở multi-stage build, giúp image giảm từ khoảng ~1GB xuống ~200MB.
> 
> 

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Các layer cài đặt thư viện phía trên (`RUN pip install`) được dùng lại từ cache, chỉ layer copy code và các lệnh phía sau chạy lại. Nếu đặt `COPY . .` lên trước `RUN pip install`, mỗi lần bạn sửa dù chỉ một ký tự trong code, toàn bộ cache sẽ bị hủy và Docker phải cài lại toàn bộ thư viện từ đầu.
> 
> 

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu container chạy bằng quyền root và ứng dụng dính lỗ hổng thực thi mã độc từ xa, kẻ tấn công có thể chiếm quyền root bên trong container, từ đó leo thang đặc quyền để kiểm soát toàn bộ hệ thống máy chủ host bên ngoài. Lệnh `USER appuser` tạo ra một người dùng có quyền hạn bị giới hạn, cắt đứt chuỗi nguy hiểm này ngay từ gốc.
> 
> 

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Một người dùng có thể gửi tối đa **20 request** trong 2 giây liên tiếp (10 request vào giây thứ 59 của phút trước và 10 request vào giây thứ 00 của phút sau). Giải thích: Do cơ chế đếm theo phút đồng hồ sẽ reset bộ đếm về 0 ngay khi sang phút mới, tạo ra kẽ hở cho phép dồn dập gửi gấp đôi hạn mức ở khoảng thời gian giáp ranh.
> 
> 

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> * Khác biệt: Rate limit giới hạn tốc độ và số lượng request trong khoảng thời gian, trong khi cost guard giới hạn tổng chi phí tài chính (token).
> 
> 
> * Tình huống rate limit cho qua nhưng cost guard chặn: User gửi 1 request duy nhất trong phút nhưng câu hỏi chứa cực kỳ nhiều token, không vi phạm số lượng request nhưng vượt ngân sách tháng.
> 
> 
> * Tình huống ngược lại: User gửi liên tục 15 request ngắn (vượt rate limit 10/phút) nhưng mỗi câu hỏi cực kỳ ngắn và rẻ, chưa chạm ngưỡng ngân sách tháng.
> 
> 
> 
> 

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Khi Redis mất kết nối, cả 3 container đồng thời báo unhealthy trên endpoint gộp $\rightarrow$ Orchestrator (Kubernetes/Docker) hiểu lầm rằng toàn bộ ứng dụng đã hỏng và tiến hành **restart đồng thời cả 3 container** $\rightarrow$ Hậu quả là làm sập toàn bộ hệ thống thay vì chỉ từ chối nhận traffic tạm thời, biến sự cố kết nối nhỏ thành thời gian chết toàn cục.
> 
> 

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lưu trong một dict Python cục bộ, mỗi container `agent` sẽ quản lý RAM riêng. Do load balancer phân phối ngẫu nhiên request sang các container khác nhau, bạn sẽ thấy `history_length` nhảy cóc hỗn loạn hoặc không tăng đều (ví dụ: 1 -> 1 -> 2 -> 1), vì container hiện tại không biết lịch sử mà container trước đã lưu.
> 
> 

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> * Lỗi gặp phải: Khi triển khai dự án lên Render qua Blueprint, quá trình sync thất bại với thông báo lỗi không thể tạo instance Redis do vượt quá giới hạn 1 instance Key Value miễn phí trên tài khoản (cannot have more than 1 free tier Key Value instance), dẫn đến việc web service bị hủy tạo.  
> 
> 
> * Cách tìm nguyên nhân: Kiểm tra chi tiết trạng thái lỗi trong phần "Syncs" và nhật ký log trực tiếp trên giao diện Dashboard của Render.
> 
> * Cách sửa: Truy cập vào mục Resources trên Render để xóa bỏ instance Redis cũ không còn sử dụng nhằm giải phóng hạn mức miễn phí, sau đó bấm nút "Manual sync" để hệ thống tiến hành tạo lại từ đầu thành công.  
>
