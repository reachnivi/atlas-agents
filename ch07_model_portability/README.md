# Chapter 7 — One Agent, Many Models (LiteLLM, Ollama, DSPy)

> **Atlas v0.7** — run the same agent task across several providers, compare cost, latency, and tool-calling reliability, and add automatic fallback.

## TL;DR

Don't hard-wire your agent to one vendor. **LiteLLM** gives you one `completion()` call for OpenAI, Anthropic, Gemini, and local **Ollama** models. With that you can **benchmark** models on *your* tasks, **route** each task to the cheapest model that handles it, and **fall back** when a provider is down. **DSPy** goes a step further: you declare inputs and outputs, and it *compiles* the prompt for you.

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Unified API** | `litellm.completion(model="anthropic/...", ...)` works for every provider. Only the model string changes. | `benchmark.py` |
| **Model benchmark** | Same cases on every model, measuring success, tool-call correctness, keywords, tokens, and cost (`litellm.completion_cost`). | `run_single_benchmark()` |
| **Tool-call reliability** | Models differ in *whether* they call a tool when they should, and not just in answer quality. | `expect_tool_call` in `BENCHMARK_CASES` |
| **Fallback router** | Try providers in priority order and put a failed provider on cooldown. | `online/model_router.py` |
| **Local / air-gapped** | Llama 3.1+ on Ollama supports native tool calls with no data leaving the machine. | `online/ollama_tool_calling.py` |
| **Declarative prompting (DSPy)** | Signatures + Modules + a metric; `compile()` tunes the prompt against examples. | `online/dspy_agent_pipeline.py` |
| **Key-based model discovery** | The benchmark includes only providers whose API key is set in `.env`. Ollama is always attempted. | `MODELS` list |

## Files

| File | What it shows |
|---|---|
| `benchmark.py` | The chapter project: 3 cases × every available model, printed as a comparison table. |
| `online/model_router.py` | Production fallback router with per-provider health and cooldown. |
| `online/ollama_tool_calling.py` | Fully local tool-calling loop against Ollama's HTTP API. |
| `online/dspy_agent_pipeline.py` | Research → fact-check pipeline compiled with DSPy. |

## What needs to be done

- [ ] `pip install litellm`, and set whichever of `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY` you have.
- [ ] (Optional) Install Ollama and `ollama pull llama3:8b` for the local row.
- [ ] Run the benchmark and write down which model is best on **cost per correct answer**, not only accuracy.
- [ ] Add a benchmark case from your own workload (for example log parsing or SQL generation).
- [ ] Break a provider (wrong key) and watch `model_router.py` fall back to the next one.
- [ ] Run the DSPy pipeline before and after `compile()` and compare.

## Run it

```bash
cd ch07_model_portability
python benchmark.py
python online/ollama_tool_calling.py          # needs `ollama serve`
python online/dspy_agent_pipeline.py          # pip install dspy-ai
```

## Production notes

- Benchmarks on generic questions tell you little. Benchmark on **your** traffic.
- Keep model IDs in config (`shared/config.py`, `.env`), never scattered through code.
- A router without health tracking keeps hammering a dead provider.
- Chapter 20 turns this static routing table into a *learned* one (`learned_router.py`).

## Takeaways

1. Model choice is a cost/latency/quality trade-off per task type, not a global decision.
2. An abstraction layer (LiteLLM) makes switching models a config change.
3. Local models are a real option for privacy-sensitive or offline workloads.

**Prev:** [Ch. 6](../ch06_multi_agent/README.md) · **Next:** [Chapter 8 — Open Protocols: MCP and A2A](../ch08_mcp_a2a/README.md)
