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
```
kind: Deployment
```
- Meaning: This tells Kubernetes the type of resource.

- Why needed: K8s needs to know if this is a Deployment, Service, ConfigMap, etc.

- Other options: StatefulSet, DaemonSet, Job, etc. depending on the workload.

```
metadata:
  name: {{ include "myapp.fullname" . }}
```
- Meaning: Name of the Deployment in Kubernetes.

- include → calls _helpers.tpl template "myapp.fullname" with current context .

- Why needed: Ensures a unique name per release, avoids collisions.

- Production tip: Always use fullname or a combination of release name + chart name. Avoid hardcoding names.

```
spec:
  replicas: {{ .Values.replicaCount }}
```
- Meaning: Number of pod replicas to run.

- .Values.replicaCount → value from values.yaml

- Why needed: Controls scaling of your app.

- Production tip: Consider using HPA (Horizontal Pod Autoscaler) instead of fixed replica count.

```
selector:
  matchLabels:
    app: {{ include "myapp.name" . }}
```
- Meaning: K8s uses this to select which pods belong to this deployment.

- Why needed: Must match the pod template labels (metadata.labels) for Deployment to manage pods.

- Production tip: Always keep selector labels minimal and stable. Changing them later can break upgrades.

```
  template:
  metadata:
    labels:
      app: {{ include "myapp.name" . }}
```
- Meaning: Labels for pods created by this deployment.

- Why needed: Must match selector, helps service discovery and monitoring tools.

- Production tip: Add more labels for environments, team, version:

```
labels:
  app: {{ include "myapp.name" . }}
  env: {{ .Values.envName | default "prod" }}
  version: {{ .Chart.AppVersion }}
```
```
spec:
  containers:
    - name: myapp
      image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```
- name → container name in pod.

- image → Docker image to run.

- {{ .Values.image.repository }}:{{ .Values.image.tag }} → values from values.yaml, makes it configurable.

- Production tip: Use digest instead of tag in prod for immutable deployments:

```
  image: "myrepo/myapp@sha256:abc123..."
```

```
ports:
  - containerPort: 8080
```
- Meaning: Exposes port inside container.

- Why needed: K8s needs to know which ports the container uses.

- Production tip: Usually matches the port your app listens on. Can add multiple ports if needed.

```
  env:
  - name: ENV
    value: {{ .Values.env.ENV | quote }}
```
- Meaning: Sets environment variable ENV in container.

- quote → ensures the value is wrapped in double quotes, even if it contains special characters.

- Why needed: Prevents YAML errors for values like true, 1, or strings with spaces.

- Production tip: Use ConfigMaps/Secrets for sensitive data instead of hardcoding values.

Example using Secret:
```
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```
```
resources:
  {{- toYaml .Values.resources | nindent 12 }}
```
- Meaning: Assign CPU/memory limits and requests.

- toYaml → converts .Values.resources (from values.yaml) to proper YAML.

- nindent 12 → adds 12 spaces of indentation to the generated YAML so it fits correctly under resources:

- Why needed: Kubernetes requires indentation to parse YAML correctly.

- Production tip: Always set requests & limits for production pods to prevent cluster resource issues.

```
  resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
  requests:
    cpu: "250m"
    memory: "256Mi"
```

## ✅ Optional production-ready enhancements

