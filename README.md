# My Research Skills (`magent`)

A lightweight, high-signal set of rules and skills tailored for **LLM & Agent Post-Training Research** in `magent`.

## Architecture

```text
.
├── rules/
│   └── research-routing.md        # Global complexity & clarity router (~15 lines)
└── skills/
    ├── grill/
    │   └── SKILL.md               # Socratic brainstorming & assumption interrogation before planning
    ├── research-review/
    │   └── SKILL.md               # Modular & joint dual-model review (Plan / Coding / Test / Coding+Test)
    └── evolve/
        └── SKILL.md               # Anti-bloat lesson router (Memory for repo pitfalls vs. Skill for workflow gaps)
```

## Core Features

1. **`grill` (`/grill`)**: Interrogate open-ended research ideas before writing plans or code. Grounds questions in the actual codebase, exposes mathematical/tensor edge cases and baseline confounders, and recommends concrete defaults.
2. **`research-review` (`/research-review`)**: Modular or combined review gates (`Mode A: Plan`, `Mode B: Coding`, `Mode C: Test`, `Mode B+C: Joint Coding & Test`) using native `spawn_subagent` with parallel read-only reviewers (defaulting to `mcli` models `deepseek-v4.1-flash` and `glm-5.3`, or user-specified models) and incremental `resume_from` closure.
3. **`evolve` (`/evolve`)**: Automatically separates codebase/framework pitfalls (routed to `AGENTS.md` / Memory) from workflow/reviewer blind spots (routed to `SKILL.md`), with strict line caps to prevent prompt bloat.

## Installation

Copy the rules and skills into your global `~/.magent/` directory:

```bash
mkdir -p ~/.magent/rules ~/.magent/skills
cp rules/research-routing.md ~/.magent/rules/
cp -R skills/* ~/.magent/skills/
```
