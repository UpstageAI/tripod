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
    "model": "tripod",
    "messages": [{"role": "user", "content": "What is 15% of 80?"}],
    "reasoning_effort": "high"
  }'
```

---

## The `reasoning_effort` Parameter

The `reasoning_effort` parameter controls whether reasoning is enabled and how deep the model thinks.

| Value | Reasoning | Description |
|-------|-----------|-------------|
| `high` | ✅ ON | Deep reasoning, best for complex problems |
| `medium` | ✅ ON | Balanced reasoning for general use |
| `low` | ❌ OFF | No reasoning, fastest response (default) |

**Default value:** `low` (reasoning disabled)

---

## Response Format

### When Reasoning is ON (`high` or `medium`)

The response includes a `reasoning` field with the model's thought process:

```json
{
  "id": "chatcmpl-abc123",
  "model": "tripod",
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
    "prompt_tokens": 15,
    "completion_tokens": 45,
    "total_tokens": 60,
    "completion_tokens_details": {
      "reasoning_tokens": 35
    }
  }
}
```

### When Reasoning is OFF (`low`)

The `reasoning` field is not included:

```json
{
  "id": "chatcmpl-abc123",
  "model": "tripod",
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
    "prompt_tokens": 15,
    "completion_tokens": 10,
    "total_tokens": 25,
    "completion_tokens_details": {
      "reasoning_tokens": 0
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
    model="tripod",
    messages=[{"role": "user", "content": "What is 15% of 80?"}],
    extra_body={"reasoning_effort": "high"},
    stream=True
)

for chunk in stream:
    delta = chunk.choices[0].delta
    
    if hasattr(delta, 'reasoning') and delta.reasoning:
        print(f"🧠 {delta.reasoning}", end='')
    
    if hasattr(delta, 'content') and delta.content:
        print(f"💬 {delta.content}", end='')
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

# Enable reasoning with high effort
response = client.chat.completions.create(
    model="tripod",
    messages=[{"role": "user", "content": "Solve: 3x + 5 = 20"}],
    extra_body={"reasoning_effort": "high"}
)

message = response.choices[0].message

# Access reasoning (only available when reasoning_effort is high or medium)
if hasattr(message, 'reasoning'):
    print(f"Thinking: {message.reasoning}")

print(f"Answer: {message.content}")
```

### Choosing reasoning_effort

```python
# For simple questions - use low (default, fastest)
response = client.chat.completions.create(
    model="tripod",
    messages=[{"role": "user", "content": "Hi, how are you?"}]
    # reasoning_effort defaults to "low"
)

# For moderate tasks - use medium
response = client.chat.completions.create(
    model="tripod",
    messages=[{"role": "user", "content": "Summarize this article..."}],
    extra_body={"reasoning_effort": "medium"}
)

# For complex problems - use high
response = client.chat.completions.create(
    model="tripod",
    messages=[{"role": "user", "content": "Prove that √2 is irrational"}],
    extra_body={"reasoning_effort": "high"}
)
```

---

## Token Usage

When reasoning is enabled, the `usage` object includes `reasoning_tokens`:

```json
"usage": {
  "prompt_tokens": 15,
  "completion_tokens": 45,
  "total_tokens": 60,
  "completion_tokens_details": {
    "reasoning_tokens": 35
  }
}
```

- `completion_tokens` = `reasoning_tokens` + content tokens
- Use `reasoning_tokens` for detailed cost tracking

---

## Best Practices

### 1. Choose the Right Effort Level

| Use Case | Recommended |
|----------|-------------|
| Simple Q&A, chitchat | `low` |
| Summarization, translation | `medium` |
| Math, logic, coding | `high` |

### 2. Exclude Reasoning from Conversation History

When building multi-turn conversations, only include `content` in the history:

```python
messages = []

# User message
messages.append({"role": "user", "content": user_input})

# Get response
response = client.chat.completions.create(
    model="tripod",
    messages=messages,
    extra_body={"reasoning_effort": "high"}
)

# Add only content to history (not reasoning)
messages.append({
    "role": "assistant",
    "content": response.choices[0].message.content
})
```

---

## Summary

| Feature | Description |
|---------|-------------|
| **Parameter** | `reasoning_effort` |
| **Values** | `high`, `medium` (ON) / `low` (OFF, default) |
| **Output Field** | `reasoning` |
| **Streaming** | Reasoning first, then content |
| **Token Tracking** | `completion_tokens_details.reasoning_tokens` |

