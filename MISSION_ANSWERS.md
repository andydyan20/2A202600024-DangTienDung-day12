#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Đặng Tiến Dũng 
> **Student ID:** 2A202600024  
> **Date:** 17-04-2026

---

##  Submission Requirements

Submit a **GitHub repository** containing:

### 1. Mission Answers (40 points)

Create a file `MISSION_ANSWERS.md` with your answers to all exercises:

# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
- API key hardcode
- Port cố định
- Debug mode
- Không có health check
- Không xử lý shutdown

### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| Config | Hardcode | Env vars | Tránh lộ thông tin (API key, password..), Mỗi môi trường cần config khác nhau, dễ dàng thay đổi |
| Health check | Không có | Có | Kiểm tra service còn sống không |
| Logging | print() | JSON | Unify format, lưu log theo 1 structure để tool minitor truy xuất (kibana) |
| Shutdown | Đột ngột | Graceful | Các request không bị ngắt đột ngột làm lost data, lỗi luồng, khó khăn cho việc BAU sau này (xử lý lại request lỗi) |

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. Base image là gì?
- Môi trường base cơ bản, thường là bản slim hoặc alpine
2. Working directory là gì?
- Là root directory của ứng dụng
3. Tại sao COPY requirements.txt trước?
- Để cache lại file đấy sau khi build lần đầu
- Cần để trước layer pip install
4. CMD vs ENTRYPOINT khác nhau thế nào?
- CMD có thể bị ghi đề, ENTRYPOINT chỉ có thể nối thêm tham số

### Exercise 2.3: Image size comparison
- Develop: 1670 MB
- Production: 262 MB
- Difference [Production/Develop]: 15.7%

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://pacific-communication-production-db01.up.railway.app/docs
- Screenshot: [Demo](screenshots/lab03_railway.png)

## Part 4: API Security

### Exercise 4.1-4.3: Test results
1. API key được check ở đâu?
- Check trong header với mỗi request.
2. Điều gì xảy ra nếu sai key?
- Trả về lỗi 401 Unauthorized
3. Làm sao rotate key?
- Sử dụng config token, hoặc jwt token tự sinh với secret config có expired time

### Exercise 4.4: Cost guard implementation
```python
import redis
from datetime import datetime

r = redis.Redis()

def check_budget(user_id: str, estimated_cost: float) -> bool:
    month_key = datetime.now().strftime("%Y-%m")
    key = f"budget:{user_id}:{month_key}"
    
    current = float(r.get(key) or 0)
    if current + estimated_cost > 10:
        return False
    
    r.incrbyfloat(key, estimated_cost)
    r.expire(key, 32 * 24 * 3600)  # 32 days
    return True
```

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes

log nhận được:

```bash
{"ts":"2026-04-17 13:32:02,024","lvl":"INFO","msg":"{"event": "ready"}"}
INFO:     Application startup complete.

{"ts":"2026-04-17 13:32:02,901","lvl":"INFO","msg":"{"event": "request", "method": "GET", "path": "/health", "status": 200, "ms": 1.0}"}
INFO:     100.64.0.2:50531 - "GET /health HTTP/1.1" 200 OK

```

implementation:

```python
@app.get("/health")
def health():
    """Liveness probe — container còn sống không?"""
    uptime = round(time.time() - START_TIME, 1)

    # Kiểm tra dependencies quan trọng
    checks = {}

    # Check memory
    try:
        import psutil
        mem = psutil.virtual_memory()
        checks["memory"] = {
            "status": "ok" if mem.percent < 90 else "degraded",
            "used_percent": mem.percent,
        }
    except ImportError:
        checks["memory"] = {"status": "ok", "note": "psutil not installed"}

    overall_status = "ok" if all(
        v.get("status") == "ok" for v in checks.values()
    ) else "degraded"

    return {
        "status": overall_status,
        "uptime_seconds": uptime,
        "version": "1.0.0",
        "environment": os.getenv("ENVIRONMENT", "development"),
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "checks": checks,
    }

@app.get("/ready")
def ready():
    """Readiness probe — sẵn sàng nhận traffic không?"""
    if not _is_ready:
        raise HTTPException(
            status_code=503,
            detail="Agent not ready. Check back in a few seconds.",
        )
    return {
        "ready": True,
        "in_flight_requests": _in_flight_requests,
    }


