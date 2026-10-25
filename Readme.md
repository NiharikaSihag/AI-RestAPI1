# LLM REST API

A FastAPI service that sends text to an LLM through OpenRouter and returns its sentiment, a short summary, and up to five topics as JSON.

## Requirements

- Python 3.9 or newer
- An OpenRouter API key
- An OpenRouter model ID that supports structured outputs, such as `openai/gpt-4o-mini`

Install the dependencies:

```powershell
pip install fastapi uvicorn openai pydantic
```

## Configuration

The app reads two environment variables at startup. If either is missing, the app fails to start with a `KeyError`.

| Variable | Description |
| --- | --- |
| `OPENROUTER_API_KEY` | Your OpenRouter API key |
| `OPENAI_MODEL` | OpenRouter model ID, for example `openai/gpt-4o-mini` |

Set them in PowerShell for the current session:

```powershell
$env:OPENROUTER_API_KEY = "sk-or-..."
$env:OPENAI_MODEL = "openai/gpt-4o-mini"
```

## Running the server

From the project folder:

```powershell
uvicorn main:app --reload
```

The server runs at `http://127.0.0.1:8000`. Interactive API docs are at `http://127.0.0.1:8000/docs`.

## Endpoint

### `POST /analyze`

Request body (`application/json`):

```json
{
  "text": "I love this product, it works great."
}
```

- `text` is required, must be 1 to 10,000 characters, and no other fields are allowed.

Successful response (`200`):

```json
{
  "sentiment": "positive",
  "summary": "The user praises a product that works well.",
  "topics": ["product", "quality"]
}
```

- `sentiment` is one of `positive`, `negative`, or `neutral`.
- `summary` is 3 to 500 characters.
- `topics` is a list of up to 5 strings.

### Example

```powershell
curl.exe -X POST http://127.0.0.1:8000/analyze `
  -H "Content-Type: application/json" `
  -d '{"text": "I love this product, it works great."}'
```

## Error responses

| Status | Meaning |
| --- | --- |
| `422` | The request body is invalid, or the model refused the request |
| `502` | The model returned no result, an invalid result, or the provider connection failed |
| `503` | The provider's rate limit was reached |
| `504` | The provider request timed out |

## Notes

- The model is instructed to treat the input text as data, not as instructions.
- Requests to the provider time out after 30 seconds, with one retry.
 