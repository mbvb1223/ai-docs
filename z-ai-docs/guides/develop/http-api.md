# Z.AI HTTP API

Z.AI offers HTTP API interfaces supporting multiple programming languages with cross-platform compatibility and RESTful design.

## API Setup

**General Endpoint:**
```
https://api.z.ai/api/paas/v4/
```

**Dedicated Coding Endpoint:**
```
https://api.z.ai/api/coding/paas/v4
```

**Required Headers:**
```
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY
```

## Getting Started

1. Register/login at [Z.AI Open Platform](https://z.ai)
2. Generate an API Key through the management dashboard
3. Include the key in request headers

## Authentication Methods

### API Key (Simplest)

```bash
curl 'https://api.z.ai/api/paas/v4/chat/completions' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "glm-4.7",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
```

### JWT Token (Higher Security)

Requires PyJWT installation. Generate tokens with 1-hour validity using split API key components and HS256 algorithm.

```python
import jwt
import time

def generate_jwt_token(api_key):
    parts = api_key.split(".")
    if len(parts) != 2:
        raise ValueError("Invalid API key format")

    key_id, secret = parts

    payload = {
        "api_key": key_id,
        "exp": int(time.time()) + 3600,  # 1 hour expiry
        "timestamp": int(time.time())
    }

    return jwt.encode(payload, secret, algorithm="HS256")
```

## Code Examples

### Python

```python
import requests

url = "https://api.z.ai/api/paas/v4/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_API_KEY"
}

data = {
    "model": "glm-4.7",
    "messages": [
        {"role": "user", "content": "Hello, how are you?"}
    ]
}

response = requests.post(url, headers=headers, json=data)
result = response.json()
print(result["choices"][0]["message"]["content"])
```

### JavaScript (Node.js)

```javascript
const url = "https://api.z.ai/api/paas/v4/chat/completions";

const response = await fetch(url, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_API_KEY"
  },
  body: JSON.stringify({
    model: "glm-4.7",
    messages: [
      { role: "user", content: "Hello, how are you?" }
    ]
  })
});

const data = await response.json();
console.log(data.choices[0].message.content);
```

### Java

```java
import okhttp3.*;
import com.fasterxml.jackson.databind.ObjectMapper;

public class HttpExample {
    public static void main(String[] args) throws Exception {
        OkHttpClient client = new OkHttpClient();
        ObjectMapper mapper = new ObjectMapper();

        String json = mapper.writeValueAsString(Map.of(
            "model", "glm-4.7",
            "messages", List.of(
                Map.of("role", "user", "content", "Hello!")
            )
        ));

        Request request = new Request.Builder()
            .url("https://api.z.ai/api/paas/v4/chat/completions")
            .addHeader("Content-Type", "application/json")
            .addHeader("Authorization", "Bearer YOUR_API_KEY")
            .post(RequestBody.create(json, MediaType.parse("application/json")))
            .build();

        try (Response response = client.newCall(request).execute()) {
            System.out.println(response.body().string());
        }
    }
}
```

## Streaming Responses

For streaming, set `stream: true` in the request:

```bash
curl -X POST 'https://api.z.ai/api/paas/v4/chat/completions' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "glm-4.7",
    "messages": [{"role": "user", "content": "Tell me a story"}],
    "stream": true
  }'
```

### Python Streaming

```python
import requests

url = "https://api.z.ai/api/paas/v4/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_API_KEY"
}

data = {
    "model": "glm-4.7",
    "messages": [{"role": "user", "content": "Tell me a story"}],
    "stream": True
}

response = requests.post(url, headers=headers, json=data, stream=True)

for line in response.iter_lines():
    if line:
        line = line.decode("utf-8")
        if line.startswith("data: "):
            print(line[6:])
```

## Error Handling

```python
import requests

try:
    response = requests.post(url, headers=headers, json=data)
    response.raise_for_status()
    result = response.json()
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 401:
        print("Invalid API key")
    elif e.response.status_code == 429:
        print("Rate limit exceeded")
    elif e.response.status_code == 500:
        print("Server error")
    else:
        print(f"HTTP error: {e}")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

## Best Practices

1. **Secure API keys** via environment variables, not hardcoded values
2. **Implement connection pooling** for high-traffic applications
3. **Use exponential backoff** retry logic for transient failures
4. **Monitor call frequency**, success rates, and response times
5. **Use HTTPS** in production with appropriate security measures
6. **Set appropriate timeouts** for requests

### Environment Variables

```bash
export ZAI_API_KEY="your-api-key"
```

```python
import os

api_key = os.environ.get("ZAI_API_KEY")
headers = {
    "Authorization": f"Bearer {api_key}"
}
```

## Rate Limits

Check documentation for current rate limits. Implement backoff when receiving 429 responses:

```python
import time
import requests

def make_request_with_retry(url, headers, data, max_retries=5):
    for attempt in range(max_retries):
        response = requests.post(url, headers=headers, json=data)

        if response.status_code == 429:
            wait_time = 2 ** attempt  # Exponential backoff
            time.sleep(wait_time)
            continue

        return response

    raise Exception("Max retries exceeded")
```