Readiness & Liveness probes (for automatic restarts if container fails)
```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 15
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```
Affinity & nodeSelector (for pod placement)
```
nodeSelector:
  node-type: app-node
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - myapp
        topologyKey: "kubernetes.io/hostname"
```
SecurityContext (run as non-root)
```
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
```
ServiceAccount (for RBAC)
```
serviceAccountName: myapp-sa
```
Probes + Resources + Env + ConfigMaps + Secrets = production-ready deployment ✅
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "myapp.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.name" . }}
        release: {{ .Release.Name }}
    spec:
      serviceAccountName: {{ include "myapp.serviceAccountName" . }}
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
          env:
            # Environment variables from values.yaml
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}
            # Secrets
            {{- range $key, $secret := .Values.secrets }}
            - name: {{ $key }}
              valueFrom:
                secretKeyRef:
                  name: {{ $secret.name }}
                  key: {{ $secret.key }}
            {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          livenessProbe:
            httpGet:
              path: {{ .Values.liveness.path }}
              port: {{ .Values.liveness.port }}
            initialDelaySeconds: {{ .Values.liveness.initialDelaySeconds }}
            periodSeconds: {{ .Values.liveness.periodSeconds }}
            timeoutSeconds: {{ .Values.liveness.timeoutSeconds }}
          readinessProbe:
            httpGet:
              path: {{ .Values.readiness.path }}
              port: {{ .Values.readiness.port }}
            initialDelaySeconds: {{ .Values.readiness.initialDelaySeconds }}
            periodSeconds: {{ .Values.readiness.periodSeconds }}
            timeoutSeconds: {{ .Values.readiness.timeoutSeconds }}
      nodeSelector:
        {{- toYaml .Values.nodeSelector | nindent 8 }}
      affinity:
        {{- toYaml .Values.affinity | nindent 8 }}
      tolerations:
        {{- toYaml .Values.tolerations | nindent 8 }}
```
Sample values.yaml for this deployment
```
replicaCount: 2

image:
  repository: myrepo/myapp
  tag: "1.4.0"
  pullPolicy: IfNotPresent

service:
  port: 8080

env:
  ENV: prod
  LOG_LEVEL: info

secrets:
  DB_PASSWORD:
    name: myapp-secret
    key: password

resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
  requests:
    cpu: "250m"
    memory: "256Mi"

liveness:
  path: /healthz
  port: 8080
  initialDelaySeconds: 10
  periodSeconds: 15
  timeoutSeconds: 5

readiness:
  path: /ready
  port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 5

nodeSelector: {}

affinity: {}

tolerations: []
```
```
GitLab Variables
   ↓
GitLab CI/CD
   ↓
kubectl create secret
   ↓
Kubernetes Secret (etcd)
   ↓
Deployment.yaml (secretKeyRef)
   ↓
Pod Environment Variable
   ↓
Application reads ENV
```
### Summary — WHEN to use WHAT
| Data type      | Where stored   | How injected   |
| -------------- | -------------- | -------------- |
| ENV, LOG_LEVEL | values.yaml    | Helm env.value |
| DB password    | GitLab CI/CD   | K8s Secret     |
| API tokens     | GitLab CI/CD   | K8s Secret     |
| Certificates   | Secret / Vault | volume mount   |

STEP 1: Store secret in GitLab (safe place)

Go to:

GitLab → Project → Settings → CI/CD → Variables

Add:
```
Key	Value	Masked	Protected
DB_PASSWORD	MyStrongPassword123	✅	✅
```
### STEP 2: GitLab CI creates/updates Kubernetes Secret
In .gitlab-ci.yml:
```
deploy:
  stage: deploy
  script:
    - kubectl create secret generic db-secret \
        --from-literal=password="$DB_PASSWORD" \
        --dry-run=client -o yaml | kubectl apply -f -
    - helm upgrade --install myapp ./helm/myapp
```
What happens here:

- GitLab injects $DB_PASSWORD

- Secret is created/updated in Kubernetes

Helm is deployed

- ✅ Fully automated
- ✅ Secure
- ✅ Reproducible

### STEP 3: Helm Deployment references Secret (NO secret in Helm)
```
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```
### STEP 4: Pod startup flow (MOST IMPORTANT)

- Pod starts

- Kubernetes sees valueFrom.secretKeyRef

- Kubernetes fetches secret from etcd

- Injects it into container env

- App reads it using:
```
import os
db_password = os.getenv("DB_PASSWORD")
```
#### ConfigMap = configuration
#### Secret = sensitive configuration

## service.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  selector:
    app: {{ include "myapp.name" . }}
  ports:
    - name: http          
      port: 80
      targetPort: 8080
```

#### Why service?

Pods are ephemeral(👉 Pods can be created, destroyed, and recreated at any time.  Pod name/IP changes)

Service provides stable network endpoint

##### Nice, this is a very important object.
- A Service is what makes your ephemeral Pods usable.

I’ll explain each line, then why it exists, then production-ready improvements

#### What this object is (big picture)

- Service = stable network endpoint for unstable Pods

- Pods come and go, IPs change →
#### Service gives:

- Stable DNS name

- Stable virtual IP

- Load balancing

```
apiVersion: v1
```
Why?

- Tells Kubernetes which API schema to use

- Service is a core object, so it lives in v1

- ✅ Mandatory
- ❌ Cannot change randomly

```
kind: Service
```
Why?

- Tells Kubernetes what object to create

- This creates a Service, not a Pod, not Deployment

```
metadata:
```
Why?

- Metadata = identity info

- Name, labels, annotations live here

```
 name: {{ include "myapp.fullname" . }}
```
Why?

- Helm template expression

- Produces a unique name like: `myrelease-myapp`

Why not hardcode?

- Avoid name collisions

- Support multiple installs:

- dev-myapp

- prod-myapp

```
spec:
```
Why?

- spec defines desired state

- What ports? Which Pods? How to expose?

```
  selector:
```
Why?

Selector tells Service:

- “Which Pods should I send traffic to?”

- Without selector → Service does nothing

```
    app: {{ include "myapp.name" . }}
```
Why?

- Matches Pods that have this label

- Example Pod label:
```
labels:
  app: myapp
```
👉 If labels don’t match → traffic goes nowhere ❌

```
  ports:
```
Why?

- Service can expose one or more ports

```
    - name: http
```
Why?

Named port (optional but best practice)

Used by:

- Ingress

- Istio

- Network policies

- Good habit 👍

  ```
        port: 80
  ```
Meaning:

- Port exposed by the Service

- Clients connect to this port

Example:
```
curl http://myapp
```
(default HTTP = 80)
```
      targetPort: 8080
```
Meaning:

- Port inside the container

- Your app listens on 8080

Traffic flow:
```
Client → Service:80 → Pod:8080
```
Visual flow (important)
```
Browser / Ingress
        |
        v
Service (port 80)
        |
        v
Pod (port 8080)
```
Why Service is REQUIRED

Without Service:

- You must talk directly to Pod IP ❌

- Pod IP changes ❌

- No load balancing ❌

- Service solves all three.

#### Improvements for production (VERY IMPORTANT)
1️⃣ Explicit Service type

By default:
```
type: ClusterIP
```
Best to be explicit:
```
spec:
  type: ClusterIP
```
Other types:

| Type         | Use case               |
| ------------ | ---------------------- |
| ClusterIP    | Internal communication |
| NodePort     | Debug / small setups   |
| LoadBalancer | Cloud external access  |
| ExternalName | DNS alias              |

#### 2️⃣ Support multiple ports (future-proof)

```
ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9090
    targetPort: 9090

```
Use labels instead of only app

Better selector:
```
selector:
  app.kubernetes.io/name: {{ include "myapp.name" . }}
  app.kubernetes.io/instance: {{ .Release.Name }}

```
This avoids cross-service collisions.

#### 4️⃣ Full production-ready Service (recommended)
```
apiVersion: v1
kind: Service
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "myapp.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: {{ include "myapp.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
```
Common mistakes ❌

- Selector labels don’t match Deployment labels

- Forgetting Service and trying to access Pod IP

         Hardcoding names

- Using NodePort in production unnecessarily

### Interview one-liner 🎯

        A Kubernetes Service provides a stable virtual IP and DNS name to expose ephemeral Pods and load-balances traffic across them

## Big picture first (very important)

    Service = exposes Pods inside the cluster
    Ingress = exposes Services to the outside world (HTTP/HTTPS)

They solve different problems.

1️⃣ Kubernetes Service (recap, but important)

What problem does Service solve?

Pods are:

- Ephemeral

- Have changing IPs

- Multiple replicas

Service provides:

- Stable IP

- Stable DNS

- Load balancing]

