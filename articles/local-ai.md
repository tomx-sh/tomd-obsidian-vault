---
publish: true
created: 2026-07-31
---

# Running AI models locally
#ai

Local AI performance depends on the model, quantization, runtime, and request settings. Measure both generation speed and the delay before the first token.

## Apple Silicon benchmark

Tested on **4 October 2026**: MacBook Pro M1 Pro, 32 GB unified memory, macOS 27.0.1. Magnitude version: 0.2.5.

Each configuration received the same prompt: “Write a detailed essay about the history of computing.” Temperature was 0, with a 256-token output limit, one discarded warmup, and three measured runs. Only one model was loaded at a time. Values below are medians; Ollama and LM Studio results are from earlier runs on the same machine.

| Runtime | Model | Generation (tokens/s) | First token (s) |
|---|---|---:|---:|
| Magnitude | Gemma 4 26B-A4B Q4 QAT, GGUF | **31.55** | 0.228 |
| Ollama | `gemma4:12b-mlx` | 20.71 | 0.061 |
| Ollama | `qwen3.8:27b-mlx` | 11.26 | 0.271 |
| LM Studio | `qwen/qwen3.8-27b` | 10.62 | 2.047 |
| LM Studio | `qwen3.8-27b-uncensored-mlx` | 10.66 | 2.003 |
| LM Studio | `qwen2.5-0.5b-instruct-mlx` | 244.58 | 0.103 |

Magnitude completed 256 tokens in **8.37 seconds**, with measured generation rates of 31.54–31.86 tokens/s. Its Gemma configuration was 1.52× faster than the Ollama Gemma 12B configuration.

These results compare complete configurations, not runtime efficiency or answer quality. Models and quantization differ; Magnitude used 256K context, while the earlier runs requested 8K. Some baseline runs generated hidden reasoning; Magnitude generated visible text only.

Magnitude and LM Studio timings come from streamed responses. Ollama uses server timings, with first-token latency approximated by load plus prompt-processing time. These short-prompt results do not measure long-context or agent performance.

Magnitude's local OpenAI-compatible API base URL:

```text
http://127.0.0.1:10100/inference/v1
```

## Earlier hardware notes

Model candidates recorded in July 2026:

| Hardware | Models |
|---|---|
| Raspberry Pi, 8 GB | `qwen3:4b`, `qwen3:1.7b`, `gemma4:e2b-it-qat` |
| Raspberry Pi, 16 GB | `qwen3:8b`, `gemma4:e4b-it-qat`, `gemma4:12b-it-qat` |
| M4 MacBook Air, 32 GB | `gemma4:e4b-mlx` |
| [[hp-elitedesk-g4-ubuntu\|HP EliteDesk G4 AMD]], 8 GB | `gemma4:e2b-it-qat` |

Earlier coding trials favored `qwen3.6:35b-a3b-coding-nvfp4` in Ollama. oMLX trials were less satisfactory; no timed comparison is included here.

Dedicated hardware options to investigate: AMD Threadripper Pro on WRX90 and [Sipeed K3](https://sipeed.com/k3).

## Reducing agent latency

Earlier OpenCode tests on a MacBook Air M4 with 24 GB found delays from hidden reasoning, automatic title generation, large prompts, and competing requests that disrupted prompt-cache reuse.

The useful changes were:

- Disable automatic title generation and make reasoning optional.
- Keep one model loaded and serialize requests.
- Limit working context and enable conversation compaction.

OpenCode settings used in those tests:

```json
{
  "agent": {
    "title": { "disable": true }
  },
  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 4096
  }
}
```

Ollama environment:

```text
OLLAMA_KEEP_ALIVE=30m
OLLAMA_MAX_LOADED_MODELS=1
OLLAMA_NUM_PARALLEL=1
```

That setup used a 32,768-token context and a 4,096-token output limit. Disabling title generation reduced visible first-token latency from roughly 5–10 seconds to under one second on repeated Qwen 2B requests. Stable cache reuse mattered more than decode speed.

## Related tools

- Speech: [`blaizy/mlx-audio`](https://github.com/blaizy/mlx-audio)
- Headless browser: [`obscura`](https://github.com/h4ckf0r0day/obscura)
- Fine-tuning: [Gemma Trainer walkthrough](https://dev.to/googleai/master-local-fine-tuning-with-gemma-trainer-3ipp)
