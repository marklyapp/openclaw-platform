# Local LLM Setup (Optional)

Run a local language model for free inference. Useful for high-volume, simple tasks where you don't want to pay cloud API costs.

## Requirements

- NVIDIA GPU with 16+ GB VRAM (e.g., RTX 4090, RTX 3090, A6000)
- CUDA drivers installed
- A local LLM server with an OpenAI-compatible API

## Server Options

Any server that exposes an OpenAI-compatible `/v1/chat/completions` endpoint works:

| Server | Notes |
|--------|-------|
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | Lightweight, C++, GGUF models |
| [vLLM](https://github.com/vllm-project/vllm) | High throughput, production-grade |
| [Ollama](https://ollama.ai) | Easy setup, pull models by name |
| [LM Studio](https://lmstudio.ai) | GUI, one-click model downloads |

## Example: llama.cpp

```bash
# Start the server (adjust model path and GPU layers)
./llama-server \
  -m ./models/qwen3-coder.gguf \
  --host 0.0.0.0 \
  --port 8090 \
  -ngl 99 \
  -c 16384
```

## Configure openclaw.json

Add a custom model provider in your workflow's config:

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "local-gpu": {
        "baseUrl": "http://YOUR_GPU_IP:8090/v1",
        "api": "openai-completions",
        "apiKey": "local",
        "models": [
          {
            "id": "your-model-name",
            "name": "your-model-name",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 16384,
            "maxTokens": 8192,
            "compat": {
              "supportsDeveloperRole": false,
              "supportsReasoningEffort": false
            }
          }
        ]
      }
    }
  }
}
```

Then assign it to an agent:

```json
{
  "agents": {
    "list": [
      {
        "id": "my-local-agent",
        "model": "local-gpu/your-model-name"
      }
    ]
  }
}
```

## Cost Tracking

Set all cost fields to `0` for local models. OpenClaw tracks costs per-session — local models show as free in the dashboard.

## Concurrency

Local GPU models can typically handle only **one session at a time**. If you assign multiple agents to the same local model, they'll queue. Plan your workflow accordingly — route simple/sequential tasks to the local model and parallel tasks to cloud models.

## No GPU? Skip This

If you don't have a local GPU, just use cloud models for all agents. Change any agent using a local model to a cloud model:

```json
"model": "anthropic/claude-sonnet-4-6"
```
