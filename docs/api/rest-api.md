# REST API Reference

The Leichte Sprache API provides REST endpoints for analyzing and generating German texts in Simple Language format.

## Base URL when used locally

```
http://localhost:8000
```

## Authentication

Currently, no authentication is required for analysis endpoints. LLM generation requires API keys to be configured server-side.

## Rate Limiting

- **Analysis**: 100 requests/minute per IP
- **Generation**: 10 requests/minute per IP (due to LLM costs)

## Content Type

All endpoints accept and return JSON:
```
Content-Type: application/json
```

## Endpoints Overview

| Endpoint | Method | Purpose | Rate Limit |
|----------|---------|---------|------------|
| `/analyse` | POST | Analyze text for violations | 100/min |
| `/generate` | POST | Generate Simple Language text | 10/min |
| `/health` | GET | Health check | Unlimited |
| `/info` | GET | API information | Unlimited |
| `/help/json-escaping` | GET | JSON formatting help | Unlimited |

## Text Analysis

### POST `/analyse`

Analyzes German text for Leichte Sprache compliance violations.

#### Parameters

**Query Parameters:**
- `format` (optional): Response format
  - `full` (default): Complete analysis with statistics and issues
  - `annotated_text`: Only annotated text

#### Request Body

```json
{
  "text": "string (required, max 50000 characters)"
}
```

!!! warning "JSON Escaping"
    Quotes in text must be escaped: `"Text mit \"Anführungszeichen\""`

#### Example Request

```bash
curl -X POST "http://localhost:8000/analyse" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Die komplexe Verwaltungsprozedur wurde durch die zuständigen Beamten implementiert."
  }'
```

#### Response (format=full)

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
  "issues": [
    {
      "rule_id": "regel_fremdwoerter_issue",
      "text": "komplexe",
      "message": "Möglicherweise Fremdwort - durch deutsches Wort ersetzen oder erklären.",
      "start": 4,
      "end": 12
    },
    {
      "rule_id": "regel_komposita_issue",
      "text": "Verwaltungsprozedur",
      "message": "Kompositum mit 3+ Wortteilen - aufteilen oder erklären.",
      "start": 13,
      "end": 32
    },
    {
      "rule_id": "regel_fremdwoerter_issue",
      "text": "implementiert",
      "message": "Möglicherweise Fremdwort - durch deutsches Wort ersetzen oder erklären.",
      "start": 75,
      "end": 88
    }
  ]
}
```

#### Response (format=annotated_text)

```json
{
  "annotated_text": "Die komplexe[regel_fremdwoerter: Fremdwort] Verwaltungsprozedur[regel_komposita: Kompositum] wurde durch die zuständigen Beamten implementiert[regel_fremdwoerter: Fremdwort]."
}
```

#### Response Schema

**AnalyseResponse:**
- `annotated_text` (string): Text with violation markers
- `statistics` (object): Summary statistics
  - `total_violations` (integer): Total number of violations
  - `unique_violations` (integer): Number of unique violations
  - `violations_by_rule` (object): Violations per rule
- `issues` (array): Detailed violation list
  - `rule_id` (string): Rule identifier
  - `text` (string): Problematic text segment
  - `message` (string): Description and suggestion
  - `start` (integer, optional): Character start position
  - `end` (integer, optional): Character end position

## Text Generation

### POST `/generate`

Transforms German text into Leichte Sprache using LLM technology.

#### Request Body

```json
{
  "text": "string (required, max 10000 characters)",
  "max_iterations": 10,
  "target_violations": 2,
  "provider": "openai",
  "model": "gpt-4o",
  "debug": "INFO"
}
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | string | - | Text to transform (max 10,000 chars) |
| `max_iterations` | integer | 10 | Maximum optimization iterations (1-15) |
| `target_violations` | integer | 2 | Stop when violations ≤ this number (0-10) |
| `provider` | string | "openai" | LLM provider: openai, google, ollama, mistral, anthropic |
| `model` | string | null | Specific model (uses provider default if null) |
| `debug` | string | "INFO" | Log level: INFO, WARN, DEBUG |

#### Supported Providers & Models

=== "OpenAI"
    ```json
    {
      "provider": "openai",
      "model": "gpt-4o"  // or gpt-4o-mini, gpt-3.5-turbo
    }
    ```

