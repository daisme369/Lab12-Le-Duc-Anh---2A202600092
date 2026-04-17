#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Lê Đức Anh  
> **Student ID:** 2A202600092  
> **Date:** 17/4/2026

---
# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1.API key hardcode trong code -> Nếu push lên GitHub → key bị lộ ngay lập tức
2. Vấn đề 2: Không có config management
3.  Vấn đề 3: Print thay vì proper logging
4. Vấn đề 4: Không có health check endpoint ->Nếu agent crash, platform không biết để restart
5. Port cố định — không đọc từ environment. Trên Railway/Render, PORT được inject qua env var
...

### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| Config / Secrets | Hardcode trong code | Đọc từ Environment variables | Bảo mật (không lọt key lên GitHub), dễ thay đổi theo môi trường (Tách biệt code và config). |
| Logging | Dùng lệnh `print()` | Structured JSON logging | Dễ truy xuất lỗi, có format chuẩn cho log aggregator, tránh in nhầm secret. |
| Host Binding | `localhost` | `0.0.0.0` | `0.0.0.0` cho phép app nhận traffic từ bên ngoài khi chạy trong Docker/Cloud. |
| Port | Hardcode port `8000` | Đọc từ biến `PORT` | Trên Cloud (Railway, Render), port được gán tự động động (dynamic). |
| Health Check | Không có | Có các endpoint `/health`, `/ready` | Giúp Cloud platform biết khi nào app crash để tự động restart. |
| Lifecycle | Bị ngắt ngang lập tức | Graceful shutdown (với `lifespan`) | Chờ hoàn thành nốt các request đang dang dở trước khi tắt app hoàn toàn. |

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. Base image: Là môi trường cài sẵn các thư viện cần thiết để chạy ứng dụng từ provider thay vì phải cài đặt thử công
2. Working directory: Directory gốc chứa code của ứng dụng
3. Cần copy requirements trước vì: 
  - Để khi pip install có requirements 
  - Sử dụng cache của Docker để lưu được short memory, thay vì mỗi khi chạy app phải caiaf lại
4. CMD và ENTRYPOINT: 
  - CMD: Lệnh mặc định sẽ chạy khi container khởi động
  - ENTRYPOINT: Lệnh sẽ được chạy khi container khởi động
...

### Exercise 2.3: Image size comparison
- Develop: 1.66GB
- Production: 236 MB
- Difference: 85.78%

### Exercise 2.4:  Docker Compose stask


## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://lab12-production-b776.up.railway.app
- Screenshot: [Link to screenshot in repo]

## Part 4: API Security
- Trong `develop/app.py`:
  - API Key được client gửi lên thông qua HTTP Header tên là X-API-Key.
  - Việc kiểm tra diễn ra bên trong hàm verify_api_key
  - Hàm này được tiêm (inject) dưới dạng một Dependency (Depends(verify_api_key)) vào endpoint cần bảo vệ là POST /ask

  Nếu sai key:
   - Nếu không truyền key (thiếu header X-API-Key): Sẽ bị trả về mã lỗi HTTP 401 Unauthorized kèm thông báo "Missing API key. Include header: X-API-Key: <your-key>".
   - Nếu truyền sai key (key không khớp): Sẽ bị ngắt và trả về mã lỗi HTTP 403 Forbidden kèm thông báo "Invalid API key."

  - Để rotate key:
    Cập nhật lại giá trị biến môi trường AGENT_API_KEY trên server.
    Khởi động lại (Restart) ứng dụng/container FastAPI thì nó mới nhận giá trị key mới.

### Exercise 4.1-4.3: Test results
 - **4.1**
{
    "detail": "Missing API key. Include header: X-API-Key: <your-key>"
}

{
    "detail": "Invalid API key."
}

 - **4.2**
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzdHVkZW50Iiwicm9sZSI6InVzZXIiLCJpYXQiOjE3NzY0Mzk2ODQsImV4cCI6MTc3NjQ0MzI4NH0.msCCKhuVvJeJ2CjwjX036H73vYCbl4m4Vy5GfvCpv8E",
    "token_type": "bearer",
    "expires_in_minutes": 60,
    "hint": "Include in header: Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
}

