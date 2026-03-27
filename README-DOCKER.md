# LV Migration Platform - Docker Deployment Guide

**Jenkins to Azure DevOps Migration Platform - Production-Ready Demo**

## 🎯 Quick Start (5 Minutes)

### Prerequisites
- **Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux)
  - Download: https://www.docker.com/products/docker-desktop
  - Minimum: Docker 20.10+, Docker Compose 2.0+
- **4GB RAM** available
- **LLM API Key** (Anthropic Claude API, OpenAI, or AWS Bedrock access)
- **Jenkins credentials** (URL, username, token)

---

## 📦 Deployment Steps

### Step 1: Download Package
Extract the `lv-migration-platform.zip` to your working directory.

### Step 2: Configure Environment
```bash
# Copy the example file
cp .env.example .env

# Edit with your credentials (use notepad, nano, vim, etc.)
nano .env
```

**Required values in `.env`:**
```bash
DOCKER_USERNAME=yourdockerhubusername  # Pre-configured, don't change
VERSION=latest                          # Pre-configured, don't change

# LLM Configuration (choose one provider)
LLM_PROVIDER=anthropic
LLM_API_KEY=sk-ant-api03-your-actual-key-here

# Jenkins Configuration
JENKINS_MASTERS={"local":{"url":"http://your-jenkins-url:8080","user":"admin","token":"your-jenkins-token"}}
```

### Step 3: Start Platform
```bash
# Pull images from Docker Hub (fast, ~1 minute)
docker-compose -f docker-compose.prod.yml pull

# Start all services
docker-compose -f docker-compose.prod.yml up -d
```

### Step 4: Verify Services
```bash
# Check all services are running
docker-compose -f docker-compose.prod.yml ps

# Check health
curl http://localhost:9000/health  # MCP Gateway
curl http://localhost:8000/api/v1/health  # Backend API
```

### Step 5: Access Platform
Open your browser:
- **Web UI**: http://localhost:5173
- **API Docs**: http://localhost:8000/api/docs
- **MCP Gateway**: http://localhost:9000/health

---

## 🚀 Demo Workflow

### 1. Browse Jenkins Jobs
1. Go to **Scope** page
2. Select Jenkins master
3. Click **Browse Jenkins Jobs**
4. Select jobs for migration (checkbox selection)

### 2. Run Intake (Risk Assessment)
1. Click **Add to Session**
2. Go to **Waves** page
3. Click **Start Intake**
4. Watch Agent 1 analyze jobs:
   - Validates jobs exist in Jenkins
   - Detects ghost jobs
   - Fetches config.xml
   - **Risk scoring** (7 deterministic signals)
   - Assigns wave numbers

### 3. Review Risk Scores
- View risk breakdown (Low, Moderate, High, Critical, Extreme)
- See individual job scores before wave assignment
- Check scored jobs list with color-coded tiers

### 4. Approve Waves
1. Review wave assignments
2. Click **Approve** for Wave 0 (mandatory)
3. Approve additional waves as needed

### 5. Start Migration
1. Click **Start Migration**
2. Watch Agent 3 generate Azure Pipelines YAML
3. Monitor progress in real-time
4. Download generated YAML files

### 6. Test Kill Switch ⚡
1. **While migration is running**, click the red **Kill Switch** button
2. **All agents stop immediately** (within 2 seconds)
3. Status shows: "⚡ Kill switch activated - cannot proceed"
4. Click **Deactivate** to resume operations

### 7. Validate YAML
1. Go to **Validator** page
2. Select generated YAML files
3. Run 3-plane validation:
   - **Plane 1**: Syntactic (schema validation)
   - **Plane 2**: Security (forbidden patterns, secrets detection)
   - **Plane 3**: Semantic (LLM-based alignment check)

---

## 🛠️ Troubleshooting

### Services Not Starting
```bash
# Check logs
docker-compose -f docker-compose.prod.yml logs backend
docker-compose -f docker-compose.prod.yml logs mcp-gateway
docker-compose -f docker-compose.prod.yml logs frontend

# Restart specific service
docker-compose -f docker-compose.prod.yml restart backend
```

### "Connection refused" errors
```bash
# Check if services are healthy
docker ps

# Wait for health checks to pass (30-40 seconds)
docker-compose -f docker-compose.prod.yml ps
```

### LLM API errors
- **Anthropic**: Check API key starts with `sk-ant-api03-`
- **Bedrock**: Ensure AWS credentials are configured (`aws configure`)
- **Rate limits**: Platform has built-in exponential backoff and 1-second delays between jobs

### Jenkins connection issues
- Ensure Jenkins URL is accessible from Docker container
- Use `http://host.docker.internal:8080` for Jenkins on same machine
- Check Jenkins CSRF token is valid
- Verify Jenkins user has API access permissions

### Kill Switch not working
- Check MCP Gateway is healthy: `curl http://localhost:9000/health`
- View logs: `docker-compose -f docker-compose.prod.yml logs mcp-gateway`
- Restart gateway: `docker-compose -f docker-compose.prod.yml restart mcp-gateway`

---

