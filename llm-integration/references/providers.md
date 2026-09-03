# Provider Reference: SDKs, Auth, and Patterns

---

## OpenAI

**Install:** `pip install openai`

**Auth:** `api_key` passed to `OpenAI(api_key=...)`

**Chat pattern:**
```python
from openai import OpenAI

client = OpenAI(api_key=api_key)
response = client.chat.completions.create(
    model=model,
    messages=messages,
    temperature=temperature,
    max_tokens=max_tokens,
)
return response.choices[0].message.content
```

**Streaming:**
```python
stream = client.chat.completions.create(..., stream=True)
for chunk in stream:
    if chunk.choices[0].delta.content:
        yield chunk.choices[0].delta.content
```

---

## Anthropic

**Install:** `pip install anthropic`

**Auth:** `api_key` passed to `Anthropic(api_key=...)`

**Note:** System prompt is a top-level parameter, NOT part of the messages list.

**Chat pattern:**
```python
from anthropic import Anthropic

client = Anthropic(api_key=api_key)
response = client.messages.create(
    model=model,
    max_tokens=max_tokens,
    system=system_prompt,
    messages=messages,  # must NOT include system role here
)
return response.content[0].text
```

**Streaming:**
```python
with client.messages.stream(
    model=model, max_tokens=max_tokens, system=system_prompt, messages=messages
) as stream:
    for text in stream.text_stream:
        yield text
```

**Model quirk:** `temperature` is supported but capped at 1.0 for most models.

---

## Google Gemini

**Install:** `pip install google-generativeai`

**Auth:** `genai.configure(api_key=api_key)`

**Note:** Gemini uses a different message format (`parts` instead of `content`).
Roles are `user` and `model` (not `assistant`).

**Chat pattern:**
```python
import google.generativeai as genai

genai.configure(api_key=api_key)
model_obj = genai.GenerativeModel(
    model_name=model,
    system_instruction=system_prompt,
    generation_config={"temperature": temperature, "max_output_tokens": max_tokens},
)
# Convert messages: role "assistant" → "model", content → parts
history = [{"role": "model" if m["role"] == "assistant" else m["role"],
            "parts": [m["content"]]} for m in messages[:-1]]
chat = model_obj.start_chat(history=history)
response = chat.send_message(messages[-1]["content"])
return response.text
```

**Streaming:**
```python
response = chat.send_message(messages[-1]["content"], stream=True)
for chunk in response:
    yield chunk.text
```

---

## Mistral

**Install:** `pip install mistralai`

**Auth:** `api_key` passed to `Mistral(api_key=...)`

**Note:** Mistral's SDK uses `client.chat.complete(...)` (synchronous).

**Chat pattern:**
```python
from mistralai import Mistral

client = Mistral(api_key=api_key)
response = client.chat.complete(
    model=model,
    messages=messages,
    temperature=temperature,
    max_tokens=max_tokens,
)
return response.choices[0].message.content
```

**Streaming:**
```python
stream = client.chat.stream(model=model, messages=messages, temperature=temperature, max_tokens=max_tokens)
for event in stream:
    if event.data.choices[0].delta.content:
        yield event.data.choices[0].delta.content
```

---

## Groq

**Install:** `pip install groq`

**Auth:** `api_key` passed to `Groq(api_key=...)`

**Note:** Groq's SDK is OpenAI-compatible. Same pattern as OpenAI.

**Chat pattern:**
```python
from groq import Groq

client = Groq(api_key=api_key)
response = client.chat.completions.create(
    model=model,
    messages=messages,
    temperature=temperature,
    max_tokens=max_tokens,
)
return response.choices[0].message.content
```

**Streaming:** Same as OpenAI streaming pattern but using `Groq` client.

---

## Ollama (local)

**Install:** `pip install openai`  ← uses OpenAI-compatible endpoint

**Auth:** No real API key needed. Pass `api_key="ollama"` as a dummy value.

**Base URL:** `http://localhost:11434/v1`

**Setup note:** Ollama must be running locally (`ollama serve`) and the model must be pulled
(`ollama pull llama3.2`) before use.

**Chat pattern:** Identical to OpenAI but with `base_url="http://localhost:11434/v1"`:
```python
from openai import OpenAI

client = OpenAI(api_key="ollama", base_url="http://localhost:11434/v1")
response = client.chat.completions.create(model=model, messages=messages, ...)
return response.choices[0].message.content
```

---

## OpenRouter

**Install:** `pip install openai`  ← uses OpenAI-compatible endpoint

**Auth:** `api_key` passed as Bearer token. Also pass `HTTP-Referer` and `X-Title` headers
(optional but recommended by OpenRouter for attribution).

**Base URL:** `https://openrouter.ai/api/v1`

**Chat pattern:**
```python
from openai import OpenAI

client = OpenAI(
    api_key=api_key,
    base_url="https://openrouter.ai/api/v1",
    default_headers={
        "HTTP-Referer": "https://github.com/your-project",  # optional
        "X-Title": "My LLM App",                            # optional
    },
)
response = client.chat.completions.create(model=model, messages=messages, ...)
return response.choices[0].message.content
```

**Model quirk:** Model names include the provider prefix, e.g. `mistralai/mistral-7b-instruct`.

---

## HuggingFace Inference API

**Install:** `pip install huggingface_hub`

**Auth:** `token` passed to `InferenceClient(token=api_key)`

**Note:** HuggingFace Inference API has two modes:
- **Serverless** (free tier, limited): works for many public models
- **Dedicated endpoints**: enterprise, custom URL

The skill uses the serverless `chat_completion` method (available for chat-compatible models).
For non-chat models (e.g., text-generation only), fall back to `text_generation`.

**Chat pattern:**
```python
from huggingface_hub import InferenceClient

client = InferenceClient(token=api_key)
response = client.chat_completion(
    model=model,
    messages=messages,
    temperature=temperature,
    max_tokens=max_tokens,
)
return response.choices[0].message.content
```

**Streaming:**
```python
stream = client.chat_completion(model=model, messages=messages, stream=True, ...)
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        yield delta
```

**Model quirk:** Gated models (e.g., Meta Llama) require the user to accept terms on
HuggingFace and use a Pro or paid account token.