=== "Anthropic"
    ```json
    {
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20250929"  // or claude-opus-4-6, claude-haiku-4-5
    }
    ```

=== "Google"
    ```json
    {
      "provider": "google",
      "model": "gemini-3-flash"  // or gemini-3-pro
    }
    ```

=== "Ollama (Local)"
    ```json
    {
      "provider": "ollama",
      "model": "mistral-nemo:12b"  // or llama3:8b, llama3.1:8b
    }
    ```

=== "Mistral"
    ```json
    {
      "provider": "mistral",
      "model": "mistral-large-latest"  // or mistral-medium-latest
    }
    ```

#### Example Request

```bash
curl -X POST "http://localhost:8000/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Die 4m breite Feuerwehrzufahrt sowie der Reversierplatz für das Feuerwehrfahrzeug sind in jedem Fall freizuhalten.",
    "max_iterations": 10,
    "target_violations": 1,
    "provider": "openai",
    "model": "gpt-4o"
  }'
```

#### Response

```json
{
  "original": "Die 4m breite Feuerwehrzufahrt sowie der Reversierplatz für das Feuerwehrfahrzeug sind in jedem Fall freizuhalten.",
  "result": "Die Zufahrt für die Feuerwehr ist 4 Meter breit. Halten Sie die Zufahrt immer frei. Das gilt auch für den Platz zum Wenden.",
  "success": true,
  "iterations": 3,
  "final_violations": 0,
  "final_escalation_level": 0,
  "hix": 18.2,
  "hix_rating": "simple",
  "faithfulness_score": 4,
  "stop_reason": "target_reached",
  "iterations_detail": [
    {
      "iteration": 1,
      "text": "Die Feuerwehr-Zufahrt ist 4 Meter breit. Der Wendeplatz für Feuerwehr-Fahrzeuge muss frei bleiben.",
      "violations": 5
    },
    {
      "iteration": 2,
      "text": "Die Zufahrt für die Feuerwehr ist 4 Meter breit. Halten Sie die Zufahrt frei. Das gilt auch für den Wendeplatz.",
      "violations": 2
    },
    {
      "iteration": 3,
      "text": "Die Zufahrt für die Feuerwehr ist 4 Meter breit. Halten Sie die Zufahrt immer frei. Das gilt auch für den Platz zum Wenden.",
      "violations": 0
    }
  ],
  "provider": "openai",
  "model": "gpt-4o"
}
```

#### Response Schema

**GenerateResponse:**
- `original` (string): Original input text
- `result` (string): Generated Simple Language text
- `success` (boolean): Whether target violations achieved
- `iterations` (integer): Number of iterations performed
- `final_violations` (integer): Final violation count
- `final_escalation_level` (integer): Escalation level used (0-3)
- `hix` (number, optional): HIX readability score (0-20)
- `hix_rating` (string, optional): HIX rating category
- `faithfulness_score` (integer, optional): Semantic faithfulness (1-5)
- `stop_reason` (string, optional): Why iteration stopped
- `iterations_detail` (array): Details of each iteration
- `provider` (string): LLM provider used
- `model` (string): LLM model used

#### Escalation Strategies

When generation stagnates, the system automatically escalates:

| Level | Strategy | Description |
|-------|----------|-------------|
| 0 | Normal | All rules, standard temperature |
| 1 | Focused | Top 5 priority violations |
| 2 | High Temperature | More creative solutions (temp=0.9) |
| 3 | Rule-by-Rule | Focus on single rule category |

## System Endpoints

### GET `/health`

Health check endpoint for monitoring.

#### Response

```json
{
  "status": "healthy",
  "service": "Leichte Sprache API",
  "version": "1.0.0"
}
```

Possible status values:
- `healthy`: All systems operational
- `unhealthy`: Issues detected (check error field)

### GET `/info`

Returns API information and available rules.

#### Response

```json
{
  "api_name": "Leichte Sprache API",
  "version": "1.0.0",
  "description": "REST-API für die Analyse und Generierung deutscher Texte in Leichter Sprache",
  "endpoints": {
    "/analyse": "POST - Textanalyse auf Leichte-Sprache-Konformität",
    "/generate": "POST - Automatische Umwandlung in Leichte Sprache"
  },
  "verfuegbare_regeln": [
    "regel_fremdwoerter",
    "regel_satzlaenge",
    "regel_komposita"
  ],
  "format_optionen": {
    "full": "Vollständige Analyse mit Statistics und Issues",
    "annotated_text": "Nur annotierter Text"
  },
  "generator": {
    "verfuegbar": true,
    "regeln_geladen": 18,
    "provider": {
      "openai": {
        "verfuegbar": true,
        "modelle": ["gpt-4o", "gpt-4o-mini"],
        "standard": "gpt-4o"
      }
    }
  }
}
```

