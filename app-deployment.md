```
Developer → GitLab → CI/CD → OpenShift
                    |
                    ├── dev namespace
                    ├── test namespace
                    └── prod namespace
```
- One Helm chart

- Different values per environment

- GitLab CI controls promotion

- Prod protected

  Repository Structure (Production-ready)

  ```
  myapp/
├── app/
│   ├── main.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── helm/
│   └── myapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-test.yaml
│       ├── values-prod.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── route.yaml
│           ├── configmap.yaml
│           └── _helpers.tpl
│
├── .gitlab-ci.yml
└── README.md
```

app/main.py

```
from fastapi import FastAPI
import os

app = FastAPI()

ENV = os.getenv("ENV", "unknown")

@app.get("/")
def read_root():
    return {
        "message": "Hello from OpenShift!",
        "environment": ENV
    }

```
app/requirements.txt

```
fastapi
uvicorn
```

app/Dockerfile:

```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY main.py .

EXPOSE 8080

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```
helm/myapp/Chart.yaml
```
apiVersion: v2
name: myapp
description: Sample production-ready app
type: application
version: 0.1.0
appVersion: "1.0.0"
```
helm/myapp/values.yaml
```
