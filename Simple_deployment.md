code: https://gitlab.com/obulareddy-group/myapp

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


## 1️⃣ What is TLS (in simple words)

TLS = Transport Layer Security

    TLS encrypts data between client (browser) and server, so no one can read or modify it in between.
Without TLS:

    Browser ---- plain text ---- Server
With TLS:

    Browser ---- 🔐 encrypted ---- Server

#### 2️⃣ Why TLS is needed

TLS gives you 3 guarantees:

1. Encryption 🔐

- Passwords, tokens, cookies are hidden

- Prevents man-in-the-middle attacks

2. Identity verification 🆔

- Browser verifies: “Am I really talking to myapp.example.com?”

3. Data integrity ✅

- Data cannot be modified in transit

That’s why browsers show:

- 🔒 HTTPS → safe

- ⚠️ HTTP → “Not Secure”

#### 3️⃣ TLS in Kubernetes context

TLS is NOT handled by your Pod in most cases.

Instead:
```
Browser
  ↓ TLS
Ingress Controller (TLS termination)
  ↓ HTTP
Service
  ↓
Pod
```

👉 This is called TLS termination at Ingress

#### 4️⃣ What TLS needs (very important)

To enable TLS you need:

Domain name

    myapp.example.com
TLS certificate

    Public certificate

    Private key

In Kubernetes, these are stored as a Secret.

#### 5️⃣ TLS Secret (core object)
TLS Secret format
```
apiVersion: v1
kind: Secret
metadata:
  name: myapp-tls
type: kubernetes.io/tls
data:
  tls.crt: <base64-cert>
  tls.key: <base64-key>
```

But you usually don’t write this by hand.

#### 6️⃣ Creating TLS Secret (realistic way)
Option 1: From certificate files
```
kubectl create secret tls myapp-tls \
  --cert=fullchain.pem \
  --key=privkey.pem
```

This stores cert safely in Kubernetes.

#### 7️⃣ Ingress with TLS (step by step)
Basic Ingress (no TLS)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
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
This gives:

    http://myapp.example.com ❌

##### Ingress with TLS ✅
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
spec:
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
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

Now you get:
    
    https://myapp.example.com 🔒

### 8️⃣ What actually happens at runtime (VERY IMPORTANT)

Browser connects to https://myapp.example.com

Ingress controller:

- Reads TLS cert from myapp-tls

- Completes TLS handshake

- Traffic is decrypted at Ingress

- Forwarded to Service as HTTP

- Service routes to Pods

- Pods never see TLS

#### 9️⃣ Where TLS certificates come from
Option A: Self-signed cert (dev only)
```
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt
```

⚠️ Browser warning

Option B: Let’s Encrypt (production ⭐)

- This is the industry standard.

You use:

- cert-manager

- Automatic renewals

- Trusted by browsers

#### 🔟 TLS with cert-manager (production setup)

##### Step 1: Install cert-manager

(done once per cluster)

##### Step 2: Create Issuer
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: you@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
```

##### Step 3: Annotate Ingress
```
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
```

Ingress automatically gets:

- TLS cert

- Secret created

- Auto-renewed

You never touch certificates again 🎉


##### 1️⃣ TLS in OpenShift (important for you)

OpenShift uses Route, not Ingress.

Example:
```
kind: Route
spec:
  tls:
    termination: edge
```

OpenShift:

- Can auto-generate certs

- Can integrate with external certs

- Same TLS concept, different object

##### 2️⃣ Common TLS patterns

| Pattern                    | Meaning          |
| -------------------------- | ---------------- |
| TLS termination at Ingress | Most common      |
| TLS passthrough            | App handles TLS  |
| Re-encryption              | TLS again to Pod |

##### 3️⃣ Common mistakes ❌

- Forgetting TLS secret

- Host mismatch in cert

- Expecting Pod to handle HTTPS

- Using self-signed cert in prod

##### 4️⃣ One-line interview answer 🎯

    TLS encrypts traffic between client and server, and in Kubernetes it is commonly terminated at the Ingress using a TLS certificate stored as a Secret.

Mental model (remember this)
```
TLS = lock 🔒
Certificate = proof of identity 🆔
Ingress = security gate 🚪
Secret = secure storage 🔐
```

### route.yaml (OpenShift specific)

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  to:
    kind: Service
    name: {{ include "myapp.fullname" . }}
  port:
    targetPort: http        # 👈 must match service port name
  tls:
    termination: edge       # optional but recommended
```
Why Route?

- OpenShift equivalent of Ingress

- Exposes app to outside world
```
tls:
  termination: edge
```

##### 1️⃣ What is an OpenShift Route?

In plain Kubernetes, we expose apps using:

- NodePort

- LoadBalancer

- Ingress

In OpenShift, the preferred way is:
- 👉 Route

A Route:

- Exposes your application to the outside world

- Automatically creates a public URL

- Handles DNS + TLS + routing via OpenShift’s router (HAProxy)

Think of it as:

- Ingress + LoadBalancer + TLS manager (OpenShift-style)
- HTTPS handled by OpenShift router

##### 2️⃣ Your YAML – Line by Line Explanation
🔹 apiVersion & kind
```
apiVersion: route.openshift.io/v1
kind: Route
```

Tells Kubernetes:

- “This is an OpenShift object”

- Not available in vanilla Kubernetes

- Managed by OpenShift Router

🔹 metadata
```
metadata:
  name: {{ include "myapp.fullname" . }}

```
Name of the Route resource

Helm template:

- myapp.fullname → something like
- myapp-dev, myapp-test, myapp-prod

Good practice ✅ (avoids naming collisions)

🔹 spec → to (Where traffic goes)
```
spec:
  to:
    kind: Service
    name: {{ include "myapp.fullname" . }}

```
This is CRITICAL.

