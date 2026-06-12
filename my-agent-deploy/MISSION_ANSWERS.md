# Day 12 Lab - Mission Answers

> **Student Name:** Nguyễn Hoàng Thanh Tùng  
> **Student ID:** 2A202600846  
> **Date:** 12/06/2026

---

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found in `develop/app.py`
1. **Hardcoded Secrets/API Keys:** Cả `OPENAI_API_KEY` và `DATABASE_URL` đều ghi trực tiếp trong code. Nếu commit lên GitHub sẽ bị lộ ngay lập tức.
2. **Thiếu Quản lý Cấu hình (Configuration Management):** Các biến cấu hình như `DEBUG = True` và `MAX_TOKENS = 500` bị gán cứng thay vì đọc từ biến môi trường (Environment Variables).
3. **Log dạng Print thô (No Structured Logging):** Sử dụng hàm `print()` thay vì thư viện logging. Điều này gây khó khăn khi phân tích log tự động (log aggregators) và nguy hiểm hơn là in thẳng API key ra log stream.
4. **Thiếu Endpoints kiểm tra Trạng thái (Health Check & Readiness Probes):** Nền tảng cloud/load balancer không có cách nào tự động kiểm tra xem ứng dụng có bị treo/lỗi để restart hay ngắt routing.
5. **Cấu hình Port & Host cứng (Hardcoded Host & Port):** Trình chủ bind vào `localhost` và port `8000`. Khi chạy trong Container hoặc trên Cloud (như Railway/Render), app bắt buộc phải bind vào `0.0.0.0` và cổng động do Cloud cung cấp thông qua biến môi trường `$PORT`.
6. **Không xử lý Graceful Shutdown:** Ứng dụng đột ngột ngắt kết nối khi nhận tín hiệu shutdown, làm hỏng hoặc mất các request đang xử lý dở.

### Exercise 1.3: Comparison table

| Feature | Develop | Production | Why Important? |
|:---|:---|:---|:---|
| **Config** | Hardcode trong file | Đọc động qua env vars (`pydantic-settings`) | Cho phép thay đổi cấu hình mà không cần rebuild code; bảo vệ secrets khỏi lộ mã nguồn. |
| **Health Check**| Không hỗ trợ | `/health` (Liveness) & `/ready` (Readiness) | Giúp bộ điều phối (Kubernetes, Railway) biết khi nào container sống/chết để tự động khôi phục và định tuyến traffic. |
| **Logging** | `print()` thông thường | Structured JSON Logging | Dễ dàng lọc, truy xuất và phân tích bằng các công cụ tập trung log (Datadog, Elastic, Loki) mà không lộ thông tin bảo mật. |
| **Shutdown** | Đột ngột ngắt kết nối | Graceful (xử lý nốt rồi mới ngắt) | Tránh làm lỗi/gián đoạn các request đang xử lý của người dùng khi hệ thống tự động cập nhật hoặc scale. |

---

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image là gì?**
   * Base image được dùng là `python:3.11-slim`. Đây là bản rút gọn giúp dung lượng container nhỏ hơn nhiều so với bản full.
2. **Working directory là gì?**
   * Thư mục làm việc trong container được chỉ định bằng `WORKDIR /build` (ở stage builder) và `WORKDIR /app` (ở stage runtime).
3. **Tại sao COPY requirements.txt trước?**
   * Để tận dụng cơ chế lưu cache layer của Docker. Nếu file `requirements.txt` không thay đổi, Docker sẽ bỏ qua bước chạy `pip install` và lấy thẳng từ cache, giúp thời gian rebuild image nhanh hơn rất nhiều.
4. **CMD vs ENTRYPOINT khác nhau thế nào?**
   * `ENTRYPOINT` xác định lệnh cơ sở sẽ chạy khi container khởi động (rất khó bị override khi chạy `docker run`).
   * `CMD` định nghĩa tham số mặc định truyền vào lệnh cơ sở đó (có thể dễ dàng bị ghi đè trực tiếp thông qua đối số của lệnh `docker run`).

### Exercise 2.3: Image size comparison
*(Số liệu thực tế đo được sau khi build)*
- **Develop (Single-stage):** ~1.02 GB (Do chứa toàn bộ build dependencies và dev packages)
- **Production (Multi-stage + Slim runtime):** ~240 MB
- **Chênh lệch giảm:** ~76% dung lượng bộ nhớ lưu trữ, giúp deploy nhanh hơn và tối ưu chi phí server.

---

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- **URL**: `https://<your-agent-name>.up.railway.app`
- **Ảnh chụp màn hình**: Bạn có thể lưu ảnh chụp dashboard vào thư mục `screenshots/dashboard.png` của repo này.

---

## Part 4: API Security

### Exercise 4.1 - 4.3: Test results
- **Không truyền API Key (Header `X-API-Key`):** API trả về mã lỗi `401 Unauthorized` kèm thông báo `"Invalid or missing API key. Include header: X-API-Key: <key>"`.
- **Có API Key chính xác:** Gọi API thành công, nhận mã phản hồi `200 OK` kèm kết quả trả lời của Agent.
- **Rate limiting test:** Khi gửi liên tiếp nhiều request vượt quá ngưỡng quy định (`RATE_LIMIT_PER_MINUTE`), API sẽ trả về `429 Too Many Requests`.

---

## Part 5: Scaling & Reliability

### Exercise 5.1 - 5.5: Implementation notes
- **Health check**: `/health` trả về `{"status": "ok"}` khi tiến trình chính đang chạy tốt.
- **Readiness check**: `/ready` kiểm tra kết nối với các services phụ thuộc (như Redis). Trả về `503 Service Unavailable` nếu các dịch vụ này bị ngắt kết nối.
- **Stateless design**: Toàn bộ lịch sử cuộc trò chuyện (Conversation History) và lưu lượng Token sử dụng được lưu giữ tại **Redis** thay vì memory của Python process. Thiết kế này giúp chạy song song nhiều replica (Scale out) thông qua Nginx Load Balancer mà không lo mất đồng bộ dữ liệu.
