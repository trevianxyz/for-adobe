# Creative Automation Pipeline - Developer Setup Guide

## 🚀 Quick Start for New Developers

### Prerequisites

- **Docker** (latest version)
- **Git** (for cloning the repository)
- **API Keys** (Hugging Face and OpenAI - see setup below)

### 1. Clone and Setup

```bash
# Clone the repository
git clone <repository-url>
cd for-adobe

# Create environment file
cp .env.example .env
```

### 2. Configure API Keys

Edit `.env` file with your API keys:

```bash
# Required: OpenAI API Key (for LLM translation and fallback image generation)
OPENAI_API_KEY=sk-proj-your-openai-key-here

# Optional: Hugging Face API Key (for primary image generation)
HF_TOKEN=hf_your-huggingface-token-here
```

**Get API Keys:**

- **OpenAI**: https://platform.openai.com/api-keys
- **Hugging Face**: https://huggingface.co/settings/tokens

### 3. Build and Run

```bash
# Build the Docker image
docker build -t adobe-fastapi-app .

# Run the container with persistent storage
docker run -d -p 8080:80 --name adobe-fastapi-container \
  -v "$(pwd)/assets:/app/assets" \
  -v "$(pwd)/db:/app/db" \
  -v "$(pwd)/.env:/app/.env" \
  adobe-fastapi-app
```

### 4. Verify Installation

```bash
# Check container is running
docker ps --filter name=adobe-fastapi-container

# Test the API
curl http://localhost:8080/api/health

# Open the web interface
open http://localhost:8080
```

## 🌍 Features Overview

### **AI-Powered Campaign Generation**

- **Multi-Product Support**: Generate images for multiple products in one campaign
- **3 Size Variants**: Square (1:1), Landscape (16:9), Portrait (9:16)
- **Dual AI Providers**: Hugging Face (primary) + OpenAI DALL-E (fallback)
- **Smart Fallback**: Automatic failover if primary service is unavailable

### **Localization & Translation**

- **20+ Languages**: Automatic translation to native languages
- **Cultural Context**: Region-specific prompts for better relevance
- **Brand Overlay**: Werkr logo with localized text
- **Supported Regions**: US States, International countries

### **Enterprise Features**

- **Vector Search**: ChromaDB for campaign similarity
- **Analytics**: DuckDB for campaign tracking
- **Compliance**: Automated content validation
- **Persistent Storage**: All data survives container restarts

## 📁 Project Structure

```
for-adobe/
├── backend/                    # FastAPI application
│   └── app/
│       ├── main.py           # FastAPI app with lifespan events
│       ├── routes.py         # Campaign generation endpoints
│       ├── models/           # Pydantic data models
│       └── services/         # Business logic
│           ├── generator.py  # AI image generation + localization
│           ├── embeddings.py # ChromaDB vector search
│           ├── logging_db.py # DuckDB analytics
│           └── compliance.py # Content validation
├── frontend/                  # Web interface
│   ├── templates/           # Jinja2 HTML templates
│   └── static/             # CSS, JS, images
├── assets/                   # Generated content
│   └── generated/           # Campaign outputs
├── db/                      # Databases
│   ├── chroma/             # ChromaDB vector store
│   └── campaigns.duckdb    # DuckDB analytics
├── Dockerfile              # Container configuration
├── pyproject.toml          # Python dependencies
└── .env                    # API keys (create from .env.example)
```

## 🎯 Usage Examples

### Web Interface

1. Open http://localhost:8080
2. Fill out the campaign form:
   - **Products**: "safety helmet, work boots"
   - **Region**: "Mexico" (auto-translates to Spanish)
   - **Audience**: "construction workers"
   - **Message**: "Professional safety equipment"
3. Click "Generate Campaign"
4. View results with translated text and brand overlay

### API Usage

```bash
# Generate campaign via API
curl -X POST http://localhost:8080/campaigns/generate \
  -H "Content-Type: application/json" \
  -d '{
    "products": ["safety helmet", "work boots"],
    "region": "Germany",
    "audience": "construction workers",
    "message": "Professional safety equipment"
  }'
```

**Response:**

```json
{
  "campaign_id": "uuid-here",
  "outputs": {
    "1:1": "assets/generated/campaign_20251015_153700_uuid/safety_helmet/1x1/image_1x1.png",
    "16:9": "assets/generated/campaign_20251015_153700_uuid/safety_helmet/16x9/image_16x9.png",
    "9:16": "assets/generated/campaign_20251015_153700_uuid/safety_helmet/9x16/image_9x16.png"
  },
  "compliance": {
    "status": "approved",
    "issues": [],
    "message": "No compliance issues detected"
  }
}
```

## 🔧 Development Workflow

