---
name: research-review
description: On-demand dual-model review via native `spawn_subagent` (defaulting to mcli models `deepseek-v4.1-flash` and `glm-5.3`, or user-specified models) for standalone or combined research actions (Plan, Coding, Test, or Coding+Test together). Use when designing an experiment plan, reviewing post-training/RL/masking code changes, or auditing test validity.
argument-hint: "[plan|coding|test|coding+test] [models...]"
---

# Research Review (Modular & Combined Review Gates)

`Plan`, `Coding`, and `Test` can run **standalone** or **combined**:
- If you only planned, run **Mode A (Plan Review)**.
- If you wrote code and immediately wrote/ran tests (the natural flow), run **Mode B+C Combined (Code & Test Joint Review)** in a single parallel review round—do not waste time spawning two separate review rounds back-to-back.
- If you only wrote code without tests, or only wrote/ran tests for existing code, run **Mode B** or **Mode C** standalone.

## 1. Reviewer Subagent Dispatch (`spawn_subagent`)

- **Default Reviewers (`channel=mcli`)**: Call `spawn_subagent` concurrently to launch two parallel reviewers and wait for both via `get_command_or_subagent_output`:
  - Subagent 1: `model: "deepseek-v4.1-flash"` (mcli channel)
  - Subagent 2: `model: "glm-5.3"` (mcli channel)
- **User Override**: If the user specifies other models (e.g., `kimi-k3.1-pro`, `MiniMax-M3.1-Flash`, `deepseek-v4-pro`, or `friday/*` / `catpaw-ide/*` slugs), pass the user-specified `model` slugs to `spawn_subagent`.
- **Incremental Closure (`resume_from`)**:
  - **Discovery (Round 1)**: Spawn fresh subagents with only the minimal artifact (Plan, `git diff`, test code + execution stdout/stderr) and key file paths—never pass your conversation history.
  - **Closure (Round 2+, if fixes or extra tests were needed)**: Call `spawn_subagent` with `resume_from: "<subagent_id>"` for each reviewer, passing only the active findings and the fix/test delta so they verify the update without re-reading unchanged files.

## 2. Core Role Boundaries
1. **Main Agent Runs Validation First**: Running a quick syntax/smoke test before dispatching reviewers is encouraged so reviewers don't waste effort on broken syntax. Pass your exact test command and output as `Main-agent validation evidence`.
2. **Strict Read-Only Reviewers**: Instruct reviewer subagents to only read/grep files, report severity-ordered findings with `file:line` evidence, and emit `- Validation request: <exact check>` when an untested edge case needs runtime proof. Reviewers **never** edit files or run commands.
3. **Main-Agent Adjudication**: Wait for all launched reviewers to finish before editing. Reproduce candidate bugs independently, run valid `Validation requests`, and adjudicate (`accepted` / `rejected` with concrete code or math evidence).

---

## Mode A: Plan Review (Standalone)
1. **Prepare**: State the objective, files to touch, tensor shapes / data flow, and **explicit assumptions** (any unstated choice affecting training semantics, reward scale, or baseline fairness).
2. **Prompt Reviewers**:
   > "Read-only review of this research plan. Do not edit files or run commands. Challenge unstated assumptions, check for data/label leakage, baseline unfairness, and distributed/rollout bottlenecks. State `Review coverage: FULL|PARTIAL - <inspected files>` and end with `Review verdict: APPROVED|FINDINGS`. [Insert Plan & Key File Paths]"

## Mode B: Coding Review (Standalone — when no test was written yet)
1. **Prepare**: Collect the requirements/plan summary and `git diff`.
2. **Prompt Reviewers**: Instruct them to inspect changed files and one-hop callers read-only for **silent research bugs**:
   - **Masking & Alignment**: SFT/RL prompt vs. response token mask off-by-one errors, padding side mismatches, or EOS/truncation handling.
   - **Tensor Shapes & Broadcasting**: Unintended broadcasting (e.g., `(B, 1)` vs `(B,)`), reduction dimension errors (`mean`/`sum` over sequence vs. batch).
   - **Gradient & State Hygiene**: Misplaced `detach()`/`no_grad()`, KL direction, advantage normalization scope, or unreset agent environment state across rollouts.
   - Require `Review coverage` and `Review verdict: APPROVED|FINDINGS`.

## Mode C: Test Review (Standalone — when adding/auditing tests for existing code)
1. **Prepare**: Collect the test file path, target module path, and actual test execution stdout/stderr.
2. **Prompt Reviewers**:
   > "Read-only audit of this test and its execution output. Do not edit files or run commands. Did it genuinely verify the core mathematical/tensor invariants and edge cases, or did it mock away real logic / weaken assertions to pass? Emit `- Validation request: ...` if an edge case is untested. State `Review coverage` and end with `Review verdict: APPROVED|FINDINGS`."

## Mode B+C: Joint Coding & Test Review (When Main Agent coded and self-tested in one go)
1. **Prepare**: Package the requirements/plan summary, implementation + test `git diff`, and `Main-agent validation evidence` (exact test command run and its stdout/stderr).
2. **Prompt Reviewers**:
   > "Read-only joint review of the implementation diff and the Main Agent's self-test. Do not edit files or run commands.
   > 1. **Code Check**: Hunt for silent research bugs in changed files and one-hop callers (token mask off-by-one, padding/EOS alignment, unintended tensor broadcasting `(B,1)` vs `(B,)`, wrong reduction dim, misplaced `detach()`/`no_grad()`, KL/advantage math, unreset rollout state).
   > 2. **Self-Test Audit**: Check whether the Main Agent's test genuinely verifies these invariants and edge cases, or if it suffers from confirmation bias (only testing happy paths, mocking away core logic, or weak assertions). Emit `- Validation request: <exact check>` for any missing edge-case test.
   > State `Review coverage: FULL|PARTIAL - <inspected files>` and end with `Review verdict: APPROVED|FINDINGS`."
3. **Adjudicate & Closure**: Fix accepted code bugs and fulfill valid `Validation requests`; if non-trivial changes were made, use `resume_from` on both reviewers to verify the delta.

---

## 3. Lightweight Ledger (`docs/review-log.md`)

After each review gate completes, append a concise 5–10 line English entry to `docs/review-log.md` in the current project (create the file if absent). This preserves `subagent_id` handles across context compaction and maintains an experimental audit trail:

```markdown
## [YYYY-MM-DD] <Short Task Title> (<Mode A / B / C / B+C>)
- **Assumptions / Scope**: <1-line summary of key design assumptions and touched files>
- **Reviewers**: `deepseek-v4.1-flash` (`id: <subagent_id>`), `glm-5.3` (`id: <subagent_id>`)
- **Adjudication**:
  - `[ACCEPTED]` `<file:line>` — <concise bug/flaw and how it was fixed>
  - `[REJECTED]` `<file:line>` — <why the reviewer finding was a false positive, with math/code evidence>
- **Verdict & Proof**: `APPROVED` (<1-line test/smoke verification result>)
```
