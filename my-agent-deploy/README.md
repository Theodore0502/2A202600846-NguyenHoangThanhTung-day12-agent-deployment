# Production-Ready AI Agent — Day 12 Assignment

Dự án này là bài nộp bài tập Day 12 môn **AICB-P1 · VinUniversity 2026**. Mã nguồn này triển khai một AI Agent được đóng gói chuẩn sản xuất (production-ready) bằng FastAPI và Docker, tích hợp đầy đủ tính năng bảo mật, kiểm soát chi phí, giám sát trạng thái và triển khai trực tiếp lên cloud Railway.

---

## 🚀 Tính năng nổi bật (Production checklist)

- [x] **12-Factor App Configuration**: Cấu hình hoàn toàn từ biến môi trường thông qua `pydantic-settings`.
- [x] **Security & Authentication**: Xác thực bằng API Key (`X-API-Key`) trên Header của request.
- [x] **Rate Limiting**: Giới hạn số lượng request trên phút (10 req/phút mỗi API key) nhằm ngăn ngừa spam.
- [x] **Cost Guard (Kiểm soát ngân sách)**: Tính toán token tiêu thụ thực tế để giới hạn chi phí LLM hàng ngày (Daily Budget).
- [x] **Structured Logging**: Ghi log có cấu trúc dạng JSON, an toàn (không log API Key hay Secrets).
- [x] **Health Checks**: Tích hợp các đầu dò liveness (`/health`) và readiness (`/ready`) cho hệ thống.
- [x] **Graceful Shutdown**: Lắng nghe tín hiệu `SIGTERM` để kết thúc trọn vẹn các kết nối và request dang dở trước khi tắt container.
- [x] **Docker Multi-Stage Build**: Tối ưu hóa kích thước image từ 1.6 GB xuống chỉ còn **~236 MB** sử dụng base image `python:3.11-slim` sạch sẽ.
- [x] **Stateless Design & Load Balancing**: Dữ liệu lưu trữ tập trung (Redis) giúp dễ dàng nhân rộng (scale) qua Load Balancer (Nginx).

---

## 📁 Cấu Trúc Mã Nguồn

```text
day12-agent-deployment/
├── app/
│   ├── main.py         # Entry point - chứa logic API Gateway, Auth, Rate Limiter, Cost Guard & routing chính
│   └── config.py       # Load config & validate biến môi trường (12-Factor)
├── 2A202600846-NguyenHoangThanhTung-Day09/ # Thư mục chứa mã nguồn và dữ liệu của AI Shopping Assistant (Day 09)
├── utils/
│   └── mock_llm.py     # Giả lập trả lời của LLM (chạy test offline không cần API key)
├── Dockerfile          # Multi-stage Dockerfile tối ưu hóa dung lượng (hỗ trợ copy Day 09 Agent)
├── docker-compose.yml  # Định nghĩa cụm dịch vụ chạy local (Nginx LB + Agent Replicas + Redis)
├── railway.toml        # Cấu hình tự động triển khai trên nền tảng Railway
├── requirements.txt    # Danh sách thư viện Python phụ thuộc (FastAPI + LangGraph + ChromaDB)
├── .dockerignore       # Bỏ qua file rác khi build container
├── MISSION_ANSWERS.md  # [BÀI NỘP 1] - Câu trả lời cho các chặng lý thuyết của bài thực hành
└── DEPLOYMENT.md       # [BÀI NỘP 2] - Thông tin link triển khai public và kết quả test
```

---

## 🛠️ Hướng Dẫn Chạy Local

### 1. Chạy trực tiếp bằng Python Virtual Environment (`venv`)
Cài đặt thư viện và khởi chạy:
```powershell
# Khởi tạo và kích hoạt venv
python -m venv .venv
.venv\Scripts\Activate.ps1   # Trên Windows

# Cấu hình biến môi trường
Copy-Item .env.example .env

# Cài đặt thư viện
pip install -r requirements.txt

# Khởi chạy server
python app/main.py
```

### 2. Khởi chạy bằng Docker Compose (Cụm multi-replica)
```powershell
# Khởi động cụm Agent + Redis + Nginx Load Balancer
docker compose up -d

# Tắt và dọn dẹp tài nguyên
docker compose down
```

---

## 🧪 Cách Kiểm Tra (Test API)

Do sử dụng PowerShell trên Windows, khuyên dùng lệnh `Invoke-RestMethod` để tránh lỗi cú pháp JSON:

* **Kiểm tra sức khỏe hệ thống (Healthcheck):**
  ```powershell
  Invoke-RestMethod -Uri "http://localhost:8000/health"
  ```
* **Gửi request hỏi Agent (Có truyền API Key xác thực):**
  ```powershell
  Invoke-RestMethod -Uri "http://localhost:8000/ask" -Method Post -ContentType "application/json" -Headers @{"X-API-Key"="dev-key-change-me"} -Body '{"question": "What is Docker?"}'
  ```

---

## 🎯 Kiểm Định Chất Lượng (Production Readiness)

Dự án đi kèm công cụ kiểm định tự động giúp đảm bảo hệ thống đạt đủ điều kiện deploy:
```powershell
$env:PYTHONIOENCODING="utf-8"
python check_production_ready.py
```
**Kết quả mong đợi:** Đạt điểm tuyệt đối **20/20 checks passed (100%)**.

---

## 🌐 Triển khai lên Cloud (Triển khai thực tế)
Dự án được kết nối tự động với nền tảng **Railway** qua GitHub:
- **Public URL hoạt động:** [2a202600846-nguyenhoangthanhtung-day12-agent-dep-production.up.railway.app](https://2a202600846-nguyenhoangthanhtung-day12-agent-dep-production.up.railway.app)
- Vui lòng xem chi tiết log test cloud tại file **[DEPLOYMENT.md](DEPLOYMENT.md)**.
