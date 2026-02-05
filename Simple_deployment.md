## Think of your project as 4 big layers:

- Application layer (FastAPI)

- Containerization layer (Docker)

- Kubernetes / OpenShift packaging (Helm)

- CI/CD automation (GitLab CI)

## 1️⃣ Repository structure – the big picture
```
MYAPP/
├── app/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
│
├── helm/
│   └── myapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-test.yaml
│       ├── values-prod.yaml
│       └── templates/
│           ├── _helpers.tpl
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── route.yaml
│           └── configmap.yaml (if added later)
│
└── .gitlab-ci.yml
```

## 👉 Why this split?

- app/ → what you are building

- helm/ → how it runs in Kubernetes/OpenShift

- .gitlab-ci.yml → how it is built & deployed automatically

This separation is a DevOps best practice.

## 2️⃣ Application layer (FastAPI)

### main.py
```
from fastapi import FastAPI
import os

app = FastAPI()

@app.get("/")
def home():
    return {
        "message": "Hello from OpenShift",
        "environment": os.getenv("ENV", "unknown")
    }
```

### What’s happening

- Creates a FastAPI web application

- Exposes `/` endpoint

- Reads environment variable ENV

#### Why environment variable?

Because:

- Same image runs in dev / test / prod

- Behavior changes via config, not code

- This follows 12-Factor App principles.

✅ Good practice

❌ Hard-coding env logic in code would be bad

### requirements.txt

```
fastapi
uvicorn
```
Why this file exists

- Lists Python dependencies

- Used by Docker during build

#### Better approaches

Pin versions for production:
```
fastapi==0.110.0
uvicorn[standard]==0.29.0
```

## 3️⃣ Containerization layer (Docker)

### Dockerfile
```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY main.py .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```
Explanation
```
FROM python:3.11-slim
```
Base image with Python

slim = smaller image, faster pulls
```
WORKDIR /app
```
All commands run inside /app
```
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```
Install dependencies first

Docker layer caching optimization
```