### Making Changes

```bash
# Stop container
docker stop adobe-fastapi-container

# Remove container
docker rm adobe-fastapi-container

# Rebuild with changes
docker build -t adobe-fastapi-app .

# Run updated container
docker run -d -p 8080:80 --name adobe-fastapi-container \
  -v "$(pwd)/assets:/app/assets" \
  -v "$(pwd)/db:/app/db" \
  -v "$(pwd)/.env:/app/.env" \
  adobe-fastapi-app
```

### Viewing Logs

```bash
# Container logs
docker logs adobe-fastapi-container

# Follow logs in real-time
docker logs -f adobe-fastapi-container

# Check specific services
docker logs adobe-fastapi-container | grep -E "(🚀|🔄|✅|⚠️|🌍|🏷️)"
```

### Database Access

```bash
# Query DuckDB analytics
docker exec adobe-fastapi-container /app/.venv/bin/python -c "
import duckdb
conn = duckdb.connect('db/campaigns.duckdb', read_only=True)
print(conn.execute('SELECT campaign_id, created_at, region, compliance_status FROM campaigns ORDER BY created_at DESC LIMIT 5').fetchall())
"

# Check ChromaDB collections
docker exec adobe-fastapi-container /app/.venv/bin/python -c "
import chromadb
client = chromadb.Client()
print(client.list_collections())
"
```

## 🐛 Troubleshooting

### Common Issues

**1. Container won't start:**

```bash
# Check logs
docker logs adobe-fastapi-container

# Common fixes:
# - Missing .env file
# - Invalid API keys
# - Port conflicts
```

**2. Images not displaying:**

```bash
# Check if assets are accessible
curl -I http://localhost:8080/assets/generated/campaign_*/product_name/1x1/image_1x1.png

# Verify volume mounts
docker inspect adobe-fastapi-container | grep -A 10 "Mounts"
```

**3. Translation not working:**

```bash
# Check OpenAI API key
docker exec adobe-fastapi-container printenv OPENAI_API_KEY

# Test translation manually
docker exec adobe-fastapi-container /app/.venv/bin/python -c "
from app.services.generator import translate_message_with_llm
print(translate_message_with_llm('Safety first', 'Mexico'))
"
```

**4. Port conflicts:**

```bash
# Check what's using port 8080
lsof -i :8080

# Use different port
docker run -d -p 8081:80 --name adobe-fastapi-container ...
```

### Performance Optimization

**For faster generation:**

- Use OpenAI DALL-E 3 (set `HF_TOKEN=""` to skip Hugging Face)
- Reduce image quality in `generator.py`
- Use smaller image sizes

**For better translations:**

- Upgrade to GPT-4 in `translate_message_with_llm()`
- Add more language mappings in `region_languages`

## 📊 Monitoring

### Health Checks

```bash
# API health
curl http://localhost:8080/api/health

# Container status
docker ps --filter name=adobe-fastapi-container

# Resource usage
docker stats adobe-fastapi-container
```

### Analytics Queries

```bash
# Campaign success rate
docker exec adobe-fastapi-container /app/.venv/bin/python -c "
import duckdb
conn = duckdb.connect('db/campaigns.duckdb', read_only=True)
result = conn.execute('SELECT compliance_status, COUNT(*) FROM campaigns GROUP BY compliance_status').fetchall()
print('Compliance Status:', result)
"

# Most popular regions
docker exec adobe-fastapi-container /app/.venv/bin/python -c "
import duckdb
conn = duckdb.connect('db/campaigns.duckdb', read_only=True)
result = conn.execute('SELECT region, COUNT(*) as campaigns FROM campaigns GROUP BY region ORDER BY campaigns DESC LIMIT 10').fetchall()
print('Top Regions:', result)
"
```

## 🚀 Production Deployment

### Environment Variables

```bash
# Production .env
OPENAI_API_KEY=sk-proj-your-production-key
HF_TOKEN=hf_your-production-token
```

### Docker Compose (Optional)

```yaml
# docker-compose.yml
version: "3.8"
services:
  creative-pipeline:
    build: .
    ports:
      - "8080:80"
    volumes:
      - ./assets:/app/assets
      - ./db:/app/db
      - ./.env:/app/.env
    restart: unless-stopped
```

### Security Considerations

- Rotate API keys regularly
- Use environment-specific keys
- Monitor API usage and costs
- Implement rate limiting for production

## 📚 API Documentation

- **Interactive Docs**: http://localhost:8080/docs
- **OpenAPI Spec**: http://localhost:8080/openapi.json
- **Health Check**: http://localhost:8080/api/health

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details

---

**Need help?** Check the logs first: `docker logs adobe-fastapi-container`
