# 🚀 Run on Another System - Complete Guide

## ✅ What You Need

1. **Docker Desktop** installed
2. **docker-compose.prod.yml** file (single file - 8 KB)
3. Your **LLM API key** (Anthropic/OpenAI/Bedrock)
4. Your **Jenkins URL and token**

---

## 📋 Step-by-Step Commands

### **Step 1: Install Docker (One-Time)**

**Windows/Mac:**
- Download: https://www.docker.com/products/docker-desktop
- Install and start Docker Desktop

**Linux:**
```bash
curl -fsSL https://get.docker.com | sh
sudo systemctl start docker
```

Verify:
```bash
docker --version
docker-compose --version
```

---

### **Step 2: Get docker-compose.prod.yml**

**Option A: Download from cloud**
```bash
# If you shared via Google Drive/Dropbox
# Download the file to your computer
```

**Option B: Create it manually**

Save this as `docker-compose.prod.yml`:

```yaml
version: "3.9"

services:
  mcp-gateway:
    image: prince243r3/lv-mcp-gateway:v1.0
    container_name: lv-mcp-gateway
    ports:
      - "9000:9000"
    environment:
      - ENVIRONMENT=demo
      - JENKINS_MASTERS={"local":{"url":"http://host.docker.internal:8080","user":"admin","token":"YOUR_JENKINS_TOKEN","csrf_enabled":true}}
    restart: unless-stopped
    networks:
      - lv-internal

  backend:
    image: prince243r3/lv-backend:v1.0
    container_name: lv-backend
    ports:
      - "8000:8000"
    environment:
      - LLM_PROVIDER=anthropic
      - LLM_API_KEY=sk-ant-api03-YOUR_API_KEY_HERE
      - LLM_MODEL=claude-3-5-sonnet-20241022
      - MCP_GATEWAY_URL=http://mcp-gateway:9000
      - USE_AWS_SECRETS=false
      - USE_DYNAMODB=false
      - USE_SQS=false
      - USE_REDIS=false
      - CORS_ORIGINS=http://localhost:5173,http://localhost:3000,http://localhost:80
      - YAML_OUTPUT_DIR=/app/output/yaml
      - TENANT_ID=lv-demo
      - JENKINS_MASTERS={"local":{"url":"http://host.docker.internal:8080","user":"admin","token":"YOUR_JENKINS_TOKEN","csrf_enabled":true}}
    volumes:
      - yaml_output:/app/output/yaml
    depends_on:
      - mcp-gateway
    restart: unless-stopped
    networks:
      - lv-internal

  frontend:
    image: prince243r3/lv-frontend:v1.0
    container_name: lv-frontend
    ports:
      - "5173:80"
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - lv-internal

networks:
  lv-internal:
    driver: bridge

volumes:
  yaml_output:
    driver: local
```

---

### **Step 3: Edit Configuration**

Open `docker-compose.prod.yml` in any text editor.

**Change these 2 values:**

**1. Your API Key** (find line with `LLM_API_KEY`):
```yaml
- LLM_API_KEY=sk-ant-api03-YOUR_ACTUAL_KEY_HERE
```

**2. Your Jenkins Details** (find `JENKINS_MASTERS`, appears twice):
```yaml
- JENKINS_MASTERS={"local":{"url":"http://your-jenkins.com:8080","user":"admin","token":"your-token","csrf_enabled":true}}
```

**Save the file.**

---

### **Step 4: Pull Images from Docker Hub**

```bash
docker-compose -f docker-compose.prod.yml pull
```

**Output:**
```
Pulling mcp-gateway ... done
Pulling backend     ... done
Pulling frontend    ... done
```

**Time:** ~2-5 minutes (downloads ~232 MB)

---

### **Step 5: Start Platform**

```bash
docker-compose -f docker-compose.prod.yml up -d
```

**Output:**
```
Creating network ... done
Creating volume ... done
Creating lv-mcp-gateway ... done
Creating lv-backend     ... done
Creating lv-frontend    ... done
```

**Time:** ~30 seconds

---

### **Step 6: Verify**

```bash
# Check containers are running
docker ps

# Check health
curl http://localhost:9000/health
curl http://localhost:8000/api/v1/health
```

**Expected:**
```json
{"status":"ok","kill_switch":false}
{"status":"ok","environment":"local"}
```

---

### **Step 7: Access Platform**

**Open browser:**
```
http://localhost:5173
```

**Also available:**
- API Docs: http://localhost:8000/api/docs
- Gateway: http://localhost:9000/health

---

## 🎯 Complete Copy-Paste Script

### **Windows (PowerShell)**

Save as `start-platform.ps1`:

```powershell
# Download or create docker-compose.prod.yml first
# Then edit it with your API key and Jenkins URL

Write-Host "Pulling images from Docker Hub..."
docker-compose -f docker-compose.prod.yml pull

Write-Host "Starting platform..."
docker-compose -f docker-compose.prod.yml up -d

Write-Host "Waiting 30 seconds for services to start..."
Start-Sleep -Seconds 30

Write-Host "Checking health..."
curl http://localhost:9000/health
curl http://localhost:8000/api/v1/health

Write-Host "`nPlatform ready at: http://localhost:5173"
Start-Process "http://localhost:5173"
```

Run:
```powershell
.\start-platform.ps1
```

---

### **Linux/Mac (Bash)**

Save as `start-platform.sh`:

```bash
#!/bin/bash

