# Jenkins — Usage

## 🚀 Getting Started with Jenkins

This guide covers practical usage examples for Jenkins in real-world CI/CD projects.

## 📊 Example 1: Basic Pipeline

### Scenario: Simple Build Pipeline

Create a basic CI pipeline for a Java application.

### Step 1: Create Jenkinsfile
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
```

### Step 2: Create Pipeline Job
```bash
# Via Web UI:
# 1. New Item > Pipeline
# 2. Configure pipeline
# 3. Pipeline definition: Pipeline script from SCM
# 4. Repository URL: https://github.com/user/repo.git
# 5. Script path: Jenkinsfile
# 6. Save and Build
```

## 🔧 Example 2: Multi-stage Deployment

### Scenario: Deploy to Multiple Environments

Deploy application to staging and production.

### Step 1: Create Deployment Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                sh 'kubectl apply -f k8s/staging/'
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh 'mvn verify -Denv=staging'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f k8s/production/'
            }
        }
    }
}
```

## 🚀 Example 3: Docker Pipeline

### Scenario: Build and Push Docker Images

Build Docker images and push to registry.

### Step 1: Create Docker Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'my-app'
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}")
                    image.push()
                    image.push("latest")
                }
            }
        }
    }
}
```

## 🎯 Example 4: Parallel Execution

### Scenario: Run Tests in Parallel

Execute multiple test suites in parallel.

### Step 1: Create Parallel Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test -Dtest=UnitTest*'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn test -Dtest=IntegrationTest*'
                    }
                }
                stage('E2E Tests') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }
    }
}
```

## 🔄 Example 5: Conditional Deployment

### Scenario: Deploy Based on Conditions

Deploy only when certain conditions are met.

### Step 1: Create Conditional Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    expression { env.BUILD_NUMBER.toInteger() % 10 == 0 }
                }
            }
            steps {
                sh 'kubectl apply -f k8s/production/'
            }
        }
    }
}
```

## 📊 Monitoring and Debugging

### Check Pipeline Status
```bash
# View pipeline status
# Via Web UI: Pipeline > Status

# View console output
# Pipeline > Build > Console Output

# View build history
# Pipeline > Build History
```

## 🎯 Common Usage Patterns

### 1. **CI Pipeline**
```groovy
pipeline {
    stages {
        stage('Checkout') { /* ... */ }
        stage('Build') { /* ... */ }
        stage('Test') { /* ... */ }
    }
}
```

### 2. **CD Pipeline**
```groovy
pipeline {
    stages {
        stage('Build') { /* ... */ }
        stage('Deploy Staging') { /* ... */ }
        stage('Deploy Production') { /* ... */ }
    }
}
```

---

*Jenkins is now integrated into your CI/CD workflow! 🎉*