#### Service scope

👉 Inside the cluster

Example DNS:

    myapp.default.svc.cluster.local

Service types:

| Type         | Who can access                |
| ------------ | ----------------------------- |
| ClusterIP    | Only inside cluster           |
| NodePort     | External (via node IP + port) |
| LoadBalancer | External (cloud-managed LB)   |

### 2️⃣ Ingress
What problem does Ingress solve?

Without Ingress:
```
service-1 → NodePort 30001
service-2 → NodePort 30002
service-3 → NodePort 30003
```
Problems:

- Ugly ports

- No TLS management

- No routing rules

#### What is Ingress?

    Ingress is an HTTP/HTTPS router that sits at the edge of the cluster

It provides:

- Host-based routing

- Path-based routing

- TLS termination

- Single entry point

### IMPORTANT: Ingress ≠ Load balancer

- Ingress is just a rule definition.
Actual traffic handling is done by:

- NGINX Ingress Controller

- Traefik

- HAProxy

- OpenShift Router

## 3️⃣ How Service + Ingress work together
Traffic flow (VERY IMPORTANT)
```
Browser
   ↓
Ingress Controller (NGINX / OpenShift Router)
   ↓
Ingress rules
   ↓
Service (ClusterIP)
   ↓
Pods
```
Ingress never talks to Pods directly.