It means:

- Incoming traffic →

- Forward to Kubernetes Service

- Not directly to Pods

📌 Flow:
```
Browser → Route → Service → Pod
```

Why Service?

- Load balancing

- Pod replacement

- Scaling

🔹 spec → port
```
  port:
    targetPort: http

```
This is where many people get confused.

What does this mean?

- targetPort must match the Service port NAME

- Not the number ❌

- The name exactly

Example Service:
```
ports:
  - name: http
    port: 80
    targetPort: 8080
```

✅ Route matches name: http

If names don’t match → Route will NOT work

###### 📌 Why OpenShift does this?

- Supports multiple ports on same Service

- Routes can select which port to expose

🔹 spec → tls
 ```
 tls:
    termination: edge
```

This enables HTTPS

Let’s explain this clearly.

##### 3️⃣ How Traffic Flows (End to End)

With this Route:
```
User Browser (HTTPS)
        ↓
OpenShift Router (HAProxy)
        ↓ (HTTP)
Service
        ↓
Pod (HTTP)

```

- TLS ends at OpenShift Router

- Pod gets plain HTTP

- No TLS inside cluster (simpler, faster)

##### 4️⃣ TLS termination: edge (Deep Explanation)
🔸 What is “edge”?

edge means:

- TLS is terminated at the router

- Router handles certificates

- Backend stays HTTP

- OpenShift automatically:

- Generates a certificate (or uses custom one)

- Redirects HTTPS traffic correctly

🔸 Other TLS types (for comparison)

| Type          | TLS Ends At  | When to Use          |
| ------------- | ------------ | -------------------- |
| `edge`        | Router       | Most common, easiest |
| `passthrough` | Pod          | App handles TLS      |
| `reencrypt`   | Router + Pod | High security        |

For most microservices:
- 👉 edge is PERFECT

##### 5️⃣ Why This Route Is “Correct & Recommended”

Your config is clean because:

- ✅ Uses Service (not Pod)
- ✅ Uses named port (http)
- ✅ Uses TLS (edge)
- ✅ Helm-friendly naming

This is production-grade OpenShift style

##### 6️⃣ Very Common Mistakes (Watch Out)
❌ Port name mismatch

```
Route targetPort: http
Service port name: web   ❌
```
→ Route won’t work

❌ Forgetting Service

    Route cannot point to Pod directly ❌

❌ Using NodePort instead of Route in OpenShift

    Works, but not recommended

##### 7️⃣ Real Example with Everything Connected
Service
```
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  ports:
    - name: http
      port: 80
      targetPort: 8080
```
Route
```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp
spec:
  to:
    kind: Service
    name: myapp
  port:
    targetPort: http
  tls:
    termination: edge
```
###### 8️⃣ One-Line Summary (Interview-Ready)

     An OpenShift Route exposes a Service externally, provides DNS and TLS termination via the OpenShift router, and routes traffic to the service port using named ports.

### 6️⃣ CI/CD – .gitlab-ci.yml

This is where everything connects.

```
stages:
  - build
  - deploy

variables:
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$IMAGE_TAG app/
    - docker push $CI_REGISTRY_IMAGE:$IMAGE_TAG

.deploy_template:
  stage: deploy
  image:
    name: bitnami/kubectl:latest
    entrypoint: [""]
  before_script:
    - apt-get update && apt-get install -y curl bash

    # Install oc to /tmp
    - curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz
    - tar -xzf oc.tar.gz
    - chmod +x oc kubectl
    - mv oc kubectl /tmp/

    # Install helm to /tmp
    - curl -fsSL https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz | tar -xz
    - mv linux-amd64/helm /tmp/

    # Add /tmp to PATH
    - export PATH="/tmp:$PATH"

    # Verify tools
    - oc version --client
    - helm version
    # Login to OpenShift
    - oc login "$OPENSHIFT_SERVER" --token="$OPENSHIFT_TOKEN" --insecure-skip-tls-verify

  script:
    - |
      helm upgrade --install myapp-${ENV} helm/myapp \
        -f helm/myapp/values-${ENV}.yaml \
        --set image.repository=${CI_REGISTRY_IMAGE} \
        --set image.tag=${IMAGE_TAG} \
        -n ${NAMESPACE}

deploy-dev:
  extends: .deploy_template
  variables:
    ENV: dev
    NAMESPACE: ravi-dev
  only:
    - develop

deploy-test:
  extends: .deploy_template
  variables:
    ENV: test
    NAMESPACE: ravi-dev
  only:
    - main

deploy-prod:
  extends: .deploy_template
  variables:
    ENV: prod
    NAMESPACE: ravi-dev
  only:
    - tags
  when: manual
```
This is where everything connects.

Stages
```
stages:
  - build
  - deploy
```
Build stage
```
docker build -t $CI_REGISTRY_IMAGE:$IMAGE_TAG app/
docker push $CI_REGISTRY_IMAGE:$IMAGE_TAG
```
What happens

- Build Docker image

- Push to GitLab Container Registry

This gives:

- Immutable images

- Easy rollback

- Tag = commit SHA

Deploy template
```
helm upgrade --install myapp-${ENV} helm/myapp \
  -f helm/myapp/values-${ENV}.yaml \
  --set image.repository=${CI_REGISTRY_IMAGE} \
  --set image.tag=${IMAGE_TAG}
```

What happens

- Helm renders templates

- Injects:
    
    - environment config
    
    - image version

- Deploys to OpenShift

Environment-specific deploy jobs
```
deploy-dev → develop branch
deploy-test → main branch
deploy-prod → tags (manual)
```
Why this mapping?

- Safe promotion strategy

- Prod requires manual approval

✅ Production-grade pipeline

