# Can You Call Multiple AI Models Using an OpenAI-Compatible Base URL?

> **TL;DR:** Yes. With CometAPI, you can keep the OpenAI SDK request shape, set the base URL to `https://api.cometapi.com/v1`, use a CometAPI key, and choose a current model ID in the `model` field. One request still calls one model; to call a different model, send another request with a different model ID.

This is the smallest setup I would use to confirm that the connection works before adding streaming, tools, structured output, or other model-specific features.

## Before You Run the Examples

Create a CometAPI key and store it in an environment variable. Set `MODEL_ID` to a current chat model from the CometAPI model catalog.

```bash
export COMETAPI_KEY="your-key"
export MODEL_ID="your-model-id"
```

Do not commit a real key to a public repository. The examples below all use the same base URL and model variable, so switching models does not require a new client.

## Python

Install the official OpenAI package with:

```bash
pip install openai
```

Then run:

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["COMETAPI_KEY"],
    base_url="https://api.cometapi.com/v1",
)

response = client.chat.completions.create(
    model=os.environ["MODEL_ID"],
    messages=[{"role": "user", "content": "Say hello in one sentence."}],
)

print(response.choices[0].message.content)
```

## Node.js

Install the official OpenAI package with:

```bash
npm install openai
```

Then run:

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.COMETAPI_KEY,
  baseURL: "https://api.cometapi.com/v1",
});

const response = await client.chat.completions.create({
  model: process.env.MODEL_ID,
  messages: [{ role: "user", content: "Say hello in one sentence." }],
});

console.log(response.choices[0].message.content);
```

## cURL

```bash
curl https://api.cometapi.com/v1/chat/completions \
  -H "Authorization: Bearer $COMETAPI_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$MODEL_ID"'",
    "messages": [{"role": "user", "content": "Say hello in one sentence."}]
  }'
```

## How to Call Another Model

Change `MODEL_ID` to another current chat-model ID and rerun the same request. The base URL, authentication header, and basic Chat Completions shape stay the same.

This is the practical meaning of calling multiple AI models through one OpenAI-compatible base URL.

Compatibility is at the API layer, not a promise that every model behaves identically. Context limits, supported roles, tool calling, structured output, multimodal input, and optional parameters can vary by model. Check the selected model's capabilities before adding advanced fields.

## Common Errors

| Error | What to check first |
|---|---|
| `400` | Confirm that `model` and `messages` are present, the JSON is valid, and the selected model accepts any optional fields you added. |
| `401` | Check that the key is current and sent as `Authorization: Bearer $COMETAPI_KEY`. |
| Redirect, HTML, or parsing error | Use `https://api.cometapi.com/v1` exactly. A wrong base URL or endpoint path may not return a clean JSON error. |
| `429` | Retry with exponential backoff and jitter rather than immediately repeating the request. |
| `5xx` | Keep the request ID. Fix the payload if the body says `invalid_request`; otherwise retry transient failures with backoff. |

## Conclusion

So, can you call multiple AI models using an OpenAI-compatible base URL? Yes: use one CometAPI client configuration and change the `model` value between requests.

Start with the minimal chat call above, then test the exact advanced capabilities your application needs instead of assuming all models expose the same feature set.
