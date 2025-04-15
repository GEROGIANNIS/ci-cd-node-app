# 🚀 Node.js CI/CD Pipeline with GitHub Actions, Docker & Kubernetes

This project demonstrates a complete DevOps pipeline using:
- GitHub Actions for CI/CD
- Docker for containerization
- Docker Hub for image hosting
- Kubernetes (Minikube) for deployment

---

## 📁 Project Structure

```
ci-cd-node-app/
              ├── app.js
              ├── package.json
              ├── Dockerfile
              ├── k8s/
              │   └── deployment.yaml
              └── .github/
                  └── workflows/
                      └── ci-cd-pipeline.yml
```

---

## 🛠️ Prerequisites

Make sure you have:
- Node.js + npm
- Docker installed and logged in to Docker Hub
- Minikube or a Kubernetes cluster
- A GitHub account and a public repository

---

## 🔧 Step-by-Step Setup

### 1. Clone This Repo

```bash
git clone https://github.com/GEROGIANNIS/ci-cd-node-app.git
cd ci-cd-node-app
```

---

### 2. Run the App Locally

```bash
npm install
npm start
```

Visit `http://localhost:3000`

---

### 3. Dockerize the App

#### Dockerfile

```Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

Build & run:

```bash
docker build -t GEROGIANNIS/ci-cd-node-app .
docker run -p 3000:3000 GEROGIANNIS/ci-cd-node-app
```

---

### 4. Create GitHub Actions CI/CD Workflow

Inside `.github/workflows/ci-cd-pipeline.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    env:
      IMAGE_NAME: GEROGIANNIS/ci-cd-node-app
      TAG: latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: docker build -t $IMAGE_NAME:$TAG .

      - name: Push Docker image
        run: docker push $IMAGE_NAME:$TAG

      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/deployment.yaml
```

---

### 5. Add Secrets to GitHub Repo

Go to your repo → **Settings** → **Secrets and Variables** → **Actions** → **New Repository Secret**

Add:
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN` (from Docker Hub > Security > New Access Token)

---

### 6. Kubernetes Deployment Config

Create `k8s/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
      - name: node-app
        image: GEROGIANNIS/ci-cd-node-app:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  type: NodePort
  selector:
    app: node-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30036
```

Apply it locally:
```bash
kubectl apply -f k8s/deployment.yaml
minikube service node-app-service
```

---

## ✅ Result

- Every push to `main` triggers:
  - Build
  - Test
  - Dockerize
  - Push to Docker Hub
  - Deploy to Kubernetes

---
