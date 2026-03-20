# 🚀 Distributed Assessment Engine Demo

## 📌 Project Title - Assessment Engine and Candidate Execution Framework

## 📖 Project Overview
This project demonstrates Assessment Engine and Candidate Execution Framework using FastAPI, Celery, and Redis. It allows candidates to start sessions, answer questions, and track task execution asynchronously.

---

## 🏗️ Architecture
- **FastAPI** → REST API server for candidate sessions.  
- **Celery** → Task queue for asynchronous question processing.  
- **Redis** → Message broker for Celery workers.  
- **Docker** → Simplifies Redis setup.  

**Flow:**  
Candidate → FastAPI → Celery Task → Redis Queue → Worker → Result → FastAPI Response.

---

## 📂 Project Structure
```
project-root/
│── app/
│   ├── main.py          # FastAPI entrypoint
│   ├── models.py        # Data models
│   ├── routes.py        # API routes
│   └── tasks.py         # Celery tasks
│── celery_app/
│   └── celery_app.py    # Celery configuration
│── requirements.txt
│── README.md
```

---

## 🛠️ Technologies Used
- Python 3.10+  
- FastAPI  
- Celery  
- Redis  
- Docker  
- Uvicorn  

---

## 🔄 Workflow
1. Candidate creates a session.  
2. Candidate starts a question → triggers Celery task.  
3. Task runs asynchronously in worker.  
4. Candidate checks task status via API.  
5. Session can be recovered if interrupted.  

---

## ▶️ How to Run (Step-by-Step)

### 1. Install Redis
```bash
docker run -d --name assessment-redis -p 6379:6379 redis:7
```
Or install Redis locally and ensure it runs on `localhost:6379`.

### 2. Setup Python Environment
```bash
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Start Celery Worker
```bash
celery -A celery_app.celery_app worker --loglevel=info -Q celery --concurrency=4
```
Or simply:
```bash
celery -A celery_app.celery_app worker --loglevel=info
```

### 4. Start FastAPI Server
```bash
uvicorn app.main:app --reload
```

### 5. Test with curl
```bash
# Create session
curl -X POST "http://127.0.0.1:8000/sessions" \
     -H "Content-Type: application/json" \
     -d '{"candidate_id": "cand_123"}'

# Start a question
curl -X POST "http://127.0.0.1:8000/sessions/<session_id>/start_question" \
     -H "Content-Type: application/json" \
     -d '{"question_id":"q1","payload":{"prompt":"Answer this"}}'

# Check task status
curl "http://127.0.0.1:8000/tasks/<task_id>"
```

---

## 🧪 Testing
- **Timeout Behavior** → Modify `sleep_seconds` in `app/tasks.py` (>1800) to trigger `SoftTimeLimitExceeded`.  
- **Session Recovery** →  
```bash
curl -X POST "http://127.0.0.1:8000/sessions/<session_id>/recover"
```
---

## 📊 Output
- **Session Creation Response** → JSON with `session_id`.  
- **Task Status Response** → JSON with `task_id` and status (`PENDING`, `STARTED`, `SUCCESS`, `FAILURE`).  
- **Recovered Session** → JSON with previous attempts.  


## ⚠️ Notes for Production
- Use **Postgres** instead of in-memory DB.  
- Run migrations.  
- Use a process manager for multiple Celery workers.  
- Add authentication, rate limiting, and input validation.  
- Consider containerization and sandboxing for per-candidate isolation.  
