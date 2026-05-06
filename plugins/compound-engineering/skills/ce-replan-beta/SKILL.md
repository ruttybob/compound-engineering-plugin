---
name: ce-replan-beta
description: "[BETA] Replan from an existing PR after new learnings have emerged. Runs as a two-phase re-brainstorm + re-plan flow: phase one forks the original requirements doc with R-IDs carried forward stably; phase two derives a fresh full-redo plan that always starts from main. Original PR, plan, and brainstorm are preserved as superseded artifacts; no Git execution. Use when a PR's approach has been outgrown by review back-and-forth, code reading, or a new brainstorm. Invoke with /ce-replan-beta [PR number, or blank for current branch's PR]."
disable-model-invocation: true
argument-hint: "[PR number, or blank for current branch's PR]"
allowed-tools: Bash(bash *detect-pr.sh), Bash(bash *fetch-pr-context.sh), Bash(bash *find-original-plan.sh), Bash(bash *find-original-brainstorm.sh)
---

# Replan from an Existing PR (Beta)

`ce-brainstorm` defines **WHAT** to build. `ce-plan` defines **HOW** to build it. `ce-work` executes. `ce-replan-beta` is for the moment when an existing PR's approach has been outgrown by new learnings — review back-and-forth, code reading, a new brainstorm, or a "this could be much simpler" realization — and the original requirements and plan are grounded in assumptions that no longer hold.

The skill runs as two sequential phases. **Phase one (re-brainstorm)** re-questions the original requirements with the new learnings folded in; it forks the original `*-requirements.md` into a new dated revision with R-IDs carried forward stably. **Phase two (re-plan)** derives a fresh plan from the forked brainstorm — always a full redo from `main`, never a delta layered on the existing PR's tree.

The skill performs no Git operations. The original PR, original plan, and original brainstorm remain untouched on disk and on GitHub; the new artifacts supersede them by reference. The user starts a fresh branch from `main` themselves.

The shape closes a compounding loop. Each revolution sharpens the requirements layer (with stable R-IDs as anchors) and produces a new plan that traces back to those requirements. The chain of `supersedes:` links preserves the history of revolutions.

## Phase content

Phase content is filled in by the SKILL.md body (Phase 0 through Phase 5) and the references under `references/`. Phases load in this order:

- Phase 0 — Mode detection (`<input>` argument routing).
- Phase 1 — Discovery via the four scripts under `scripts/`.
- Phase 2a — Re-brainstorm (loads `references/rebrainstorm-workflow.md`).
- Phase 3a — Re-brainstorm synthesis checkpoint.
- Phase 4a — Write forked brainstorm (loads `references/rebrainstorm-template.md`).
- Phase 2b — Re-plan.
- Phase 3b — Re-plan synthesis checkpoint.
- Phase 4b — Write plan (loads `references/replan-template.md`).
- Phase 5 — Handoff with branch base committed.

The detailed phase instructions are written below in this file (filled in by U6 of `docs/plans/2026-05-06-002-replan-ce-replan-beta-beta-plan.md`).

> **Note:** This is a beta skill. The invocation contract, doc shapes, and discovery heuristics may change before promotion to a stable `ce-replan`. See `docs/solutions/skill-design/beta-skills-framework.md` for the framework and promotion path.
