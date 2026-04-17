#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Đặng Tiến Dũng 
> **Student ID:** 2A202600024  
> **Date:** 17-04-2026

---

### 3. Service Domain Link

Create a file `DEPLOYMENT.md` with your deployed service information:

```markdown
# Deployment Information

## Public URL
https://day12-06-production-e5fc.up.railway.app/

## Platform
Railway

## Test Commands

### Health Check
```bash
curl https://your-agent.railway.app/health

{
  "status": "ok",
  "version": "1.0.0",
  "environment": "development",
  "uptime_seconds": 3306.2,
  "total_requests": 2,
  "checks": {
    "llm": "mock"
  },
  "timestamp": "2026-04-17T14:32:09.036825+00:00"
}
```

### API Test (with authentication)
```bash
curl -X POST https://your-agent.railway.app/ask \
  -H "X-API-Key: YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test", "question": "Hello"}'

{
  "question": "hello",
  "answer": "Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.",
  "model": "gpt-4o-mini",
  "timestamp": "2026-04-17T13:37:18.426157+00:00"
}
```

## Environment Variables Set
```
HOST=0.0.0.0  
PORT=8000 
ENVIRONMENT=development  
DEBUG=true  
APP_NAME=AI Agent  
APP_VERSION=1.0
OPENAI_API_KEY=  
LLM_MODEL=gpt-4o-mini  

AGENT_API_KEY=dev-key-change-me-in-production  
```

## Screenshots
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
```

##  Pre-Submission Checklist

- [X] Repository is public (or instructor has access)
- [X] `MISSION_ANSWERS.md` completed with all exercises
- [X] `DEPLOYMENT.md` has working public URL
- [X] All source code in `app/` directory
- [X] `README.md` has clear setup instructions
- [X] No `.env` file committed (only `.env.example`)
- [X] No hardcoded secrets in code
- [X] Public URL is accessible and working
- [X] Screenshots included in `screenshots/` folder
- [X] Repository has clear commit history

---