### GET `/help/json-escaping`

Provides help for properly formatting JSON requests with quotation marks.

## Error Handling

### Error Response Format

```json
{
  "error": "Description of the error"
}
```

### HTTP Status Codes

| Code | Meaning | Common Causes |
|------|---------|---------------|
| 200 | Success | Request completed successfully |
| 400 | Bad Request | Invalid input, malformed JSON |
| 503 | Service Unavailable | Missing API keys, model loading failed |
| 500 | Internal Server Error | Unexpected server error |
| 429 | Rate Limit Exceeded | Too many requests |

### Common Errors

#### 400 Bad Request

```json
{
  "error": "Text-Feld ist erforderlich"
}
```

Causes:
- Missing required `text` field
- Text exceeds maximum length
- Invalid JSON format

#### 503 Service Unavailable

```json
{
  "error": "OpenAI nicht verfügbar: OPENAI_API_KEY nicht gesetzt."
}
```

Causes:
- Missing LLM provider API key
- Provider package not installed
- External service unavailable

#### 500 Internal Server Error

```json
{
  "error": "Unerwarteter Serverfehler: Model loading failed"
}
```

Causes:
- Model file corruption
- Insufficient memory
- Configuration errors

## Best Practices

### Request Optimization

1. **Use appropriate format**: Use `format=annotated_text` for faster responses when you only need violation annotations

2. **Batch processing**: For multiple texts, send requests in parallel but respect rate limits

3. **Caching**: Cache analysis results for unchanged texts

4. **Error handling**: Implement retry logic with exponential backoff

### Text Preparation

1. **Clean input**: Remove unnecessary formatting and special characters

2. **Segment long texts**: Break very long documents into smaller chunks (< 5000 words)

3. **Escape quotes**: Always escape quotation marks in JSON: `\"`

### Generation Optimization

1. **Reasonable targets**: Set `target_violations` to 0-3 for best results

2. **Iteration limits**: Use 5-15 iterations depending on text complexity

3. **Provider selection**: Choose provider based on your needs:
   - **OpenAI**: Best quality, moderate speed
   - **Anthropic**: High quality, good for complex texts
   - **Google**: Fast, cost-effective
   - **Ollama**: Free, runs locally, requires setup

## Code Examples

### Python

```python
import requests

class LeichteSpracheClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url

    def analyze(self, text, format="full"):
        response = requests.post(
            f"{self.base_url}/analyse",
            json={"text": text},
            params={"format": format}
        )
        response.raise_for_status()
        return response.json()

    def generate(self, text, provider="openai", target_violations=2):
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": text,
                "provider": provider,
                "target_violations": target_violations
            }
        )
        response.raise_for_status()
        return response.json()

# Usage
client = LeichteSpracheClient()
result = client.analyze("Die komplexe Verwaltung organisiert eine Veranstaltung.")
print(f"Found {result['statistics']['total_violations']} violations")
```

### JavaScript

```javascript
class LeichteSpracheClient {
  constructor(baseUrl = 'http://localhost:8000') {
    this.baseUrl = baseUrl;
  }

  async analyze(text, format = 'full') {
    const response = await fetch(`${this.baseUrl}/analyse?format=${format}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text })
    });

    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }

  async generate(text, options = {}) {
    const body = {
      text,
      provider: 'openai',
      target_violations: 2,
      ...options
    };

    const response = await fetch(`${this.baseUrl}/generate`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    });

    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }
}

// Usage
const client = new LeichteSpracheClient();
const result = await client.analyze('Die komplexe Verwaltung organisiert eine Veranstaltung.');
console.log(`Found ${result.statistics.total_violations} violations`);
```

## OpenAPI Specification

The complete OpenAPI specification is available at:
- **Interactive docs**: `/docs` (Swagger UI)
- **ReDoc**: `/redoc`
- **JSON spec**: `/openapi.json`
- **YAML file**: Available in project root