### 4️⃣ Concrete example (step by step)

#### Step 1: Deployment (Pods)

Your app listens on port 8080.
#### Step 2: Service (internal exposure)
```
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```
Now app is accessible inside cluster:
```
http://myapp
```
#### Step 3: Ingress (external exposure)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp
                port:
                  number: 80
```
Now accessible from internet:
```
https://myapp.example.com
```
#### 5️⃣ Service vs Ingress (clear comparison)
| Feature           | Service      | Ingress         |
| ----------------- | ------------ | --------------- |
| Purpose           | Expose Pods  | Expose Services |
| Works at          | L4 (TCP/UDP) | L7 (HTTP/HTTPS) |
| Stable IP         | ✅ Yes        | ❌ No            |
| Routing rules     | ❌ No         | ✅ Yes           |
| TLS               | ❌ No         | ✅ Yes           |
| Talks to Pods     | ✅ Yes        | ❌ No            |
| Talks to Services | ❌ No         | ✅ Yes           |

#### 6️⃣ Why both are needed (key insight)

Ingress depends on Service.

Ingress cannot:

- Select Pods

- Load balance Pods

- Handle Pod IP changes

- Service does all that.

#### 7️⃣ Production setup (REAL WORLD)
```
Internet
   ↓
Cloud LoadBalancer / OpenShift Router
   ↓
Ingress Controller
   ↓
Ingress (rules)
   ↓
Service (ClusterIP)
   ↓
Pods (Deployment)
```
#### 8️⃣ Why not expose Service directly?
NodePort ❌

-Security risk

-Random ports

-Hard to manage

LoadBalancer ❌ (for many apps)

- One LB per Service

- Expensive

- No smart routing

Ingress:

- One LB

-  apps

- Clean URLs

#### 9️⃣ OpenShift note (important for you)

OpenShift uses Route, not standard Ingress (but same idea).

```
kind: Route
```
Internally:

- Route → Service → Pod

Same concept, different object

#### 🔟 Interview-ready explanation 🎯

A Service provides stable internal access to Pods, while an Ingress provides external HTTP/HTTPS access by routing traffic to Services based on host and path rules.

#### Common mistakes ❌

- Creating Ingress without Service

- Using NodePort in production

- Expecting Ingress to talk directly to Pods

- Missing Ingress Controller

