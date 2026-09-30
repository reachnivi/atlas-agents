# Chapter 15 — Agent Harness Engineering

> **Atlas v0.15** — an **execution harness** between the model and the world: every file write, path, and shell command is validated, verified, and logged.

## TL;DR

The model *proposes* actions and the **harness** *decides* what actually happens. A good harness does four things, none of which depend on the model behaving:

1. **Path sandboxing**: resolve every path (`..`, symlinks) and refuse anything outside the workspace.
2. **Command allowlisting**: only approved executables run (for example `python`, `ruff`, `pytest`).
3. **Automatic verification**: every `.py` write is syntax-checked and linted by the harness itself, and the result goes back to the model.
4. **Trajectory logging**: every action is recorded for audit, replay, and later learning (Ch. 20–21).

```
model tool call ─▶ AgentHarness.execute_tool()
                     ├─ _validate_path()   → PermissionError if it escapes the sandbox
                     ├─ allowlist check    → "not on the command allowlist"
                     ├─ do the action
                     ├─ _verify_python()   → compile() + ruff (exit codes, not string matching)
                     └─ _record()          → trajectory log
```

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Harness vs. agent** | The agent reasons. The harness enforces. Security lives in the harness. | `AgentHarness` |
| **`Path.resolve()` + `is_relative_to`** | No `../../../etc/passwd` trick survives resolution. | `_validate_path()` |
| **Allowlist, not denylist** | Anything not explicitly allowed is blocked. | `_run_command()` |
| **Verification the model can't skip** | Linting runs inside the write, so the model can't skip it. | `_verify_python()` |
| **Progressive disclosure of references** | `read_reference("api_spec")` loads docs only when asked. | `_read_reference()` |
| **Swappable backends** | Replace local `subprocess` with an E2B microVM without changing the harness API. | `online/e2b_sandbox.py` |
| **Rollback** | Snapshot files before the first write and restore them if verification still fails after N retries. | `online/rollback_harness.py` |

## Files

| File | What it shows |
|---|---|
| `harness.py` | The chapter project. The demo writes a valid file, attempts a path-escape (blocked), and writes a broken file (verification catches it). |
| `online/e2b_sandbox.py` | `E2BAgentHarness`: same harness, commands run in a microVM. |
| `online/rollback_harness.py` | `RollbackHarness`: in-memory snapshots and automatic restore. |

## What needs to be done

- [ ] `pip install ruff` (no API key needed for the core demo).
- [ ] Run `harness.py` and read the three outcomes: success, blocked escape, verification failure.
- [ ] Try to escape the sandbox with a symlink or an absolute path. It should fail.
- [ ] Add `mypy` to the verification step and to the allowlist.
- [ ] Wire the harness under your Ch. 1 or Ch. 3 agent loop as its tool executor.
- [ ] Run `rollback_harness.py` and watch a file get restored after repeated failures.

## Run it

```bash
cd ch15_agent_harness
python harness.py
python online/rollback_harness.py
E2B_API_KEY=... python online/e2b_sandbox.py
```

## Production notes

- `shell=True` plus checking only the first word is still risky (`python -c ...`, `;`, `&&`). Parse with `shlex` and drop shell mode where possible.
- Keep the trajectory log. It feeds evals (Ch. 18), fix loops (Ch. 20), and `/learn` (Ch. 21).
- Verification output should be short and actionable, because the model reads it.

## Takeaways

1. Don't rely on the model for safety. The harness enforces it.
2. Verification built into the harness runs on every action, whatever the model does.
3. A good harness makes the agent both safer and better, because it gets fast, precise feedback.

**Prev:** [Ch. 14](../ch14_guardrails/README.md) · **Next:** [Chapter 16 — Always-On Agents](../ch16_always_on_agents/README.md)
