# 🧩 Microservices Deployment with Helm & Helmfile on DigitalOcean Kubernetes

## 📖 Overview

This project showcases a complete **microservices deployment** using **Helm** and **Helmfile** on a **DigitalOcean Kubernetes cluster**.  
It includes:
- A dedicated **Redis Helm chart** for caching and message queuing.
- A reusable **Microservice Helm chart** shared across nine services.
- Centralized **Helmfile orchestration** to deploy and manage all services consistently.

---

## ⚙️ Project Structure

```
.
├── charts/
│   ├── redis/                     # Redis Helm chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │
│   └── microservice/              # Generic Helm chart for all services
│       ├── Chart.yaml
│       ├── templates/
│       │   ├── deployment.yaml
│       │   └── service.yaml
│
├── values/                        # Values for each microservice
│   ├── redis-values.yaml
│   ├── email-service-values.yaml
│   ├── cart-service-values.yaml
│   ├── currency-service-values.yaml
│   ├── payment-service-values.yaml
│   ├── recommendation-service-values.yaml
│   ├── productcatalog-service-values.yaml
│   ├── shipping-service-values.yaml
│   ├── ad-service-values.yaml
│   ├── checkout-service-values.yaml
│   └── frontend-values.yaml
│
└── helmfile.yaml                  # Helmfile for orchestrating all charts
```

---

## 🚀 Deployment Overview

### 1️⃣ Helmfile

The `helmfile.yaml` defines all service releases (Redis + 9 microservices):

```yaml
releases:
  - name: rediscart
    chart: charts/redis
    values:
      - values/redis-values.yaml
      - appReplicas: "1"
      - volumeName: "redis-cart-data"
  
  - name: emailservice
    chart: charts/microservice
    values:
      - values/email-service-values.yaml

  - name: cartservice
    chart: charts/microservice
    values:
      - values/cart-service-values.yaml

  - name: currencyservice
    chart: charts/microservice
    values:
      - values/currency-service-values.yaml

  - name: paymentservice
    chart: charts/microservice
    values:
      - values/payment-service-values.yaml

  - name: recommendationservice
    chart: charts/microservice
    values:
      - values/recommendation-service-values.yaml

  - name: productcatalogservice
    chart: charts/microservice
    values:
      - values/productcatalog-service-values.yaml

  - name: shippingservice
    chart: charts/microservice
    values:
      - values/shipping-service-values.yaml

  - name: adservice
    chart: charts/microservice
    values:
      - values/ad-service-values.yaml

  - name: checkoutservice
    chart: charts/microservice
    values:
      - values/checkout-service-values.yaml

  - name: frontendservice
    chart: charts/microservice
    values:
      - values/frontend-values.yaml
```

Helmfile ensures that all Helm releases are deployed consistently and in the correct order.

---

## 🧱 Microservice Helm Chart

Each microservice shares a common Helm chart that defines both the Deployment and Service templates.

### `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
        ports:
        - containerPort: {{ .Values.containerPort }}
        env:
        {{- range .Values.containerEnvVars }}
          - name: {{ .name }}
            value: {{ .value | quote }}
        {{- end }}
```

### `templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  type: {{ .Values.serviceType }}
  selector:
    app: {{ .Values.appName }}
  ports:
    - protocol: TCP
      port: {{ .Values.servicePort }}
      targetPort: {{ .Values.containerPort }}
```

---

## ⚙️ Example Service Configuration

Example file: `values/email-service-values.yaml`

```yaml
appName: emailservice
appImage: gcr.io/google-samples/microservices-demo/emailservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 8080
containerEnvVars:
  - name: PORT
    value: "8080"

servicePort: 5000
```

---

## ☸️ Deployment Steps

### Step 1 — Connect to Your DigitalOcean Cluster

Download the kubeconfig from DigitalOcean and export it:

```bash
export KUBECONFIG=~/.kube/do-cluster.yaml
kubectl get nodes
```

---

### Step 2 — Deploy All Charts via Helmfile

Run:

```bash
helmfile apply
```

This will:
- Deploy **Redis**
- Deploy all **9 microservices**
- Apply all environment configurations automatically

---

### Step 3 — Verify Deployment

```bash
kubectl get pods
kubectl get svc
kubectl get deployments
```

Check logs for a specific service:

```bash
kubectl logs -l app=emailservice
```

---

## 🔁 Managing Releases

### Upgrade all releases
```bash
helmfile sync
```

### Roll back a specific service
```bash
helm rollback emailservice 1
```

### Delete all releases
```bash
helmfile destroy
```

---

## 🧠 Key Concepts

| Concept | Description |
|----------|-------------|
| **Helm** | Package manager for Kubernetes |
| **Helm Chart** | A reusable Kubernetes template with parameterized values |
| **Helmfile** | Tool to manage multiple Helm releases declaratively |
| **Redis Chart** | Caching backend used by multiple microservices |
| **Microservice Chart** | Shared Helm chart for all application services |
| **DigitalOcean Cluster** | Managed Kubernetes environment for deployment |

---

## ✅ Summary

- Modular architecture: **Redis + 9 Microservices**
- Managed with **Helm** and **Helmfile**
- Deployed on **DigitalOcean Kubernetes**
- Fully customizable through `values/*.yaml`
- Scalable, reusable, and CI/CD friendly

---

