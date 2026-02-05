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
COPY main.py .
```
Copy application code

```
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```
- Starts the app

- Listens on 8080 (important for K8s)

### Why Docker?

- Same runtime everywhere

- No “works on my machine” issues

- Required for Kubernetes

### Better approaches

- Use multi-stage builds for larger apps

- Use non-root user for security

- Add healthcheck

## 4️⃣ Helm layer (Kubernetes/OpenShift)
Helm = package manager for Kubernetes

Instead of writing raw YAML per environment, you:

- Template it

- Inject values dynamically  

### Chart.yaml
```
apiVersion: v2
name: myapp
version: 0.1.0
```
Purpose

- Metadata about the Helm chart

- Helm uses this to identify the app

- Think of this like package.json for Kubernetes.

### values.yaml (default values)
```
replicaCount: 1

image:
  repository: ""
  tag: ""

env:
  ENV: default

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
```
Why this exists

- Central config file

- Templates read from .Values

## Environment-specific values
### values-dev.yaml
```
env:
  ENV: dev
```
### values-test.yaml
```
replicaCount: 2
env:
  ENV: test
```
### values-prod.yaml
```
replicaCount: 3
env:
  ENV: prod
```
Why this approach?

- Same chart

- Different behavior per environment

- No duplication of YAML

✅ Very good practice

## 5️⃣ Helm templates

### _helpers.tpl
```
{{- define "myapp.name" -}}
myapp
{{- end }}

{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ include "myapp.name" . }}
{{- end }}
```
Why helpers?

- Avoid repetition

- Consistent naming everywhere

Example:
```
Release name: myapp-dev
Result: myapp-dev-myapp
```
### deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.name" . }}
    spec:
      containers:
        - name: myapp
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 8080
          env:
            - name: ENV
              value: {{ .Values.env.ENV | quote }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```
```
replicas: {{ .Values.replicaCount }}
```
→ Scales app per environment
```
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```
→ Injected by CI pipeline
```
env:
  - name: ENV
    value: {{ .Values.env.ENV | quote }}
```
→ Passes ENV to FastAPI


```
resources:
  {{- toYaml .Values.resources | nindent 12 }}
```
→ Resource requests (important for cluster stability)
