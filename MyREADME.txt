# Rinat Reviews App - Kubernetes Deployment

This repository contains Kubernetes manifests to deploy a full-stack reviews application composed of three components: a **Frontend**, a **Backend API**, and a **MySQL Database**. Docker images are stored in AWS ECR and deployments are managed via `kubectl`.

---

## ✨ Architecture Overview

```
+----------------------+          +--------------------+          +----------------------+
|                      |  HTTP    |                    |  TCP     |                      |
|    LoadBalancer      +--------->   Frontend Pod      +--------->     Backend Pod       |
|  frontend-service    |          | (React or HTML/JS) |          | (Node.js/Python API) |
+----------------------+          +--------------------+          +----------------------+
                                                                       |
                                                                       |  TCP
                                                                       v
                                                             +----------------------+
                                                             |   MySQL StatefulSet   |
                                                             |   database-service    |
                                                             +----------------------+
```

---

## 📁 Project Structure

```
.
├── backend/
│   └── k8s/
│       ├── backend-deployment.yaml
│       └── backend-service.yaml
├── frontend/
│   ├── Dockerfile
│   └── k8s/
│       ├── frontend-deploymnet.yaml
│       └── frontend-service.yaml
├── database/
│   ├── Dockerfile
│   └── k8s/
│       ├── database-deployment.yaml (now StatefulSet)
│       ├── database-service.yaml
│       ├── storage-class.yaml
│       └── database-secret.yaml
```

---

## 🌐 Components

### 1. **Frontend**

* Docker image: `frontend:1.1`
* Deployed as a Kubernetes **Deployment**
* Exposed via **LoadBalancer Service** (`frontend-service`)

### 2. **Backend**

* Docker image: `backend:1.0`
* Uses environment variables for DB connection
* Password pulled from Kubernetes **Secret** (`backend-secret`)
* Exposed via **LoadBalancer Service** (`backend-service`)

### 3. **Database (MySQL)**

* Docker image: `database:1.1`
* Deployed using a **StatefulSet** for persistent storage
* Mounted volume via a **PersistentVolumeClaim**
* Credentials and DB info passed via **Secret** (`database-secret`)
* Storage provisioned via custom **StorageClass** `mysql-gp2`
* Exposed via **ClusterIP Service** (`database-service`)

---

## 📅 Deployment Steps (Used Commands)

```bash
# Clone and switch to personal GitHub repo
git clone https://github.com/312school/rinat-reviews-app-base.git
cd rinat-reviews-app-base

git remote remove origin
git remote add origin https://github.com/rinatmyrzaliev/rinat-reviews-app-base-k8s

git checkout -b feature/myrinat-reviews-app-base

# Build Docker images
cd frontend
docker build . -t frontend:1.1 --platform=linux/amd64
docker tag frontend:1.1 <ECR_URL>/frontend:1.1
docker push <ECR_URL>/frontend:1.1

cd ../api-backend
docker build . -t backend:1.0 --platform=linux/amd64
docker push <ECR_URL>/backend:1.0

cd ../database
docker build . -t database:1.1 --platform=linux/amd64
docker push <ECR_URL>/database:1.1

# Create Kubernetes resources
kubectl apply -f frontend/k8s/frontend-deploymnet.yaml
kubectl apply -f frontend/k8s/frontend-service.yaml

kubectl apply -f backend/k8s/backend-deployment.yaml
kubectl apply -f backend/k8s/backend-service.yaml

kubectl apply -f database/k8s/storage-class.yaml
kubectl apply -f database/k8s/database-secret.yaml
kubectl apply -f database/k8s/database-deployment.yaml
kubectl apply -f database/k8s/database-service.yaml

# Validate
kubectl get pods
kubectl get svc
kubectl get ep
kubectl logs <pod-name>
```

---

## 🔑 Secrets (Base64 Encoded)

### `backend-secret`

```yaml
data:
  DB_PASSWORD: YWRtaW4xMjM0   # admin1234
```

### `database-secret`

```yaml
data:
  MYSQL_ROOT_PASSWORD: cm9vdDEyMzQ=     # root1234
  MYSQL_PASSWORD: YWRtaW4xMjM0          # admin1234
  MYSQL_USER: YWRtaW4=                  # admin
  MYSQL_DATABASE: cmV2aWV3cy1kYg==      # reviews-db
```

---

## 🌧️ Services Summary

| Service Name     | Type         | Ports | Targets           |
| ---------------- | ------------ | ----- | ----------------- |
| frontend-service | LoadBalancer | 80    | frontend pod      |
| backend-service  | LoadBalancer | 80    | backend pod       |
| database-service | ClusterIP    | 3306  | MySQL StatefulSet |

---

## ✅ Final Notes

* All images are built and hosted on **AWS ECR**.
* Sensitive values are stored securely using **Kubernetes Secrets**.
* Database is deployed as a **StatefulSet** with persistent volumes.
* Services are accessible via **AWS LoadBalancer endpoints**.




🧠 Project Goal Summary
You worked on a full-stack microservice-style project called rinat-reviews-app-base, where you:

✅ Cloned and reconnected a GitHub repo

✅ Built and pushed Docker images for frontend, backend, and database apps to AWS ECR

✅ Deployed them to a Kubernetes cluster (likely via kubectl)

✅ Created supporting Kubernetes services (like LoadBalancer, ClusterIP)

✅ Validated pod connectivity and service exposure via DNS and curl

✅ Committed and pushed changes using Git to your own feature branch

⚙️ Key Components You Built
🖥️ 1. Frontend
Built with Docker: frontend:1.0, then updated to frontend:1.1

Pushed to ECR:
517121893325.dkr.ecr.us-east-1.amazonaws.com/frontend:1.1

Deployed to Kubernetes via frontend-deploymnet.yaml

Exposed with a LoadBalancer service (frontend-service.yaml)

Tested via DNS with curl, nslookup, dig

⚙️ 2. Backend API
Dockerized as backend:1.0

Pushed to ECR

Created backend-deployment.yaml

Exposed with a LoadBalancer service backend-service.yaml

Validated via kubectl exec, pod logs, and service status

🗄️ 3. Database (MySQL)
Built and pushed your own image database:1.1

Created database-deployment.yaml using the MySQL image and later your custom image

Added secrets like MYSQL_ROOT_PASSWORD, MYSQL_USER, and MYSQL_PASSWORD

Fixed a crash caused by missing MYSQL_DATABASE env or bad init SQL (No database selected)

Created a ClusterIP service database-service.yaml

Ran kubectl exec into the pod to validate

🛠️ Infrastructure / CI Work
Switched Git remote to your own repo

Created feature branch: feature/myrinat-reviews-app-base

Committed changes in structured steps: Dockerfiles, K8s YAMLs, service manifests

Used aws sts get-caller-identity to verify AWS credentials

Deployed to Kubernetes using a kops-based cluster (or similar)

Verified nodes, pods, endpoints, and logs repeatedly

🧹 Cleanup and Troubleshooting
Uninstalled Docker manually at one point

Linked missing Docker CLI binaries (docker, docker-credential-desktop)

Troubleshot failed deployments (CrashLoopBackOff, missing DB)

Fixed YAML mistakes (like typos in deployment file names)

Used kubectl get, describe, logs, and exec to investigate

✅ In Short: You...
Cloned, rebuilt, and personalized a full-stack app

Dockerized all three components (frontend, backend, MySQL)

Deployed them to EKS (or kops-based cluster) using Kubernetes manifests

Exposed the services and verified connectivity

Pushed everything to your GitHub repo feature branch