# Download or create docker-compose.prod.yml first
# Then edit it with your API key and Jenkins URL

echo "Pulling images from Docker Hub..."
docker-compose -f docker-compose.prod.yml pull

echo "Starting platform..."
docker-compose -f docker-compose.prod.yml up -d

echo "Waiting 30 seconds for services to start..."
sleep 30

echo "Checking health..."
curl http://localhost:9000/health
curl http://localhost:8000/api/v1/health

echo ""
echo "Platform ready at: http://localhost:5173"
open http://localhost:5173  # Mac
# xdg-open http://localhost:5173  # Linux
```

Run:
```bash
chmod +x start-platform.sh
./start-platform.sh
```

---

## 📋 Daily Usage Commands

### **Start** (if stopped)
```bash
docker-compose -f docker-compose.prod.yml start
```

### **Stop** (keeps containers)
```bash
docker-compose -f docker-compose.prod.yml stop
```

### **Restart** (after changes)
```bash
docker-compose -f docker-compose.prod.yml restart
```

### **View Logs**
```bash
# All services
docker-compose -f docker-compose.prod.yml logs -f

# Specific service
docker-compose -f docker-compose.prod.yml logs -f backend
```

### **Stop and Remove**
```bash
docker-compose -f docker-compose.prod.yml down
```

### **Complete Reset** (removes data too)
```bash
docker-compose -f docker-compose.prod.yml down -v
```

---

## 🛠️ Troubleshooting

### **"Port already in use"**

Check what's using the port:
```bash
# Windows
netstat -ano | findstr :5173

# Linux/Mac
lsof -i :5173
```

**Fix:** Change port in docker-compose.prod.yml:
```yaml
ports:
  - "8080:80"  # Changed from 5173
```

### **"Cannot pull images"**

Check internet connection and try again:
```bash
docker-compose -f docker-compose.prod.yml pull
```

If images are private, login first:
```bash
docker login
# Username: prince243r3
# Password: [Docker Hub password]
```

### **Services not healthy**

Wait longer (40-60 seconds), then check:
```bash
docker-compose -f docker-compose.prod.yml ps
docker-compose -f docker-compose.prod.yml logs backend
```

### **Frontend shows blank page**

Check backend is responding:
```bash
curl http://localhost:8000/api/v1/health
```

Restart frontend:
```bash
docker-compose -f docker-compose.prod.yml restart frontend
```

### **"No such file or directory"**

Make sure you're in the directory with docker-compose.prod.yml:
```bash
ls -la docker-compose.prod.yml
```

---

## 🌐 Different Jenkins Setups

### **Jenkins on same machine**
```yaml
- JENKINS_MASTERS={"local":{"url":"http://host.docker.internal:8080","user":"admin","token":"token"}}
```

### **Jenkins on remote server**
```yaml
- JENKINS_MASTERS={"prod":{"url":"https://jenkins.company.com","user":"admin","token":"token"}}
```

### **Multiple Jenkins masters**
```yaml
- JENKINS_MASTERS={"prod":{"url":"https://jenkins1.com","user":"admin","token":"tok1"},"staging":{"url":"https://jenkins2.com","user":"admin","token":"tok2"}}
```

---

## 🔑 Different LLM Providers

### **Using OpenAI instead of Anthropic**

Edit docker-compose.prod.yml:
```yaml
- LLM_PROVIDER=openai
- LLM_API_KEY=sk-your-openai-key-here
- LLM_MODEL=gpt-4
```

### **Using AWS Bedrock**

Edit docker-compose.prod.yml:
```yaml
- LLM_PROVIDER=bedrock
- BEDROCK_REGION=us-east-1
- BEDROCK_MODEL_ID=anthropic.claude-3-5-sonnet-20241022-v2:0
```

Make sure AWS credentials are configured:
```bash
aws configure
```

---

## ✅ Success Checklist

Platform is ready when you can:
- [ ] Access http://localhost:5173 (frontend loads)
- [ ] Browse Jenkins jobs
- [ ] Start intake (risk scoring works)
- [ ] See kill switch widget
- [ ] Approve waves
- [ ] Generate YAML files

---

## 🎉 Summary - Absolute Minimum

**For someone with Docker already installed:**

```bash
# 1. Get the file (download docker-compose.prod.yml)

# 2. Edit it (add API key and Jenkins URL)

# 3. Run these 2 commands:
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# 4. Open browser:
# http://localhost:5173
```

**That's it! 4 steps total.** 🚀

---

## 📦 What Gets Downloaded

When you run `pull`:
- **prince243r3/lv-frontend:v1.0** - 26 MB (React UI)
- **prince243r3/lv-backend:v1.0** - 119 MB (FastAPI + Agents)
- **prince243r3/lv-mcp-gateway:v1.0** - 87 MB (Security Gateway)

**Total:** ~232 MB (one-time download)

---

## 🔗 Useful Links

- **Docker Hub Images:** https://hub.docker.com/u/prince243r3
- **Docker Desktop:** https://www.docker.com/products/docker-desktop
- **Docker Compose Docs:** https://docs.docker.com/compose/

---

**Need help? Check the full README-DOCKER.md guide!**
