# 🌸 Iris Classifier with GKE, FastAPI, Logging & Telemetry

This project demonstrates a full-stack MLOps deployment of an Iris classifier using **FastAPI**, **Docker**, and **Google Kubernetes Engine (GKE)** with built-in **structured logging**, **Google Cloud Trace**, **health probes**, and **autoscaling**.

---

## 🚀 Features

- 🔍 Real-time Iris flower classification (`/predict` endpoint)
- 📈 Autoscaling using Kubernetes HPA (based on CPU)
- 🔧 Structured logs to Google Cloud Logging
- 📊 Tracing with OpenTelemetry → Cloud Trace
- ✅ Liveness & readiness probes
- 🐋 Dockerized & deployed on GKE
- ⚙️ Workload Identity-based GCP access

---

## 🗂️ Project Structure

```
.
├── demo_log.py              # FastAPI app with ML logic, tracing & logging
├── iris-model.joblib        # Trained iris model
├── requirements.txt         # Python dependencies
├── Dockerfile               # Container setup
├── deployment.yaml          # Kubernetes Deployment (includes probes + SA)
├── service.yaml             # Kubernetes Service (LoadBalancer)
├── hpa.yaml                 # Horizontal Pod Autoscaler config
├── post.lua                 # wrk load test script
```

---

## ⚙️ Prerequisites

- ✅ GCP Project with Billing Enabled
- ✅ Artifact Registry created (e.g., `my-repo`)
- ✅ `gcloud` CLI installed and authenticated
- ✅ Kubernetes cluster created with Workload Identity

---

## 📦 Build and Push Docker Image

```bash
docker build -t demo_log .
docker tag demo_log us-central1-docker.pkg.dev/YOUR_PROJECT_ID/my-repo/demo_log:latest
docker push us-central1-docker.pkg.dev/YOUR_PROJECT_ID/my-repo/demo_log:latest
```

---

## ☸️ Deploy on GKE

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

Then get external IP:

```bash
kubectl get service demo-log-ml-service
```

Test it:

```bash
curl -X POST http://<EXTERNAL-IP>:80/predict \
  -H "Content-Type: application/json" \
  -d '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}'
```

---

## 📊 Load Testing

Install `wrk`:
```bash
sudo apt install wrk
```

Create `post.lua`:

```lua
wrk.method = "POST"
wrk.body   = '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}'
wrk.headers["Content-Type"] = "application/json"
```

Run load test:

```bash
wrk -t4 -c100 -d30s --latency -s post.lua http://<EXTERNAL-IP>:80/predict
```

Monitor autoscaler:

```bash
kubectl get hpa
kubectl get pods -w
```

---

## 🔍 Observe Logs & Traces

### ✅ Logs
[Cloud Logging Dashboard](https://console.cloud.google.com/logs/query)

- Check for structured logs from `demo-log-ml-service`
- Filter by `resource.type="k8s_container"` and severity `INFO`

### ✅ Traces
[Cloud Trace Dashboard](https://console.cloud.google.com/traces/list)

- View latency and trace IDs for `/predict` endpoint

---

## ✅ Health Checks

- `/live_check` → for liveness probe
- `/ready_check` → for readiness (after model load delay)

---

## 🧠 Model Input Format

```json
{
  "sepal_length": 5.1,
  "sepal_width": 3.5,
  "petal_length": 1.4,
  "petal_width": 0.2
}
```

---

## 🙌 Author

**Satyam Srivastava** — _Deployed via GKE with ❤️_

---

## 📌 License

This project is for educational/demo purposes only.
