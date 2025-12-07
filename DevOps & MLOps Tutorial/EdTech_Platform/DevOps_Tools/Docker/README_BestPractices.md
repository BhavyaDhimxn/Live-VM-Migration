# Docker — Best Practices

## 🎯 Production Best Practices

### 1. **Dockerfile Optimization**
```dockerfile
# Use multi-stage builds
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/index.js"]
```

### 2. **Security**
```dockerfile
# Use non-root user
FROM node:16-alpine
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs
```

## 🔐 Security Best Practices

### 1. **Image Security**
```bash
# Scan images for vulnerabilities
docker scan my-image:latest

# Use official base images
FROM python:3.9-slim  # Official image

# Keep images updated
docker pull python:3.9-slim  # Get latest
```

### 2. **Secrets Management**
```bash
# Don't hardcode secrets in Dockerfile
# Use environment variables or secrets
docker run -e PASSWORD=$PASSWORD my-app
```

## 📊 Image Best Practices

### 1. **Layer Optimization**
```dockerfile
# Order commands by change frequency
# Install dependencies first (changes less)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy code last (changes more)
COPY . .
```

### 2. **Use .dockerignore**
```bash
# .dockerignore
node_modules
.git
.env
*.log
.DS_Store
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use Latest Tag**
```dockerfile
# Bad: Use latest tag
FROM python:latest

# Good: Use specific version
FROM python:3.9-slim
```

### 2. **Don't Run as Root**
```dockerfile
# Bad: Run as root
FROM python:3.9
CMD ["python", "app.py"]

# Good: Use non-root user
FROM python:3.9
RUN useradd -m appuser
USER appuser
CMD ["python", "app.py"]
```

---

*Follow these best practices to build secure and efficient Docker containers! 🎯*