# Installation Guide

This guide covers various installation methods and advanced configuration options.

## System Requirements

### Minimum Requirements
- **Python**: 3.11+
- **Memory**: 8GB RAM
- **Storage**: 5GB free space (for models)
- **CPU**: 2+ cores recommended

### Recommended Setup
- **Python**: 3.12
- **Memory**: 16GB RAM
- **Storage**: 10GB free space
- **CPU**: 4+ cores
- **GPU**: Optional, CUDA-compatible for faster ML inference

## Installation Methods

### 1. Docker (Production)

The most reliable method for production deployments:

```bash
# Pull the latest image
docker pull acolono/leichte-sprache-api:latest

# Run with basic configuration
docker run -d \
  --name leichte-sprache \
  -p 8000:8000 \
  acolono/leichte-sprache-api:latest

# Run with LLM support
docker run -d \
  --name leichte-sprache \
  -p 8000:8000 \
  -e OPENAI_API_KEY=your_openai_key \
  -e ANTHROPIC_API_KEY=your_anthropic_key \
  acolono/leichte-sprache-api:latest
```

#### Docker Compose

For easier management:

```yaml title="compose.yaml"
version: '3.8'
services:
  api:
    image: acolono/leichte-sprache-api:latest
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - LOG_LEVEL=INFO
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

Start with:
```bash
docker-compose up -d
```

### 2. Local Development

#### Using uv (Recommended)

[uv](https://github.com/astral-sh/uv) is the fastest Python package manager:

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone repository
git clone https://github.com/acolono/leichte-sprache.git
cd leichte-sprache

# Setup environment
uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
uv sync

# Download language model
uv run python -m spacy download de_core_news_lg

# Start development server
uv run python api_main.py
```

#### Using pip

Traditional Python package management:

```bash
# Clone repository
git clone https://github.com/acolono/leichte-sprache.git
cd leichte-sprache

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Download language model
python -m spacy download de_core_news_lg

# Start development server
python api_main.py
```

#### Using conda

For scientific Python environments:

```bash
# Create conda environment
conda create -n leichte-sprache python=3.12
conda activate leichte-sprache

# Clone and install
git clone https://github.com/acolono/leichte-sprache.git
cd leichte-sprache
pip install -r requirements.txt

# Download language model
python -m spacy download de_core_news_lg
```

### 3. From Source

For contributors and advanced users:

```bash
# Clone repository
git clone https://github.com/acolono/leichte-sprache.git
cd leichte-sprache

# Install in development mode
pip install -e .

# Install development dependencies
pip install -r requirements-dev.txt

# Setup pre-commit hooks
pre-commit install

# Run tests
python test-suite/test_runner.py
```

## Model Downloads

The API requires several ML models that are downloaded automatically on first run:

### Core Models

```bash
# German language model (required)
python -m spacy download de_core_news_lg

# Alternative minimal model (lower memory)
python -m spacy download de_core_news_sm
```

### ML Rule Models

These download automatically but can be pre-cached:

```bash
# Pre-download ML models (optional)
python -c "
from regeln.abkuerzungen.regel import pruefe_regel
from regeln.mehrere_aussagen.regel import pruefe_regel as pruefe_aussagen
from regeln.komplexitaet.regel import pruefe_regel as pruefe_komplexitaet
print('Models downloaded successfully')
"
```

## Configuration

### Environment Variables

Create a `.env` file or set environment variables:

```bash title=".env"
# LLM Provider API Keys
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=your-anthropic-key
MISTRAL_API_KEY=your-mistral-key
GOOGLE_API_KEY=your-google-key

# Server Configuration
HOST=0.0.0.0
PORT=8000
LOG_LEVEL=INFO
DEBUG=false

# Performance Tuning
WORKERS=4
MAX_REQUESTS=1000
TIMEOUT=300

# Model Configuration
SPACY_MODEL=de_core_news_lg
MINIMAL_MODE=false
```

### Production Configuration

For production deployments:

```bash title=".env.production"
# Security
DEBUG=false
LOG_LEVEL=WARNING

# Performance
WORKERS=8
MAX_REQUESTS=10000
TIMEOUT=60

# Monitoring
HEALTH_CHECK_INTERVAL=30
PROMETHEUS_ENABLED=true
```

## Verification

### Test Installation

```bash
# Health check
curl http://localhost:8000/health

# API information
curl http://localhost:8000/info

# Quick analysis test
curl -X POST "http://localhost:8000/analyse" \
  -H "Content-Type: application/json" \
  -d '{"text": "Test installation."}'
```

### Expected Output

Successful installation should show:

```json
{
  "status": "healthy",
  "service": "Leichte Sprache API",
  "version": "1.0.0"
}
```

### Run Test Suite

```bash
# Run all tests
python test-suite/test_runner.py

# Test specific rule
python test-suite/test_runner.py fremdwoerter

# Verbose output
python test-suite/test_runner.py -v
```

## Troubleshooting

### Common Installation Issues

#### Model Download Fails

```bash
# Manual model download
wget https://github.com/explosion/spacy-models/releases/download/de_core_news_lg-3.7.0/de_core_news_lg-3.7.0.tar.gz
pip install de_core_news_lg-3.7.0.tar.gz
```

#### Memory Issues

```bash
# Use minimal configuration
export MINIMAL_MODE=true
export SPACY_MODEL=de_core_news_sm
python api_main.py
```

#### Permission Errors

```bash
# Fix permissions
chmod +x scripts/setup.sh
sudo chown -R $USER:$USER .
```

#### Port Conflicts

```bash
# Use different port
export PORT=8080
python api_main.py

# Or with uvicorn
uvicorn api_main:app --port 8080
```

### Performance Optimization

#### For Low-Memory Systems

```yaml title="docker-compose.override.yml"
version: '3.8'
services:
  api:
    environment:
      - MINIMAL_MODE=true
      - SPACY_MODEL=de_core_news_sm
    deploy:
      resources:
        limits:
          memory: 4G
```

#### For High-Performance

```yaml title="docker-compose.prod.yml"
version: '3.8'
services:
  api:
    environment:
      - WORKERS=16
      - MAX_REQUESTS=50000
    deploy:
      resources:
        limits:
          memory: 32G
          cpus: '8.0'
      replicas: 3
```

## Next Steps

After successful installation:

1. **[Configuration](configuration.md)**: Customize settings for your environment
2. **[Quick Start](quick-start.md)**: Run your first text analysis
3. **[API Reference](../api/rest-api.md)**: Learn about available endpoints
4. **Production Deployment**: Set up monitoring and scaling

## Need Help?

- **Installation Issues**: Check the GitHub Issues page
- **Performance Problems**: Review the system architecture documentation
- **Docker Issues**: Consult Docker documentation
- **Bug Reports**: File an issue on [GitHub](https://github.com/acolono/leichte-sprache/issues)