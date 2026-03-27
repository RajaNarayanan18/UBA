# 🚀 LV Migration Platform - Quick Start (3 Steps)

## Prerequisites
- Docker Desktop installed: https://www.docker.com/products/docker-desktop

---

## Step 1: Download docker-compose.prod.yml

Download the `docker-compose.prod.yml` file to your computer.

---

## Step 2: Edit 2 Values

Open `docker-compose.prod.yml` in any text editor (Notepad, VSCode, etc.) and change:

### **1. Your LLM API Key** (Line ~38)
```yaml
- LLM_API_KEY=sk-ant-api03-YOUR_API_KEY_HERE
```
Change to:
```yaml
- LLM_API_KEY=sk-ant-api03-your-actual-api-key
```

### **2. Your Jenkins Details** (Line ~23 and Line ~55)
```yaml
- JENKINS_MASTERS={"local":{"url":"http://host.docker.internal:8080","user":"admin","token":"YOUR_JENKINS_TOKEN_HERE","csrf_enabled":true}}
```
Change to:
```yaml
- JENKINS_MASTERS={"local":{"url":"http://your-jenkins-url:8080","user":"admin","token":"your-actual-token","csrf_enabled":true}}
```

**Save the file.**

---

## Step 3: Run

Open terminal/command prompt in the folder with `docker-compose.prod.yml`:

```bash
# Pull images from Docker Hub
docker-compose -f docker-compose.prod.yml pull

# Start platform
docker-compose -f docker-compose.prod.yml up -d

# Wait 30 seconds, then open browser
```

**Access:** http://localhost:5173

---

## ✅ That's It!

**3 steps:**
1. Download file
2. Edit 2 values (API key + Jenkins URL)
3. Run 2 commands

**Platform is ready!**

---

## 📋 Useful Commands

### View logs
```bash
docker-compose -f docker-compose.prod.yml logs -f
```

### Stop platform
```bash
docker-compose -f docker-compose.prod.yml down
```

### Restart
```bash
docker-compose -f docker-compose.prod.yml restart
```

---

## 🆘 Troubleshooting

### "Port already in use"
Edit `docker-compose.prod.yml` and change port `5173` to `8080`:
```yaml
ports:
  - "8080:80"  # Changed from 5173
```

### "Cannot pull images"
Make sure you have internet connection. Images will download from Docker Hub (prince243r3).

### Services not starting
Wait 40 seconds for health checks to pass:
```bash
docker ps
```

---

## 📞 Support

Check full documentation in `README-DOCKER.md` for:
- Complete demo workflow
- Architecture details
- Advanced troubleshooting
- Feature explanations

---

**Built with Docker by prince243r3**
