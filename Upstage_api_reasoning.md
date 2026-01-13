# Upstage Solar Open API: Reasoning Guide

## Overview

This guide explains how to use the **Reasoning** feature in Upstage Solar Open API. Reasoning allows you to see the model's thought process before it generates the final answer.

### Get Your API Key

👉 [Get API Key from Upstage Console](https://console.upstage.ai/api-keys)

---

## Quick Start

### Enable Reasoning

Add `reasoning_effort` parameter to your request:

```bash
curl https://api.upstage.ai/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "solar-open",
    "messages": [{"role": "user", "content": "What is 15% of 80?"}],
    "reasoning_effort": "high"
  }'
```

---

## The `reasoning_effort` Parameter

The `reasoning_effort` parameter controls whether reasoning is enabled.

| Value | Reasoning | Description |
|-------|-----------|-------------|
| `high` | ✅ ON | Reasoning enabled, best for complex problems |
| `medium` | ✅ ON | Reasoning enabled (default) |
| `low` | ❌ OFF | No reasoning, fastest response |

**Default value:** `medium` (reasoning enabled)

---

## Response Format

### When Reasoning is ON (`high` or `medium`)

The response includes a `reasoning` field with the model's thought process:

```json
{
  "id": "chatcmpl-abc123",
  "model": "solar-open",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "reasoning": "The user wants to calculate 15% of 80.\n1. Convert 15% to decimal: 15/100 = 0.15\n2. Multiply: 0.15 × 80 = 12\nThe answer is 12.",
        "content": "15% of 80 is **12**."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "completion_tokens": 133,
    "prompt_tokens": 37,
    "total_tokens": 170,
    "prompt_tokens_details": {
      "audio_tokens": 0,
      "cached_tokens": 32
    }
  }
}
```

### When Reasoning is OFF (`low`)

The `reasoning` field is not included:

```json
{
  "id": "chatcmpl-abc123",
  "model": "solar-open",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "15% of 80 is **12**."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "completion_tokens": 10,
    "prompt_tokens": 37,
    "total_tokens": 47,
    "prompt_tokens_details": {
      "audio_tokens": 0,
      "cached_tokens": 0
    }
  }
}
```

---

## Streaming

When streaming is enabled, reasoning is sent **first**, followed by the content.

### Stream Sequence

**Phase 1: Reasoning**
```
data: {"choices": [{"delta": {"role": "assistant"}}]}
data: {"choices": [{"delta": {"reasoning": "The user"}}]}
data: {"choices": [{"delta": {"reasoning": " wants to"}}]}
data: {"choices": [{"delta": {"reasoning": " calculate..."}}]}
```

**Phase 2: Content**
```
data: {"choices": [{"delta": {"content": "15%"}}]}
data: {"choices": [{"delta": {"content": " of 80"}}]}
data: {"choices": [{"delta": {"content": " is **12**."}}]}
data: [DONE]
```

### Streaming Example (Python)

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.upstage.ai/v1"
)

stream = client.chat.completions.create(
    model="solar-open",
    messages=[{"role": "user", "content": "What is 15% of 80?"}],
    reasoning_effort="high",
    stream=True
)

reasoning_started = False
content_started = False

for chunk in stream:
    delta = chunk.choices[0].delta
    
    if hasattr(delta, 'reasoning') and delta.reasoning:
        if not reasoning_started:
            print("[Reasoning]")
            reasoning_started = True
        print(delta.reasoning, end='')
    
    if hasattr(delta, 'content') and delta.content:
        if not content_started:
            print("\n[Answer]")
            content_started = True
        print(delta.content, end='')
```

---

## Python Examples

### Basic Usage

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.upstage.ai/v1"
)

# Enable reasoning
response = client.chat.completions.create(
    model="solar-open",
    messages=[{"role": "user", "content": "Solve: 3x + 5 = 20"}],
    reasoning_effort="high"
)

message = response.choices[0].message

# Access reasoning (only available when reasoning_effort is high or medium)
if hasattr(message, 'reasoning'):
    print(f"Thinking: {message.reasoning}")

print(f"Answer: {message.content}")
```

### Choosing reasoning_effort

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.upstage.ai/v1"
)

# For complex problems - use high for thorough reasoning
response = client.chat.completions.create(
    model="solar-open",
    messages=[{"role": "user", "content": "Prove that √2 is irrational"}],
    reasoning_effort="high"
)

# Default behavior - medium reasoning
response = client.chat.completions.create(
    model="solar-open",
    messages=[{"role": "user", "content": "What is the capital of France?"}]
    # reasoning_effort defaults to "medium"
)

# For simple questions - disable reasoning for faster response
response = client.chat.completions.create(
    model="solar-open",
    messages=[{"role": "user", "content": "Hi, how are you?"}],
    reasoning_effort="low"
)
```

---

## Token Usage

The `usage` object provides token counts:

```json
"usage": {
  "completion_tokens": 133,
  "prompt_tokens": 37,
  "total_tokens": 170,
  "prompt_tokens_details": {
    "audio_tokens": 0,
    "cached_tokens": 32
  }
}
```

- `cached_tokens`: Number of prompt tokens served from cache

---

## Summary

| Feature | Description |
|---------|-------------|
| **Parameter** | `reasoning_effort` |
| **Values** | `high` (ON) / `medium` (ON, default) / `low` (OFF) |
| **Output Field** | `reasoning` |
| **Streaming** | Reasoning first, then content |
