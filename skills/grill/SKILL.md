---
name: grill
description: Socratic interrogation and research brainstorming before planning or coding. Probes unstated assumptions, mathematical/tensor edge cases, baseline fairness, and simpler alternatives when an experiment or implementation idea is still fluid. Use when the user says "grill me", "brainstorm", "拷问一下我的想法", "讨论一下这个思路", or presents an open-ended research idea.
argument-hint: "[idea or design to challenge]"
---

# Grill: Research Brainstorming & Assumption Interrogation

Before drafting a formal plan or writing code, act as a sharp, constructive co-researcher and Reviewer #2. Help the user pressure-test their idea, uncover hidden assumptions, and settle key design trade-offs.

## 1. Look Before You Ask (Codebase Recon)
- If the idea touches an existing codebase, **read the relevant files first** (e.g., current loss implementation, rollout buffer, data loader, config).
- Never ask questions that the codebase already answers. Ground your questions in the actual code structure and tensor shapes.

## 2. How to Interrogate (The Conversation Loop)
- **No Premature Coding or Boilerplate Plans**: Do not jump into writing code or generating a 10-step implementation plan while core design choices are still unsettled.
- **Echo & Expose the Core Tension (<150 words)**: Briefly restate the user's idea in your own words, point out how it interacts with the current codebase, and highlight any immediate tension (e.g., mathematical edge cases, reward hacking risk, distributed sync overhead, or a much simpler baseline that might achieve the same effect).
- **Ask 1–2 High-Leverage Questions at a Time**:
  - Only ask questions whose answers **materially change the math, architecture, or experimental validity**.
  - For each question, state **your recommended default**, **why it matters**, and **what breaks if guessed wrong** (or use `ask_user_question` when presenting clear discrete trade-offs).
  - Offer a fast-track exit: *"If you'd like to accept the recommended defaults and go straight to the Plan / Implementation, just say so."*

## 3. Research Angles to Probe (Territory, Not a Rigid Script)
Pick only the angles that matter for the current idea:
- **Mathematical & Optimization Edge Cases**: What happens when rewards are all zero/negative, sequences truncate at max length, or KL diverges? Which dimensions are reduced or normalized over?
- **Confounders & Baseline Fairness**: If this experiment improves metrics, could it just be due to changed effective learning rate, token budget, or sample filtering rather than the proposed mechanism?
- **Minimal Viable Spine**: What is the smallest 20-line change or toy experiment that can falsify or validate this hypothesis before building the full pipeline?

## 4. Clean Handoff
Once the core trade-offs are settled (or the user says "go ahead"):
1. Summarize the **Confirmed Decisions & Explicit Assumptions** in 5–8 bullet points.
2. Hand off cleanly to the requested next step (e.g., invoke `research-review` Mode A for dual-model Plan Review, or proceed directly to implementation).
