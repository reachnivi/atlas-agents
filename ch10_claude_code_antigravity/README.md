# Chapter 10 — Claude Code and Antigravity

> **Atlas v0.10** — run Atlas inside an **agentic coding harness**, governed by a behavioral contract (`AGENTS.md`), permission rules, and hooks (`.claude/settings.json`).

## TL;DR

Agentic coding tools (Claude Code, Antigravity) already provide a strong agent loop, file tools, and a terminal. Your job shifts from *building* the loop to *governing* it:

- **`AGENTS.md`** is the contract: what the agent may and must not do, behavior rules, a quality gate, and when to escalate.
- **`settings.json`** enforces it: an allow/deny list for shell commands and **hooks** that run automatically (lint after every edit, audit-log every command, run tests at stop).
- **Headless mode** (`claude --print`) runs the same agent in CI with no UI.

Prompt rules are *requests*. Permissions and hooks are *enforcement*. Use both.

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Behavioral contract** | Identity, capabilities (MAY / MUST NOT), behavior rules, quality gate, escalation triggers. | `AGENTS.md` |
| **Quality gate** | "Done" only when tests pass, new behavior is tested, the linter is clean, and changed files are listed. | `AGENTS.md` → Quality Gate |
| **Permission allow/deny** | `pytest*`, `ruff*`, `mypy*`, and read-only git are allowed; `git push`, `git commit`, `rm -rf`, `pip install` are denied. | `.claude/settings.json` |
| **PostToolUse hook** | After any Write/Edit, run `ruff check --fix` automatically. | `.claude/settings.json` |
| **PreToolUse hook** | Append every Bash command with a timestamp to `.claude/bash_audit.log`. | `.claude/settings.json` |
| **Stop hook** | Run the test suite when the agent says it's finished. | `.claude/settings.json` |
| **Autonomy levels** | Supervised → semi-autonomous → fully autonomous, each with its own trade-off in accuracy, speed, and human effort. | `online/autonomy_benchmarker.py` |
| **Contract testing** | Send adversarial prompts and check that the agent stays inside `AGENTS.md`. | `online/spec_validator.py` |
| **Headless CI** | `claude --print` for PR review, security scan, and test generation, with results posted as a PR comment. | `online/claude_code_ci.py` |

## Files

| File | What it shows |
|---|---|
| `AGENTS.md` | The chapter project: Atlas's behavioral contract. |
| `.claude/settings.json` | Model, permissions, and hooks that enforce the contract. |
| `online/autonomy_benchmarker.py` | Compares accuracy, cost, and interventions across autonomy levels. |
| `online/spec_validator.py` | Parses `AGENTS.md` into a contract and runs adversarial validation cases. |
| `online/claude_code_ci.py` | GitHub Actions wrapper for headless Claude Code review (includes a sample workflow in its docstring). |

## What needs to be done

- [ ] Copy `AGENTS.md` and `.claude/settings.json` into a sandbox repo with a `workspace/` and `tests/`.
- [ ] Open Claude Code there and ask for a small feature. Check that the hooks ran (lint output, `bash_audit.log`, tests at stop).
- [ ] Ask the agent to `git push` or `rm -rf` something and confirm the deny rules block it.
- [ ] Write one new MUST NOT rule in `AGENTS.md` and a matching deny/hook in `settings.json`.
- [ ] Run `spec_validator.py` against your agent endpoint and fix any contract violations.
- [ ] (Stretch) Wire `claude_code_ci.py` into a GitHub Actions workflow on a test repo.

## Run it

```bash
cd ch10_claude_code_antigravity
python online/autonomy_benchmarker.py --tasks 20
python online/spec_validator.py --agents-md ./AGENTS.md --agent-url http://localhost:8000
```

## Production notes

- Deny rules beat prompt rules. An instruction in `AGENTS.md` can be ignored or overridden by an injection; a denied command cannot run.
- Hooks make verification happen **every time** without depending on the model to remember.
- Start supervised, measure, and grant autonomy per task type as the data supports it.

## Takeaways

1. With a coding harness, your job becomes writing the contract and the enforcement.
2. `AGENTS.md` states intent, and `settings.json` makes it hold.
3. The same agent runs interactively and headless in CI.

**Prev:** [Ch. 9](../ch09_agent_skills/README.md) · **Next:** [Chapter 11 — Memory and Agentic RAG](../ch11_memory/README.md)
