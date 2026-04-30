# Configuration

This guide covers how to configure the Leichte Sprache API for different environments and use cases.

## Environment Variables

The API can be configured using environment variables. Create a `.env` file in the project root or set these variables in your environment:

### Core Configuration

```bash
# Server Settings
HOST=0.0.0.0                    # Server host (default: 0.0.0.0)
PORT=8000                       # Server port (default: 8000)
DEBUG=false                     # Debug mode (default: false)
LOG_LEVEL=INFO                  # Log level: DEBUG, INFO, WARNING, ERROR

# Performance
WORKERS=4                       # Number of worker processes
MAX_REQUESTS=1000              # Max requests per worker
TIMEOUT=300                    # Request timeout in seconds
```

### LLM Provider Configuration

Configure API keys for text generation features:

```bash
# OpenAI (recommended)
OPENAI_API_KEY=sk-your-openai-key

# Anthropic
ANTHROPIC_API_KEY=your-anthropic-key

# Google (Gemini)
GOOGLE_API_KEY=your-google-key
GEMINI_API_KEY=your-gemini-key  # Alternative

# Mistral
MISTRAL_API_KEY=your-mistral-key
```

### Model Configuration

```bash
# spaCy Model
SPACY_MODEL=de_core_news_lg     # German language model
MINIMAL_MODE=false              # Use smaller models for low memory

# ML Models
BERT_CACHE_DIR=./models/cache   # BERT model cache directory
MODEL_DOWNLOAD_TIMEOUT=300      # Model download timeout
```

## Configuration Files

### Production Configuration

Create `.env.production`:

```bash
# Security
DEBUG=false
LOG_LEVEL=WARNING

# Performance
WORKERS=8
MAX_REQUESTS=10000
TIMEOUT=60

# Monitoring
HEALTH_CHECK_INTERVAL=30
METRICS_ENABLED=true
```

### Development Configuration

Create `.env.development`:

```bash
# Development
DEBUG=true
LOG_LEVEL=DEBUG
RELOAD=true

# Performance (lighter for dev)
WORKERS=1
MAX_REQUESTS=100
```

## Docker Configuration

### Environment File

Create `docker.env`:

```bash
# Required for generation features
OPENAI_API_KEY=your-key-here

# Optional performance tuning
WORKERS=4
LOG_LEVEL=INFO
```

### Docker Compose

```yaml
version: '3.8'
services:
  api:
    image: acolono/leichte-sprache-api:latest
    ports:
      - "8000:8000"
    env_file:
      - docker.env
    environment:
      - HOST=0.0.0.0
      - PORT=8000
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
```

## Configuration Validation

Test your configuration:

```bash
# Validate configuration
python -c "
from config import validate_config
validate_config()
print('Configuration valid')
"

# Test API connectivity
curl -f http://localhost:8000/health || echo 'API not responding'
```

## Next Steps

After configuring your environment:

1. **[Test the setup](quick-start.md#testing-the-setup)**
2. **[Deploy to production](../architecture/system-overview.md)**
3. **[Monitor performance](../architecture/system-overview.md)**