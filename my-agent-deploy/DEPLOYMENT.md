# Deployment Information

> [!NOTE]
> Điền link URL public sau khi đã deploy thành công lên Railway.

## Public URL
https://2a202600846-nguyenhoangthanhtung-day12-agent-dep-production.up.railway.app

## Platform
Railway

## Test Commands

### 1. Health Check
```bash
curl https://2a202600846-nguyenhoangthanhtung-day12-agent-dep-production.up.railway.app/health
```
**Expected Response:**
```json
{
  "status": "ok",
  "version": "1.0.0",
  "environment": "development"
}
```

### 2. API Test (Không có Key - mong đợi lỗi 401)
```bash
curl https://2a202600846-nguyenhoangthanhtung-day12-agent-dep-production.up.railway.app/ask -X POST \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
```

### 3. API Test (Có Key - mong đợi thành công 200)
```bash
curl https://2a202600846-nguyenhoangthanhtung-day12-agent-dep-production.up.railway.app/ask -X POST \
  -H "X-API-Key: dev-key-change-me" \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
```

## Environment Variables Set
- `PORT`: `8080` (Cấp phát động bởi Railway)
- `ENVIRONMENT`: `development`
- `AGENT_API_KEY`: `dev-key-change-me`

## Screenshots
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