## 📊 Platform Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Docker Network                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────┐ │
│  │  Frontend    │───▶│   Backend    │───▶│   MCP    │ │
│  │  (React +    │    │   (FastAPI   │    │ Gateway  │ │
│  │   nginx)     │    │  + Agents)   │    │          │ │
│  │              │    │              │    │          │ │
│  │ Port: 5173   │    │ Port: 8000   │    │Port: 9000│ │
│  └──────────────┘    └──────────────┘    └──────────┘ │
│         │                    │                  │      │
│         │                    ├─────────────────▶│      │
│         │                    │  Kill Switch     │      │
│         │                    │  Audit Logs      │      │
│         │                    │  Rate Limiting   │      │
│         │                    │                  │      │
│         └────────────────────┴──────────────────┘      │
│                                                         │
└─────────────────────────────────────────────────────────┘
                       │
                       ▼
            ┌─────────────────────┐
            │  External Systems   │
            ├─────────────────────┤
            │ • Jenkins (REST)    │
            │ • LLM Provider      │
            │   (Anthropic/AWS)   │
            └─────────────────────┘
```

### Components

**Frontend Container** (`lv-frontend`)
- React 18 + Vite + Tailwind CSS
- nginx static file server
- Responsive UI with real-time updates

**Backend Container** (`lv-backend`)
- FastAPI async framework
- 6 specialized agents:
  - **Agent 1**: Intake & risk scoring (deterministic, no LLM)
  - **Agent 2**: Plugin registry & mapping
  - **Agent 3**: YAML generation (LLM-powered)
  - **Agent 4**: 3-plane validation
  - **Agent 5**: Deployment orchestration
  - **Agent 6**: Wave orchestrator
- In-memory stores (session, job, plugin)
- Background task processing

**MCP Gateway Container** (`lv-mcp-gateway`)
- Centralized security & control layer
- **Kill switch**: Emergency halt for all operations
- Audit logging (ALLOWED/DENIED decisions)
- Rate limiting per tool/agent
- Tool server health monitoring

---

## 🔒 Security Features

### Kill Switch
- **Purpose**: Emergency stop for all migration operations
- **Scope**: Stops intake, migration, validation immediately
- **Response time**: Within 2 seconds
- **Check frequency**: Before each job, before LLM calls
- **UI indicator**: Real-time status widget (red/green)

### Audit Logging
- Every agent action logged with timestamp
- ALLOWED/DENIED decisions tracked
- Session ID and job ID correlation
- Exportable audit trail

### Rate Limiting
- Per-tool rate caps (GitHub push, Bedrock invoke, etc.)
- Per-agent authorization matrix
- Automatic exponential backoff on throttling

### No Secrets in Code
- All credentials via environment variables
- No hardcoded tokens or passwords
- `.env` file excluded from Docker image

---

## 📁 Data Persistence

### Generated YAML Files
Located in Docker volume `yaml_output`:
```bash
# View files
docker-compose -f docker-compose.prod.yml exec backend ls -la /app/output/yaml

# Copy to host
docker cp lv-backend:/app/output/yaml ./generated-yamls
```

### Session Data
- Stored in-memory (non-persistent for demo)
- Survives container restart via volume mount
- Reset with: `docker-compose -f docker-compose.prod.yml down -v`

---

## 🛑 Stopping Platform

```bash
# Stop all services (keeps data)
docker-compose -f docker-compose.prod.yml stop

# Stop and remove containers (keeps data)
docker-compose -f docker-compose.prod.yml down

# Stop and remove everything including volumes (clean slate)
docker-compose -f docker-compose.prod.yml down -v
```

---

## 🔧 Advanced Configuration

### Custom Ports
Edit `docker-compose.prod.yml`:
```yaml
frontend:
  ports:
    - "8080:80"  # Change 5173 to 8080
```

### Multiple Jenkins Masters
Edit `.env`:
```bash
JENKINS_MASTERS={"prod":{"url":"https://jenkins1.com","user":"admin","token":"tok1"},"staging":{"url":"https://jenkins2.com","user":"admin","token":"tok2"}}
```

### Enable Debug Logs
```bash
docker-compose -f docker-compose.prod.yml logs -f backend
docker-compose -f docker-compose.prod.yml logs -f mcp-gateway
```

### Resource Limits
Add to `docker-compose.prod.yml`:
```yaml
backend:
  deploy:
    resources:
      limits:
        cpus: '2.0'
        memory: 2G
```

---

## 📞 Support

### Common Issues
1. **Port already in use**: Change ports in docker-compose.prod.yml
2. **Out of memory**: Increase Docker Desktop memory limit (Settings → Resources)
3. **Slow LLM responses**: Normal for Bedrock under load, retry logic handles it
4. **Jenkins 403**: Check CSRF token and user permissions

### Demo Tips
- Start with 5-10 jobs for quick demo (not 100+)
- Show kill switch mid-migration for dramatic effect
- Highlight risk scores with color-coded visualization
- Demonstrate folder hierarchy in job browser

---

## 📄 License & Credits

**LV Migration Platform**
Jenkins to Azure DevOps Migration Suite
© 2026 - Internal Use Only

Built with:
- FastAPI (Python)
- React 18 + Vite
- Anthropic Claude API / AWS Bedrock
- Docker & Docker Compose

---

## 🎉 Success Indicators

Your demo is ready when you can:
1. ✅ Browse Jenkins jobs with folder hierarchy
2. ✅ Run intake and see risk scores
3. ✅ Approve waves and start migration
4. ✅ See YAML files generated in real-time
5. ✅ **Activate kill switch mid-migration and see agents stop**
6. ✅ Validate YAML with 3-plane validation
7. ✅ Download generated azure-pipelines.yml files

**Total demo time**: 10-15 minutes
**Wow factor**: Kill switch + real-time risk scoring + LLM YAML generation

---

*For technical questions, contact your platform administrator.*
