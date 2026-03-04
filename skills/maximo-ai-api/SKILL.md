---
name: maximo-ai-api
description: 
  API reference for Maximo AI Platform. Use this skill whenever you need to interact with Maximo AI's chat completions API, including making inference requests, streaming responses, tool calling, and working with reasoning models.
  Trigger: "maximo ai", "maximo api", "maximo inference", "/maximo-ai-api"
license: MIT
user-invocable: true
allowed-tools:
  - Bash
  - Read
---

# Maximo AI API

Maximo AI provides access to proprietary Maximo AI models via a standardized, OpenAI-compatible endpoint.

## Base URL

```
https://api.maximoai.co/v1
```

## Authentication

All requests require an API key in the header:

```bash
curl -X POST "https://api.maximoai.co/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "model": "maximo-pandora-3-nano",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## API Compatibility

The Maximo AI API is **fully compatible with the OpenAI API specification**. You can use standard OpenAI SDKs by changing the `baseURL`:

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.maximoai.co/v1"
)

response = client.chat.completions.create(
    model="maximo-pandora-3-nano",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

## Chat Completions Endpoint

**Endpoint:** `POST /chat/completions`

### Request Body

| Parameter           | Type          | Required | Description                                        |
| ------------------- | ------------- | -------- | -------------------------------------------------- |
| `model`             | string        | Yes      | The model to use (e.g., "maximo-pandora-3-nano")   |
| `messages`          | array         | Yes      | Array of message objects with "role" and "content" |
| `stream`            | boolean       | No       | Whether to return streaming responses (SSE)        |
| `temperature`       | float         | No       | Sampling temperature (0.0 to 2.0)                  |
| `max_tokens`        | integer       | No       | Maximum tokens to generate                         |
| `top_p`             | float         | No       | Nucleus sampling parameter                         |
| `tools`             | array         | No       | Array of tool definitions for function calling     |
| `tool_choice`       | string/object | No       | Which tool to use                                  |
| `reasoning`         | boolean       | No       | Enable reasoning mode (if supported by model)      |
| `frequency_penalty` | float         | No       | Frequency penalty                                  |
| `presence_penalty`  | float         | No       | Presence penalty                                   |
| `stop`              | array/string  | No       | Stop sequences                                     |

### Message Format

```json
{
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": "What is Maximo AI?" },
    { "role": "assistant", "content": "Maximo AI is..." },
    { "role": "user", "content": "Tell me more." }
  ]
}
```

## Response Modes

### 1. Non-Streaming (stream: false)

Returns a standard JSON object once generation is complete:

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "created": 1763808444,
  "model": "maximo-pandora-3-nano",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you today?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30
  }
}
```

### 2. Streaming (stream: true)

Returns Server-Sent Events (SSE) adhering to the OpenAI `chat.completion.chunk` format:

```bash
curl -X POST "https://api.maximoai.co/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "model": "maximo-pandora-3-nano",
    "messages": [{"role": "user", "content": "Count to 5"}],
    "stream": true
  }'
```

Streaming response format:

```
data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":...,"model":"...","choices":[{"index":0,"delta":{"content":"1"},"finish_reason":null}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":...,"model":"...","choices":[{"index":0,"delta":{"content":" 2"},"finish_reason":null}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":...,"model":"...","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

## Tool Calling (Function Calling)

Maximo AI supports tool calling similar to OpenAI. Define tools and let the model decide when to use them:

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.maximoai.co/v1"
)

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="maximo-pandora-3-nano",
    messages=[
        {"role": "user", "content": "What's the weather in London?"}
    ],
    tools=tools
)

# Check if model wants to call a tool
tool_call = response.choices[0].message.tool_calls[0]
print(f"Tool: {tool_call.function.name}")
print(f"Arguments: {tool_call.function.arguments}")
```

## Available Maximo Models

Maximo AI provides access to proprietary Maximo models only. For the full list of available models, visit: https://maximoai.co/platform/models

## Error Handling

```json
{
  "error": {
    "message": "Invalid API key",
    "type": "invalid_request_error",
    "code": 401
  }
}
```

Common error codes:

- `401` - Invalid or missing API key
- `400` - Bad request (invalid parameters)
- `429` - Rate limit exceeded
- `500` - Internal server error

## SDK Examples

### Python

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.maximoai.co/v1"
)

# Non-streaming
response = client.chat.completions.create(
    model="maximo-pandora-3-nano",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)

# Streaming
stream = client.chat.completions.create(
    model="maximo-pandora-3-nano",
    messages=[{"role": "user", "content": "Count to 3"}],
    stream=True
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### JavaScript/Node.js

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.MAXIMO_API_KEY,
  baseURL: "https://api.maximoai.co/v1",
});

// Non-streaming
const response = await client.chat.completions.create({
  model: "maximo-pandora-3-nano",
  messages: [{ role: "user", content: "Hello!" }],
});
console.log(response.choices[0].message.content);

// Streaming
const stream = await client.chat.completions.create({
  model: "maximo-pandora-3-nano",
  messages: [{ role: "user", content: "Count to 3" }],
  stream: true,
});

for await (const chunk of stream) {
  if (chunk.choices[0].delta.content) {
    process.stdout.write(chunk.choices[0].delta.content);
  }
}
```

### cURL

```bash
# Non-streaming
curl -X POST "https://api.maximoai.co/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "model": "maximo-pandora-3-nano",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'

# Streaming
curl -X POST "https://api.maximoai.co/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "model": "maximo-pandora-3-nano",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": true
  }'
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement retry logic** with exponential backoff for transient errors
3. **Use streaming** for long responses to improve perceived latency
4. **Set appropriate timeouts** - longer for complex reasoning tasks
5. **Handle tool calls properly** - parse arguments as JSON
6. **Monitor token usage** via the `usage` field in responses