{
    "question": "Explain JWT",
    "answer": "Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.",
    "usage": {
        "requests_remaining": 9,
        "budget_remaining_usd": 1.6e-05
    }
}

 - **4.3**
- **Algorithm:** Sliding Window Counter
- **Limit:** Student (10 req/min), Teacher (100 req/min)
- **Bypass limit cho admin:** Thay vì bypass, cấp thêm limitation cao hơn cho admin/ teacher

### Exercise 4.4: Cost guard implementation
[Explain your approach]

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
[Your explanations and test results]
```

---

### 2. Full Source Code - Lab 06 Complete (60 points)

Your final production-ready agent with all files:

```
your-repo/
├── app/
│   ├── main.py              # Main application
│   ├── config.py            # Configuration
│   ├── auth.py              # Authentication
│   ├── rate_limiter.py      # Rate limiting
│   └── cost_guard.py        # Cost protection
├── utils/
│   └── mock_llm.py          # Mock LLM (provided)
├── Dockerfile               # Multi-stage build
├── docker-compose.yml       # Full stack
├── requirements.txt         # Dependencies
├── .env.example             # Environment template
├── .dockerignore            # Docker ignore
├── railway.toml             # Railway config (or render.yaml)
└── README.md                # Setup instructions
```

**Requirements:**
-  All code runs without errors
-  Multi-stage Dockerfile (image < 500 MB)
-  API key authentication
-  Rate limiting (10 req/min)
-  Cost guard ($10/month)
-  Health + readiness checks
-  Graceful shutdown
-  Stateless design (Redis)
-  No hardcoded secrets

---

### 3. Service Domain Link

Create a file `DEPLOYMENT.md` with your deployed service information:

```markdown
# Deployment Information

## Public URL
https://your-agent.railway.app

## Platform
Railway / Render / Cloud Run

## Test Commands

### Health Check
```bash
curl https://your-agent.railway.app/health
# Expected: {"status": "ok"}
```

### API Test (with authentication)
```bash
curl -X POST https://your-agent.railway.app/ask \
  -H "X-API-Key: YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test", "question": "Hello"}'
```

## Environment Variables Set
- PORT
- REDIS_URL
- AGENT_API_KEY
- LOG_LEVEL

## Screenshots
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
```

##  Pre-Submission Checklist

- [ ] Repository is public (or instructor has access)
- [ ] `MISSION_ANSWERS.md` completed with all exercises
- [ ] `DEPLOYMENT.md` has working public URL
- [ ] All source code in `app/` directory
- [ ] `README.md` has clear setup instructions
- [ ] No `.env` file committed (only `.env.example`)
- [ ] No hardcoded secrets in code
- [ ] Public URL is accessible and working
- [ ] Screenshots included in `screenshots/` folder
- [ ] Repository has clear commit history

---

##  Self-Test

Before submitting, verify your deployment:

```bash
# 1. Health check
curl https://your-app.railway.app/health

# 2. Authentication required
curl https://your-app.railway.app/ask
# Should return 401

# 3. With API key works
curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
  -X POST -d '{"user_id":"test","question":"Hello"}'
# Should return 200

# 4. Rate limiting
for i in {1..15}; do 
  curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
    -X POST -d '{"user_id":"test","question":"test"}'; 
done
# Should eventually return 429
```

---

##  Submission

**Submit your GitHub repository URL:**

```
https://github.com/your-username/day12-agent-deployment
```

**Deadline:** 17/4/2026

---

##  Quick Tips

1.  Test your public URL from a different device
2.  Make sure repository is public or instructor has access
3.  Include screenshots of working deployment
4.  Write clear commit messages
5.  Test all commands in DEPLOYMENT.md work
6.  No secrets in code or commit history

---

##  Need Help?

- Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Review [CODE_LAB.md](CODE_LAB.md)
- Ask in office hours
- Post in discussion forum

---

**Good luck! **
