# Research Workflow & Complexity Routing

Assess task clarity and complexity before acting. `grill`, `plan`, `coding`, and `test` are modular actions—invoke only what fits the current step:

- **Idea is Open-Ended or Under-Specified**: Invoke `grill` to pressure-test assumptions and settle key trade-offs with the user before planning or coding.
- **Direct Execution (Skip dual-model review)**: Single-file utility tweaks, plotting/visualization scripts, hyperparameter config edits, logging/formatting changes, or documentation. Execute directly.
- **Trigger `research-review` Skill**: Automatically invoke `research-review` when the work involves:
  1. Post-training core logic (loss functions, reward/advantage computation, KL penalties, rollout sampling, policy/value updates).
  2. Data pipeline, chat template formatting, token masking, or evaluation/benchmark scripts where bugs silently invalidate experimental claims.
  3. Multi-file architectural changes or new experimental modules.
  *(Note: If you write code and immediately write/run a self-test in the same flow, use `research-review` **Mode B+C Joint Review** to review both the implementation diff and your self-test in one parallel pass.)*

When `research-review` catches a non-trivial bug or requires rework, run the `evolve` skill at the end of the task.
