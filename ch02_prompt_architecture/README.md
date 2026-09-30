# Chapter 2 — Prompt Architecture for Agents

> **Atlas v0.2** — treat the system prompt as code: version it, A/B test it, and measure it against an eval suite.

## TL;DR

The system prompt is the agent's configuration: it sets persona, scope, output format, and tool budget. Small wording changes move quality, cost, and refusal behavior a lot, so **don't tune prompts by feel**. Write a few variants, run them against the same eval cases, and compare the numbers.

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Prompt anatomy** | Identity ("You are Atlas…"), behavior rules, output format, tool list, and tool budget. | `VARIANT_*` in `prompt_ab_test.py` |
| **Prompt variants** | *Concise Analyst* (≤ 3 tools, bullets), *Thorough Researcher* (≤ 5 tools, multiple perspectives), *Structured Reporter* (fixed Summary/Findings/Sources/Confidence format). | `VARIANTS` |
| **Eval cases** | Each case has a question plus expected keywords, a tool budget, or `expected_refusal`. Out-of-scope questions ("write a poem", "should I buy Bitcoin?") test scope control. | `EVAL_CASES` |
| **Metrics** | Quality (% of cases with expected keywords), average tool calls, average latency, and refusal accuracy. | `print_summary()` |
| **Routing prompts** | A classifier prompt picks an intent, then a specialist persona answers. | `online/multi_persona_router.py` |
| **Chain of Density** | Iteratively rewrite a summary, adding more entities at the same length. Produces very dense briefs. | `online/chain_of_density_prompt.py` |
| **Injection heuristics** | Cheap regex checks for "ignore previous instructions"-style attacks. A first line of defense only (see Ch. 14). | `online/prompt_injection_detector.py` |

## What you build

`prompt_ab_test.py` runs **3 prompt variants × 10 eval cases** with one LLM call per case (it measures what the model *intends* to do, without executing tools) and prints a comparison table:

```
Variant                | Quality | Avg Tools | Avg Latency | Refusal
Concise Analyst        |    ..%  |    ..     |    ..ms     |   ..%
Thorough Researcher    |    ..%  |    ..     |    ..ms     |   ..%
Structured Reporter    |    ..%  |    ..     |    ..ms     |   ..%
```

## Files

| File | What it shows |
|---|---|
| `prompt_ab_test.py` | The chapter project: prompt A/B test harness. |
| `online/multi_persona_router.py` | Intent classification → dispatch to a specialist persona. |
| `online/chain_of_density_prompt.py` | Chain-of-Density prompt template (returns 4 progressively denser summaries). |
| `online/prompt_injection_detector.py` | Regex heuristics for obvious injection attempts. |

## What needs to be done

- [ ] Set `OPENAI_API_KEY` in `.env`.
- [ ] Run the A/B harness and note which variant wins on quality vs. tool usage vs. refusals. No single variant wins everything.
- [ ] Add a 4th variant (for example, add explicit refusal instructions to *Concise*) and re-run.
- [ ] Add 5 eval cases from your own domain, including at least one out-of-scope case.
- [ ] Try the router with billing, technical, and general questions.

## Run it

```bash
cd ch02_prompt_architecture
python prompt_ab_test.py
python online/multi_persona_router.py "I need help with my API integration"
```

## Production notes

- Keyword matching is a crude quality signal. Chapter 18 replaces it with LLM-as-judge and trajectory metrics.
- Refusal detection by phrase matching is fragile. Treat it as a smoke test.
- Keep prompts in version control next to their eval results. A prompt change is a behavior change.

## Takeaways

1. Prompts are code: diff them, review them, test them.
2. Always test on a fixed eval set before and after a prompt change.
3. Scope control (refusing off-topic requests) needs its own test cases.

**Prev:** [Ch. 1](../ch01_react_from_scratch/README.md) · **Next:** [Chapter 3 — Tools, Skills, and Structured Outputs](../ch03_tools_and_skills/README.md)
