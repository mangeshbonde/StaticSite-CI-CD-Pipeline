# 🚀 StaticSite CI/CD Pipeline (Docker → Jenkins → Kubernetes)

## 📌 Project Overview

This project demonstrates a complete **CI/CD pipeline** for deploying a static website using:

* Docker (Containerization)
* Jenkins (CI Automation)
* Kubernetes (Deployment)
* AWS EC2 (Infrastructure)

The pipeline automates building, packaging, and deploying a static HTML application.

---

## 🎯 Objective

To build an automated pipeline where:

```
Code → Docker Image → Docker Hub → Kubernetes Deployment
```

---

## 🏗️ Project Structure

```
ss-cicd-devops-pipeline/
│
├── app/
│   └── index.html
│
├── Dockerfile
│
├── jenkins/
│   └── Jenkinsfile
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
├── README.md
```

---

# ⚙️ Step 1: Docker Setup

## 🐳 Dockerfile

```dockerfile
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY app/index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 🔨 Build Image

```bash
docker build -t <dockerhub-username>/nginx-co:v1 .
```

---

## ▶️ Run Container

```bash
docker run -d -p 8080:80 <dockerhub-username>/nginx-co:v1
```

---

## 🌐 Test

```
http://localhost:8080
```

---

# 📤 Step 2: Push Image to Docker Hub

## 🔐 Login

```bash
docker login
```

---

## 📦 Push Image

```bash
docker push <dockerhub-username>/nginx-co:v1
```

---

# ⚙️ Step 3: Jenkins CI Pipeline

## 🔧 Install Plugins

* Pipeline
* Git
* Docker Pipeline

---

## 🔐 Add Credentials

Jenkins → Manage Jenkins → Credentials

* Type: Username & Password
* ID: `dockerhub-creds`

---

## 📄 Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "<dockerhub-username>/nginx-co"
        TAG = "v2"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$TAG .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:$TAG'
            }
        }
    }
}
```

---

## ⚠️ Fixes Applied

* Removed duplicate Git clone stage
* Fixed branch issue (`main` instead of `master`)
* Fixed Docker permissions:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

# ☸️ Step 4: Kubernetes Deployment (EC2 + Kind)

## 🧱 Create Cluster

```bash
kind create cluster --name devops-cluster
```

---

## 📄 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-container
        image: <dockerhub-username>/nginx-co:v2
        ports:
        - containerPort: 80
```

---

## 📄 service.yaml (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  type: ClusterIP
  selector:
    app: nginx-app
  ports:
    - port: 80
      targetPort: 80
```

---

## 🚀 Apply

```bash
kubectl create namespace nginx
kubectl apply -f kubernetes/
```

---

# 🌐 Step 5: Access Application

## ⚠️ Important

ClusterIP is internal → cannot access via browser directly.

---

## ✅ Use Port Forward

```bash
kubectl port-forward -n nginx service/nginx-service 8081:80 --address 0.0.0.0
```

---

## 🌍 Access

```
http://<EC2-PUBLIC-IP>:8081
```

---

## 🔓 Security Group Rule

Allow:

```
Port: 8081
Source: 0.0.0.0/0
```

---

# 🌍 Step 6: Ingress Setup (Production Style)

## Install Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

---

## 📄 ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: nginx
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

---

## Apply

```bash
kubectl apply -f k8s/ingress.yaml
```

---

## 🌐 Access

```
http://<EC2-PUBLIC-IP>
```

---

## 🔓 Open Port

```
Port: 80
```

---

# 🎉 Final Architecture

```
User → EC2 → Ingress → Service (ClusterIP) → Pods
```

---

# 🧠 Key Learnings

* Docker build context handling
* Jenkins credential management
* CI/CD pipeline creation
* Kubernetes service types
* Ingress-based routing
* AWS networking basics

---

# 🚀 Future Enhancements

* Jenkins → Kubernetes auto deployment
* Helm charts
* HTTPS with TLS
* Monitoring (Prometheus + Grafana)
* Terraform for infra

---

