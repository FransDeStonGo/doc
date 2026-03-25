---
type: guide
tags: [ollama, vllm, local-llm, quantization, gpu, inference]
status: complete
updated: 2026-03-25
related:
  - ai/models.md
  - infrastructure/deployment.md
  - agents/cost-optimization.md
---

# Local LLM Setup Guide

Complete guide to running LLMs locally with Ollama, vLLM, and quantization techniques.

---

## Why Local LLMs?

**Pros:**
- Zero API costs after initial setup
- Full data privacy (no external API calls)
- No rate limits
- Offline capability
- Custom fine-tuning possible

**Cons:**
- Requires GPU (8GB+ VRAM minimum)
- Slower than cloud APIs
- Limited to smaller models (<70B parameters)
- Maintenance overhead

---

## 1. Ollama Setup

### Installation

```bash
# Linux
curl -fsSL https://ollama.com/install.sh | sh

# macOS
brew install ollama

# Start service
ollama serve
```

---

### Pull Models

```bash
# Small models (8GB VRAM)
ollama pull llama3.1:8b          # General purpose
ollama pull qwen2.5-coder:7b     # Code generation
ollama pull nomic-embed-text     # Embeddings (274MB)

# Medium models (16GB VRAM)
ollama pull llama3.1:70b         # High quality
ollama pull deepseek-r1:14b      # Reasoning

# Embedding models
ollama pull bge-m3               # Multilingual (1.2GB)
```

---

### Python Client

```python
import requests

def ollama_chat(prompt: str, model: str = "llama3.1:8b") -> str:
    """Chat with Ollama model."""
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": model,
            "prompt": prompt,
            "stream": False
        }
    )
    return response.json()["response"]

# Usage
answer = ollama_chat("What is RAG?", model="llama3.1:8b")
print(answer)
```

---

### Streaming Response

```python
def ollama_stream(prompt: str, model: str = "llama3.1:8b"):
    """Stream tokens from Ollama."""
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={"model": model, "prompt": prompt, "stream": True},
        stream=True
    )
    
    for line in response.iter_lines():
        if line:
            chunk = json.loads(line)
            if not chunk.get("done"):
                print(chunk["response"], end="", flush=True)

# Usage
ollama_stream("Write a poem about AI")
```

---

### Tool Use with Ollama

```python
import json

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                },
                "required": ["location"]
            }
        }
    }
]

def ollama_tools(prompt: str, tools: list, model: str = "llama3.1:8b") -> dict:
    """Use tools with Ollama."""
    response = requests.post(
        "http://localhost:11434/api/chat",
        json={
            "model": model,
            "messages": [{"role": "user", "content": prompt}],
            "tools": tools,
            "stream": False
        }
    )
    return response.json()

# Usage
result = ollama_tools("What's the weather in London?", tools)
print(result["message"]["tool_calls"])
```

---

## 2. vLLM Setup

vLLM provides faster inference with PagedAttention and continuous batching.

### Installation

```bash
# Install vLLM
pip install vllm

# Verify GPU
nvidia-smi
```

---

### Start Server

```bash
# Start vLLM OpenAI-compatible server
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-3.1-8B-Instruct \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 1

# With quantization (AWQ)
python -m vllm.entrypoints.openai.api_server \
    --model TheBloke/Llama-2-13B-chat-AWQ \
    --quantization awq \
    --port 8000
```

---

### Python Client

```python
from openai import OpenAI

# Point to local vLLM server
client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="dummy"  # vLLM doesn't require real key
)

def vllm_chat(prompt: str, model: str = "meta-llama/Llama-3.1-8B-Instruct") -> str:
    """Chat with vLLM server."""
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        max_tokens=512,
        temperature=0.7
    )
    return response.choices[0].message.content

# Usage
answer = vllm_chat("Explain quantum computing")
print(answer)
```

---

## 3. Quantization

Quantization reduces model size and memory usage with minimal quality loss.

### Quantization Methods

| Method | Size Reduction | Quality Loss | Speed | Use Case |
|--------|---------------|--------------|-------|----------|
| FP16 | 50% | None | Fast | Default for GPU |
| INT8 | 75% | Minimal | Faster | Production |
| INT4 (GPTQ) | 87.5% | Small | Fastest | Consumer GPU |
| AWQ | 87.5% | Minimal | Fastest | Best quality/size |
| GGUF | Variable | Variable | Fast | CPU inference |

---

### GGUF (CPU Inference)

```bash
# Download GGUF model
ollama pull llama3.1:8b-q4_K_M  # 4-bit quantized

# Or use llama.cpp directly
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make

# Run model
./main -m models/llama-3.1-8b.Q4_K_M.gguf -p "Hello, world!" -n 128
```

---

### GPTQ (GPU Inference)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

# Load GPTQ quantized model
model = AutoModelForCausalLM.from_pretrained(
    "TheBloke/Llama-2-13B-chat-GPTQ",
    device_map="auto",
    trust_remote_code=False,
    revision="main"
)

tokenizer = AutoTokenizer.from_pretrained("TheBloke/Llama-2-13B-chat-GPTQ")

# Generate
inputs = tokenizer("What is AI?", return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=100)
print(tokenizer.decode(outputs[0]))
```

---

### AWQ (Best Quality)

```python
from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

