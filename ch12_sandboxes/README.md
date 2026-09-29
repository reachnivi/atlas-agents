# Chapter 12 — Code Execution and Sandbox Agents

> **Atlas v0.12** — a data-analyst agent that writes Python, runs it in an **E2B microVM**, reads the output or error, and repairs its code.

## TL;DR

Letting an agent **write and run code** is very powerful (data analysis, charts, math, tests) and very dangerous (it can delete files, leak data, or loop forever). The answer is a **sandbox**: a disposable, isolated machine where generated code runs. The agent loop becomes **write code → execute in sandbox → feed stdout/stderr back → fix or finish**.

```
task ─▶ LLM writes ```python``` ─▶ E2B sandbox.run_code() ─▶ output / error ─┐
          ▲                                                                 │
          └───────────── "Here's the error, please fix it" ─────────────────┘   (max 5 iterations)
```

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Code-as-action** | Instead of calling many small tools, the agent writes a program. | `CODE_AGENT_PROMPT` |
| **Execute → observe → repair** | Errors go back to the model as the next user message. | `run_data_analyst()` |
| **E2B sandbox** | A disposable cloud microVM: upload files, `run_code()`, `kill()` when done. | `data_analyst.py` |
| **Graceful fallback** | If E2B isn't available it falls back to a local subprocess with a 30 s timeout (dev only, **not isolation**). | `run_data_analyst()` |
| **Polyglot sandboxes** | The sandbox is full Linux, so it can run Python, compile and run Go, and run Bash. | `online/e2b_multi_language.py` |
| **Compute budgets** | Wrap the sandbox with a cumulative time budget, and kill it immediately when the budget is exceeded. | `online/sandbox_cost_tracker.py` |
| **Browser sandbox** | Headless Playwright renders the page (JS included) and strips it down to readable text. | `online/browser_agent_scraper.py` |

## Files

| File | What it shows |
|---|---|
| `data_analyst.py` | The chapter project: NL → pandas/matplotlib analysis in E2B. Generates `sample_sales.csv` automatically. |
| `online/e2b_multi_language.py` | Python, Go, and Bash in the same sandbox. |
| `online/sandbox_cost_tracker.py` | `BudgetedSandbox` that raises `BudgetExceededError` and kills the VM. |
| `online/browser_agent_scraper.py` | Playwright-based scraper that keeps only visible text. |

## What needs to be done

- [ ] `pip install e2b-code-interpreter anthropic`, and set `ANTHROPIC_API_KEY` and `E2B_API_KEY`.
- [ ] Run the default analysis and read each iteration: code written, output, fixes.
- [ ] Ask for something that will fail first (for example a column that doesn't exist) and watch the repair loop.
- [ ] Run `sandbox_cost_tracker.py` and see a runaway loop get killed.
- [ ] Unset `E2B_API_KEY` and see the local fallback, then think about why that's unsafe in production.

## Run it

```bash
cd ch12_sandboxes
python data_analyst.py "Analyze sales.csv and show monthly trends"
E2B_API_KEY=... python online/e2b_multi_language.py
E2B_API_KEY=... python online/sandbox_cost_tracker.py
python online/browser_agent_scraper.py https://example.com   # pip install playwright
```

## Production notes

- Never run model-generated code on the host. The local fallback exists only for development.
- Budget time **and** money per session, because agents can loop.
- Always `kill()` sandboxes in a `finally` block, since leaked VMs keep billing.
- Chapter 15 wraps sandboxed execution in a full harness (path checks, allowlists, verification).

## Takeaways

1. Writing code is the most flexible action an agent can take, and it must run in a sandbox.
2. The error message is the most useful thing to feed back into the loop.
3. Isolation, time limits, and cost limits are all required.

**Prev:** [Ch. 11](../ch11_memory/README.md) · **Next:** [Chapter 13 — Multimodal and Voice Agents](../ch13_multimodal/README.md)
