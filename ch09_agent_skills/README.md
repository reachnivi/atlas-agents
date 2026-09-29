# Chapter 9 — Agent Skills: The Production Skill Library

> **Atlas v0.9** — package expertise as **declarative `SKILL.md` files**: YAML frontmatter + a procedure + a verification checklist, loaded only when needed.

## TL;DR

A **Skill** is a folder with a `SKILL.md`: a short **frontmatter** (`name`, `description`, `allowed-tools`) and a **body** with step-by-step process, common excuses to resist, and a verification checklist. The agent sees only the one-line *descriptions* in its catalog. When a request matches, it loads the full body (**progressive disclosure**). This gives the agent expert procedures without filling the context window, and non-engineers can write and review them.

```
catalog (names + descriptions only, cheap) ──match──▶ load full SKILL.md ──▶ follow Process ──▶ run Verification
```

## Key concepts

| Concept | What it means |
|---|---|
| **Frontmatter** | `name`, `description` (this is the *trigger*, so write it the way users phrase requests), `license`, `compatibility`, `allowed-tools`. |
| **Progressive disclosure** | Keep only descriptions in context. Load the body on activation, and load `references/` only when a step needs them. |
| **Process** | Numbered, concrete steps ("read every file under review, do not review from memory"). |
| **Rationalizations** | Excuses the agent might use to skip steps, listed with rebuttals ("it's a small change, no need to test"). |
| **Verification** | A checklist that must pass before the skill counts as done. |
| **`allowed-tools`** | Least privilege: a skill declares the tools it may use (for example `Bash(grep:*) Read`). |
| **Skill security** | A skill is *executable behavior*. Audit third-party skills like dependencies (`skill-security-audit`). |

## Skill library (`skills/`)

13 skills covering the engineering lifecycle:

| # | Skill | Triggers on… |
|---|---|---|
| 1 | `code-review` | review code, audit PR, check for vulnerabilities |
| 2 | `api-design` | design API, review endpoints, define schema |
| 3 | `git-commit` | commit changes, prepare commit, push code |
| 4 | `database-migration` | update schema, run alembic, add column |
| 5 | `dependency-audit` | check dependencies, find vulnerable libraries |
| 6 | `incident-response` | service is down, errors spiking, latency degraded |
| 7 | `test-generation` | write tests, add coverage |
| 8 | `deploy-checklist` | deploy, ship, release, promote to staging |
| 9 | `documentation-writer` | document module, write README, API docs |
| 10 | `data-pipeline` | build ETL, transform data, batch job |
| 11 | `security-hardening` | harden API, prepare for security review |
| 12 | `performance-profiling` | find bottleneck, reduce latency, memory leak |
| 13 | `skill-security-audit` | audit the skill library, check skills for prompt injection |

Each follows the same layout: `## Overview` → `## Process` → `## Rationalizations` → `## Verification`. Optional `scripts/` and `references/` folders can sit next to `SKILL.md`.

A second, simpler style (frontmatter `description` + free-form body) lives in [`../shared/declarative_skills/`](../shared/declarative_skills/) (30 role skills such as `threat_modeler`, `sysadmin_troubleshooter`, `regex_builder`). Ch. 3's `skill_auto_discovery.py` loads those.

## Loading skills

A minimal loader: parse frontmatter for the catalog, then load the body on demand.

```python
import yaml
from pathlib import Path

def discover_skills(root="ch09_agent_skills/skills"):
    catalog = {}
    for f in Path(root).glob("*/SKILL.md"):
        text = f.read_text()
        _, fm, body = text.split("---", 2)
        meta = yaml.safe_load(fm)
        catalog[meta["name"]] = {"description": meta["description"], "path": f}
    return catalog

def catalog_prompt(catalog):
    return "Available skills:\n" + "\n".join(f"- {n}: {s['description']}" for n, s in catalog.items())

def activate_skill(catalog, name):
    return catalog[name]["path"].read_text()   # full procedure, loaded only now
```

See also `../ch03_tools_and_skills/online/skill_auto_discovery.py`.

## What needs to be done

- [ ] Read `skills/code-review/SKILL.md` end to end. It is the reference example.
- [ ] Put the catalog into your agent's system prompt and check that requests trigger the right skill.
- [ ] Write a new skill for a task your team repeats (for example "on-call handoff"). Include Rationalizations and Verification.
- [ ] Tighten a skill's `allowed-tools` and confirm the agent still completes the task.
- [ ] Run the `skill-security-audit` procedure against a skill you downloaded from elsewhere.

## Production notes

- The `description` field decides whether a skill fires. Test it with real phrasing.
- Treat skills from outside sources like untrusted code: review them, pin them, and restrict their tools.
- Ch. 21's `/learn` has the agent *write* skills. Those go to a staging folder and need human review before they reach `skills/`.

## Takeaways

1. Skills are expertise stored as Markdown: easy to diff, review, and share.
2. Progressive disclosure keeps the base prompt small while the agent's skill set grows.
3. The Verification checklist turns "done?" into a concrete check.

**Prev:** [Ch. 8](../ch08_mcp_a2a/README.md) · **Next:** [Chapter 10 — Claude Code and Antigravity](../ch10_claude_code_antigravity/README.md)
