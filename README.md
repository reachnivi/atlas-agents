# Atlas Agents — Hands-on AI Agents in Production

This repository contains the source code, examples, and project implementations for the book **"Hands-on AI Agents"** — a practical, code-first guide to building AI agents and, more importantly, putting them into production.

## 🚀 Book Overview

Where *Agentic Design Patterns* taught how to **write** agents, this book is about how to **run** them: the harnesses, guardrails, evals, loops, and deployment machinery that turn a demo into a system you can trust unattended. One project — **Atlas**, an autonomous engineering assistant — grows chapter by chapter from a fifty-line ReAct loop into a self-correcting, self-improving production system.

### Key Technologies
- **Claude API / OpenAI / Gemini**: multi-provider agent cores with structured outputs.
- **LangGraph & CrewAI**: stateful agent graphs and role-based multi-agent orchestration.
- **MCP & A2A**: universal tool connectivity and agent-to-agent discovery.
- **Claude Code & Antigravity**: agentic coding harnesses and loop primitives.
- **Agent Skills**: declarative, progressively-disclosed expertise (`SKILL.md`).
- **E2B / Docker sandboxes**: safe code execution boundaries.
- **Managed Agents**: server-run sessions, outcomes, and scheduled deployments.
- **LiteLLM, DSPy, Ollama**: model portability, routing, and local inference.

## 📁 Repository Structure

Each chapter folder holds the chapter's Atlas project. Extended examples that go beyond the printed text live in each chapter's `online/` subfolder.

Every chapter has its own **README** (linked below) with a TL;DR, the key concepts mapped to the code, a file guide, a hands-on checklist of what to do, run commands, and production notes. You can use them to learn the concepts without the book.

| Folder | Chapter |
|---|---|
| [`ch01_react_from_scratch/`](ch01_react_from_scratch/README.md) | Anatomy of an Agent — the minimal ReAct loop |
| [`ch02_prompt_architecture/`](ch02_prompt_architecture/README.md) | Prompt Architecture for Agents |
| [`ch03_tools_and_skills/`](ch03_tools_and_skills/README.md) | Tools, Skills, and Structured Outputs |
| [`ch04_handoffs/`](ch04_handoffs/README.md) | Handoffs and Routines — the support triage router |
| [`ch05_state_graphs/`](ch05_state_graphs/README.md) | Stateful Agent Graphs — LangGraph persistence and HITL |
| [`ch06_multi_agent/`](ch06_multi_agent/README.md) | Multi-Agent Collaboration — CrewAI and debate protocols |
| [`ch07_model_portability/`](ch07_model_portability/README.md) | One Agent, Many Models — LiteLLM, Ollama, DSPy |
| [`ch08_mcp_a2a/`](ch08_mcp_a2a/README.md) | Open Protocols — MCP servers and A2A discovery |
| [`ch09_agent_skills/`](ch09_agent_skills/README.md) | Agent Skills — the production skill library |
| [`ch10_claude_code_antigravity/`](ch10_claude_code_antigravity/README.md) | Claude Code and Antigravity |
| [`ch11_memory/`](ch11_memory/README.md) | Memory and Agentic RAG |
| [`ch12_sandboxes/`](ch12_sandboxes/README.md) | Code Execution and Sandbox Agents |
| [`ch13_multimodal/`](ch13_multimodal/README.md) | Multimodal and Voice Agents |
| [`ch14_guardrails/`](ch14_guardrails/README.md) | Guardrails and Agent Safety |
| [`ch15_agent_harness/`](ch15_agent_harness/README.md) | Agent Harness Engineering |
| [`ch16_always_on_agents/`](ch16_always_on_agents/README.md) | Always-On Agents — daemons, watchdogs, recovery |
| [`ch17_managed_agents/`](ch17_managed_agents/README.md) | Managed Agents — let the platform run it |
| [`ch18_evaluation/`](ch18_evaluation/README.md) | Evaluation and Observability |
| [`ch19_deployment/`](ch19_deployment/README.md) | Deployment, Async Agents, and Security |
| [`ch20_loop_engineering/`](ch20_loop_engineering/README.md) | Loop Engineering — the self-correcting fix loop |
| [`ch21_harness_engineer/`](ch21_harness_engineer/README.md) | The Harness Engineer — /learn, adversarial pairs, prose verifiers |
| [`ch22_capstone/`](ch22_capstone/README.md) | Capstone: Atlas — the Autonomous Engineering Assistant |
| [`ch23_future/`](ch23_future/README.md) | What's Next — scaffold optimization and reasoning benchmarks |
| `shared/` | Global config and declarative skill models used across chapters |

## 🛠️ Prerequisites

- Python 3.10+
- API keys as needed per chapter: Anthropic, OpenAI, Google Gemini (see `shared/config.py` — keys load from a `.env` at the repo root)
- `pip install -r requirements.txt` (per-chapter extras are noted in each file's header)
- Basic understanding of LLM prompting and Python

## ▶️ Running the Examples

Every script is self-contained and documents its own usage and dependencies in its module docstring:

```bash
cd ch20_loop_engineering
python fix_loop.py --repo ./orders-service --goal "pytest green, ruff clean"
```
