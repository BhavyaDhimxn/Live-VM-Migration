# Jenkins — Best Practices

## 🎯 Production Best Practices

### 1. **Pipeline as Code**
```groovy
// Store Jenkinsfile in repository
// Version control pipelines
// Use declarative pipelines
pipeline {
    agent any
    stages {
        // Pipeline stages
    }
}
```

### 2. **Security**
```groovy
// Use credentials securely
pipeline {
    environment {
        DOCKER_PASSWORD = credentials('docker-password')
    }
    stages {
        // Use credentials
    }
}
```

## 🔐 Security Best Practices

### 1. **Credential Management**
```groovy
// Never hardcode credentials
// Bad:
sh 'docker login -u admin -p hardcoded-password'

// Good:
withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
    sh 'docker login -u $USER -p $PASS'
}
```

### 2. **Pipeline Security**
```groovy
// Use script approval
// Manage Jenkins > In-process Script Approval
// Approve only necessary scripts
```

## 📊 Pipeline Best Practices

### 1. **Idempotent Pipelines**
```groovy
// Make pipelines idempotent
pipeline {
    stages {
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/'  // Idempotent
            }
        }
    }
}
```

### 2. **Error Handling**
```groovy
// Implement proper error handling
pipeline {
    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        sh 'mvn package'
                    } catch (Exception e) {
                        echo "Build failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
    }
}
```

## 🔄 CI/CD Integration

### 1. **Git Integration**
```groovy
// Integrate with Git
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
    }
}
```

## 📈 Monitoring Best Practices

### 1. **Build Notifications**
```groovy
// Notify on build status
pipeline {
    post {
        success {
            emailext (
                subject: "Build Success: ${env.JOB_NAME}",
                body: "Build ${env.BUILD_NUMBER} succeeded",
                to: "team@example.com"
            )
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME}",
                body: "Build ${env.BUILD_NUMBER} failed",
                to: "team@example.com"
            )
        }
    }
}
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Hardcode Credentials**
```groovy
// Bad: Hardcoded credentials
sh 'docker login -u admin -p password123'

// Good: Use credentials
withCredentials([string(credentialsId: 'docker-password', variable: 'PASS')]) {
    sh 'docker login -u admin -p $PASS'
}
```

---

*Follow these best practices to build robust CI/CD pipelines with Jenkins! 🎯*