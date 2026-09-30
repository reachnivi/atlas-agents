# Chapter 22 — Capstone: Atlas, the Autonomous Engineering Assistant

> **Atlas capstone** — **GitHub issue → plan → code → sandboxed tests → security review → human approval → pull request**, combining patterns from earlier chapters.

## TL;DR

The capstone combines the book's building blocks into one pipeline:

```
GitHub Issue ─▶ Planner ─▶ Coder ─▶ Tester (E2B) ──pass──▶ Reviewer ──approved──▶ Human Approval ─▶ Publish PR
                             ▲          │ fail                 │ needs_revision
                             └──────────┴──────────────────────┘      (both stop after MAX_RETRIES = 3)
```

| Chapter pattern | Where it shows up |
|---|---|
| Ch. 5 LangGraph state + conditional edges | `AtlasState`, `check_tests`, `check_review`, `build_graph()` |
| Ch. 6 roles | Technical Lead (planner), Software Engineer (coder), Security Reviewer |
| Ch. 12 sandboxes | `tester_node` runs `pytest` inside an E2B microVM |
| Ch. 14 guardrails | `filter_pii()` blocks secrets/PII in generated code *before* LLM review |
| Ch. 5 / Ch. 10 human-in-the-loop | `human_approval_node` before anything is published |
| Ch. 8 A2A federation | `online/atlas_with_a2a_security.py`, an external security auditor |
| Ch. 16 / Ch. 17 always-on | `online/atlas_nightly_runner.py`, a scheduled issue sweep |

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Typed pipeline state** | Issue, plan, current step, code changes, test results, review, approval, retry count, dry-run flag. | `AtlasState` |
| **Structured handoffs between nodes** | Planner → numbered list. Coder → JSON `{filename: content}`. Reviewer → JSON verdict. | node prompts |
| **Two feedback loops** | Test failures and review rejections both route back to the coder, and they share one retry budget. | `check_tests`, `check_review` |
| **Guardrail before LLM** | Regex PII/secret scan runs first, which is cheap and deterministic. | `reviewer_node` |
| **Dry run** | `--dry-run` goes through the whole pipeline without opening a PR. | `main()` |
| **Integration adapters** | Swap GitHub for GitLab by overriding only the fetch and publish nodes. | `online/atlas_gitlab_adapter.py` |
| **Outsourced capabilities (A2A)** | The reviewer calls an external agent through its Agent Card + task API. | `online/atlas_with_a2a_security.py` |
| **Nightly sweep** | Fetch `atlas-ready` issues, sort P0-first then oldest, post plans as comments, skip already-processed ones. | `online/atlas_nightly_runner.py` |

## Files

| File | What it shows |
|---|---|
| `atlas_capstone.py` | The capstone pipeline (uses the `gh` CLI for GitHub). |
| `online/atlas_gitlab_adapter.py` | GitLab issues/MRs via `python-gitlab`. |
| `online/atlas_with_a2a_security.py` | Serve a security agent over A2A and call it from Atlas. |
| `online/atlas_nightly_runner.py` | Cron-friendly batch runner for labeled issues. |

## What needs to be done

- [ ] `pip install anthropic langgraph e2b-code-interpreter`; set `ANTHROPIC_API_KEY`, `E2B_API_KEY`; install and authenticate `gh`.
- [ ] Create a test repo with a small, well-defined issue (for example "add a `slugify()` util with tests").
- [ ] Run with `--dry-run` first and review the plan, code, test output, and review verdict.
- [ ] Run for real and review the PR it opens.
- [ ] Start the A2A security agent and run the A2A variant.
- [ ] Wire in the Ch. 18 eval harness and the Ch. 20 fix loop as extra quality gates, then plan how you'd deploy it (Ch. 17/19).

## Run it

```bash
cd ch22_capstone
python atlas_capstone.py --issue 42 --repo myorg/myrepo --dry-run
python atlas_capstone.py --issue 42 --repo myorg/myrepo

python online/atlas_with_a2a_security.py --serve-security-agent --port 8001   # terminal 1
python online/atlas_with_a2a_security.py --issue 42 --repo myorg/myrepo       # terminal 2

python online/atlas_nightly_runner.py --repo myorg/myrepo --max-issues 5
```

## Production notes

- Without E2B the tester **skips and reports pass**. Don't rely on that outside demos.
- Review parse failures default to `approved` in the demo. In production, fail closed.
- Scope credentials narrowly: a bot token that can open PRs but not merge them.
- Keep the human approval step until your eval data justifies removing it (see Ch. 10's autonomy benchmark).

## Takeaways

1. A production agent is a pipeline of small, verifiable steps, not one large prompt.
2. Every step that talks to the outside world (fetch, publish) is an adapter you can swap.
3. The earlier chapters (graphs, sandboxes, guardrails, protocols, HITL, scheduling) fit together into one system.

**Prev:** [Ch. 21](../ch21_harness_engineer/README.md) · **Next:** [Chapter 23 — What's Next](../ch23_future/README.md)
