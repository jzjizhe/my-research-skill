---
name: evolve
description: Route lessons learned after debugging or review rework into either Memory (codebase/framework/experiment pitfalls) or Skills (workflow/review blind spots). Use after completing a research-review task that required rework or uncovered a subtle bug.
---

# Evolve: Split Memory & Skill Maintenance

When a review gate or debugging session resolves a non-obvious issue, classify the lesson and update the right layer:

## 1. Route the Lesson

| Trigger Signal | Target Layer | Action |
| :--- | :--- | :--- |
| **Codebase, Framework, or Environment Pitfall**<br>(`verl`/`TRL`/`vLLM` quirks, FSDP/DeepSpeed behavior, tokenizer `chat_template` / padding traps, cluster paths, dataset schema quirks) | **Memory** (`AGENTS.md` or workspace memory) | Record a concise factual invariant so future sessions in this repo never repeat the mistake. |
| **Reviewer False Negative (Missed Bug)**<br>Reviewers approved, or missed a subtle bug that later surfaced in testing/training | **Skill** (`research-review` `SKILL.md`) | Add the specific failure pattern to the Mode A/B/C review checklist. |
| **Systematic False Positive or Context Gap**<br>Main Agent repeatedly had to `reject` a reviewer's finding because reviewers lacked a specific piece of context | **Skill** (`research-review` `SKILL.md`) | Update the mode preparation instructions so that context is passed upfront. |

## 2. Anti-Bloat Constraints (Strict)
- **One-to-Two Sentences Max**: Write rules as sharp, actionable invariants (e.g., *"In verl GRPO, always mask pad tokens before computing per-token KL; default mask includes trailing pad"*). Never write narrative debugging stories.
- **Merge Before Append**: Read the target file first. If a related bullet exists, tighten or merge into it instead of adding a new bullet.
- **Hard Cap**: Keep `research-review` `SKILL.md` under 85 lines and project pitfalls in `AGENTS.md` under 60 lines. Prune outdated items when adding new ones.
