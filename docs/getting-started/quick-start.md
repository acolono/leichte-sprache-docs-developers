# Quick Start

Get the Leichte Sprache API running in under 5 minutes!

## Prerequisites

- **Python 3.11+** (recommended: 3.12)
- **8GB RAM** minimum for ML models
- **Internet connection** for model downloads

## Option 1: Docker (Recommended)

The fastest way to get started:

```bash
# Pull and run the Docker image
docker run -p 8000:8000 acolono/leichte-sprache-api:latest
```

!!! success "Ready!"
    The API is now running at `http://localhost:8000`

    - Swagger UI: `http://localhost:8000/docs`
    - Health check: `http://localhost:8000/health`

### With LLM Generation

To enable text generation, provide an API key:

```bash
# OpenAI (recommended)
docker run -p 8000:8000 \
  -e OPENAI_API_KEY=your_key_here \
  acolono/leichte-sprache-api:latest

# Alternative providers
docker run -p 8000:8000 \
  -e ANTHROPIC_API_KEY=your_key_here \
  acolono/leichte-sprache-api:latest
```

## Option 2: Local Development

### 1. Clone the Repository

```bash
git clone https://github.com/acolono/leichte-sprache.git
cd leichte-sprache
```

### 2. Setup Environment

=== "uv (Recommended)"

    ```bash
    # Install uv if you haven't already
    curl -LsSf https://astral.sh/uv/install.sh | sh

    # Create virtual environment and install dependencies
    uv venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    uv sync
    ```

=== "pip"

    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    pip install -r requirements.txt
    ```

### 3. Download Language Model

```bash
# Required German NLP model (may take a few minutes)
uv run python -m spacy download de_core_news_lg
```

### 4. Start the API

```bash
# Development server with auto-reload
python api_main.py

# Or using uvicorn
uvicorn api_main:app --reload --host 0.0.0.0 --port 8000
```

!!! info "First Startup"
    The first startup takes 30-60 seconds to load all ML models. Subsequent starts are much faster.

## Your First Analysis

### Using curl

```bash
# Simple analysis
curl -X POST "http://localhost:8000/analyse" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Die komplexe Verwaltungsprozedur wurde durch die zuständigen Beamten implementiert."
  }'
```

Expected response (shortened):
```json
{
  "annotated_text": "Die komplexe[regel_fremdwoerter: Fremdwort] Verwaltungsprozedur[regel_komposita: Kompositum] wurde durch die zuständigen Beamten implementiert[regel_fremdwoerter: Fremdwort].",
  "statistics": {
    "total_violations": 3,
    "unique_violations": 3,
    "violations_by_rule": {
      "regel_fremdwoerter": 2,
      "regel_komposita": 1
    }
  },
  "issues": [...]
}
```

### Using the Swagger UI

1. Open `http://localhost:8000/docs` in your browser
2. Click "Try it out" on the `/analyse` endpoint
3. Enter your text in the request body
4. Click "Execute"

### Using Python

```python
import requests

# Analysis
response = requests.post(
    "http://localhost:8000/analyse",
    json={"text": "Die außerordentliche Verwaltung organisiert eine Veranstaltung."}
)

result = response.json()
print(f"Found {result['statistics']['total_violations']} violations")
print(f"Annotated: {result['annotated_text']}")
```

## Your First Generation

If you have an LLM API key configured:

```bash
curl -X POST "http://localhost:8000/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Die komplexe Verwaltungsprozedur wurde durch die zuständigen Beamten implementiert.",
    "provider": "openai",
    "target_violations": 1
  }'
```

Expected response:
```json
{
  "original": "Die komplexe Verwaltungsprozedur wurde...",
  "result": "Die Behörde hat ein neues Verfahren eingeführt.",
  "success": true,
  "iterations": 3,
  "final_violations": 0,
  "hix": 18.5,
  "hix_rating": "simple"
}
```

## Testing the Setup

Verify everything works:

```bash
# Health check
curl http://localhost:8000/health

# API info
curl http://localhost:8000/info

# Test with format parameter
curl -X POST "http://localhost:8000/analyse?format=annotated_text" \
  -H "Content-Type: application/json" \
  -d '{"text": "Einfacher Test."}'
```

## Common Issues

!!! warning "Model Download Failed"
    If `de_core_news_lg` download fails:
    ```bash
    # Alternative download method
    python -m pip install https://github.com/explosion/spacy-models/releases/download/de_core_news_lg-3.7.0/de_core_news_lg-3.7.0.tar.gz
    ```

!!! warning "Port Already in Use"
    If port 8000 is taken:
    ```bash
    # Use different port
    uvicorn api_main:app --port 8080

    # Or with Docker
    docker run -p 8080:8000 acolono/leichte-sprache-api:latest
    ```

!!! warning "Memory Issues"
    On systems with < 8GB RAM:
    ```bash
    # Reduce model size by setting environment variable
    export LEICHTE_SPRACHE_MINIMAL=true
    python api_main.py
    ```

## Next Steps

Now that you have the API running:

1. **[Explore the API](../api/rest-api.md)**: Learn about all available endpoints
2. **[System Architecture](../architecture/system-overview.md)**: Understand how the system works
3. **[Configuration](configuration.md)**: Customize the API for your needs
4. **[Contributing](../development/contributing.md)**: Help improve the API

## Development Mode

For development work:

```bash
# Run with debug logging
export DEBUG=1
python api_main.py

# Run tests
python test-suite/test_runner.py

# Check code style
ruff check .
black --check .
```

Ready to dive deeper? Continue with the [Installation Guide](installation.md) for advanced setup options.