# Load AWQ model
model = AutoAWQForCausalLM.from_quantized(
    "TheBloke/Llama-2-13B-chat-AWQ",
    fuse_layers=True
)

tokenizer = AutoTokenizer.from_pretrained("TheBloke/Llama-2-13B-chat-AWQ")

# Generate
inputs = tokenizer("Explain RAG", return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=200)
print(tokenizer.decode(outputs[0]))
```

---

## 4. GPU Requirements

### VRAM Requirements by Model Size

| Model Size | FP16 VRAM | INT8 VRAM | INT4 VRAM | Recommended GPU |
|-----------|-----------|-----------|-----------|-----------------|
| 7B | 14GB | 7GB | 4GB | RTX 3060 12GB |
| 13B | 26GB | 13GB | 7GB | RTX 4090 24GB |
| 30B | 60GB | 30GB | 15GB | A100 40GB |
| 70B | 140GB | 70GB | 35GB | 2x A100 80GB |

---

### Multi-GPU Setup

```python
# vLLM with tensor parallelism
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-3.1-70B-Instruct \
    --tensor-parallel-size 2  # Split across 2 GPUs

# Transformers with device_map
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.1-70B-Instruct",
    device_map="auto",  # Automatically split across GPUs
    torch_dtype="float16"
)
```

---

## 5. Performance Optimization

### Batch Processing

```python
# vLLM batch inference
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")
sampling_params = SamplingParams(temperature=0.7, max_tokens=100)

prompts = [
    "What is AI?",
    "Explain quantum computing",
    "How does RAG work?"
]

outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(output.outputs[0].text)
```

---

### KV Cache Optimization

```python
# Enable KV cache in vLLM (default)
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-3.1-8B-Instruct \
    --max-model-len 4096 \
    --gpu-memory-utilization 0.9  # Use 90% of GPU memory
```

---

### Flash Attention

```bash
# Install flash-attention
pip install flash-attn --no-build-isolation

# vLLM automatically uses Flash Attention 2 if available
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-3.1-8B-Instruct
```

---

## 6. Model Recommendations

### For Code Generation
```bash
ollama pull qwen2.5-coder:7b     # Best quality
ollama pull deepseek-coder:6.7b  # Fast
ollama pull codellama:13b        # Balanced
```

### For Chat
```bash
ollama pull llama3.1:8b          # General purpose
ollama pull mistral:7b           # Fast
ollama pull gemma2:9b            # Google's model
```

### For Reasoning
```bash
ollama pull deepseek-r1:14b      # Chain-of-thought
ollama pull qwen2.5:14b          # Strong reasoning
```

### For Embeddings
```bash
ollama pull nomic-embed-text     # 768-dim, fast
ollama pull bge-m3               # 1024-dim, multilingual
```

---

## 7. Production Deployment

### Docker Setup

```dockerfile
FROM nvidia/cuda:12.1.0-runtime-ubuntu22.04

# Install Ollama
RUN curl -fsSL https://ollama.com/install.sh | sh

# Pull models
RUN ollama serve & sleep 5 && \
    ollama pull llama3.1:8b && \
    ollama pull nomic-embed-text

EXPOSE 11434

CMD ["ollama", "serve"]
```

---

### Docker Compose

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

volumes:
  ollama_data:
```

---

### Monitoring

```python
import requests

def ollama_health() -> dict:
    """Check Ollama health and loaded models."""
    try:
        # Check service
        response = requests.get("http://localhost:11434/api/tags")
        models = response.json()["models"]
        
        return {
            "status": "healthy",
            "models": [m["name"] for m in models],
            "count": len(models)
        }
    except Exception as e:
        return {"status": "unhealthy", "error": str(e)}

# Usage
health = ollama_health()
print(health)
```

---

## 8. Cost Comparison

### Cloud vs Local (1M tokens)

| Provider | Model | Cost | Local Equivalent |
|----------|-------|------|------------------|
| OpenAI | GPT-4 | $30 | Llama 3.1 70B (free) |
| Anthropic | Claude Sonnet | $3 | Llama 3.1 8B (free) |
| OpenAI | GPT-3.5 | $0.50 | Mistral 7B (free) |

**Break-even:** ~$500-1000 in API costs = GPU purchase justified

---

## Best Practices

1. **Start small:** Test with 7B models before scaling to 70B
2. **Quantize:** Use INT4/INT8 for consumer GPUs
3. **Batch:** Process multiple requests together for efficiency
4. **Cache:** Enable KV cache for faster inference
5. **Monitor:** Track GPU memory, latency, throughput
6. **Fallback:** Keep cloud API as backup for complex queries
7. **Update:** Regularly pull new model versions

---

## Troubleshooting

### Out of Memory
```bash
# Reduce context length
ollama run llama3.1:8b --ctx-size 2048

# Use smaller quantization
ollama pull llama3.1:8b-q4_K_M
```

### Slow Inference
```bash
# Enable GPU acceleration
export CUDA_VISIBLE_DEVICES=0

# Use Flash Attention
pip install flash-attn
```

### Model Not Found
```bash
# List available models
ollama list

# Pull missing model
ollama pull llama3.1:8b
```

---

## Related

- [Cost Optimization](cost-optimization.md) — when to use local vs cloud
- [Deployment Guide](../infrastructure/deployment.md) — Docker setup
- [Models Overview](../ai/models.md) — model comparison
