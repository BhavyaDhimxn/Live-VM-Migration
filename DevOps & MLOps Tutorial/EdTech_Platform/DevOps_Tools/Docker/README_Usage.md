# Docker — Usage

## 🚀 Getting Started with Docker

This guide covers practical usage examples for Docker in real-world projects.

## 📊 Example 1: Build and Run Container

### Scenario: Containerize a Python Application

Create a Docker image for a Python web application.

### Step 1: Create Dockerfile
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
```

### Step 2: Build Image
```bash
# Build Docker image
docker build -t my-python-app:latest .

# Tag image
docker tag my-python-app:latest myregistry/my-python-app:v1.0.0
```

### Step 3: Run Container
```bash
# Run container
docker run -d -p 8080:8080 --name my-app my-python-app:latest

# View logs
docker logs my-app

# Stop container
docker stop my-app
```

## 🔧 Example 2: Multi-container Application

### Scenario: Web App with Database

Use Docker Compose for multi-container setup.

### Step 1: Create docker-compose.yml
```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

### Step 2: Run Application
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

## 🚀 Example 3: Docker in CI/CD

### Scenario: Build and Push in Pipeline

Use Docker in Jenkins/GitLab CI.

### Step 1: Jenkins Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t my-app:${BUILD_NUMBER} .'
            }
        }
        stage('Push') {
            steps {
                sh 'docker push my-registry/my-app:${BUILD_NUMBER}'
            }
        }
    }
}
```

## 🎯 Common Usage Patterns

### 1. **Development Workflow**
```bash
# Build image
docker build -t my-app .

# Run container
docker run -d -p 8080:8080 my-app

# View logs
docker logs -f <container-id>
```

### 2. **Production Deployment**
```bash
# Build production image
docker build -t my-app:prod .

# Push to registry
docker push my-registry/my-app:prod

# Deploy
docker run -d -p 8080:8080 my-registry/my-app:prod
```

---

*Docker is now integrated into your workflow! 🎉*