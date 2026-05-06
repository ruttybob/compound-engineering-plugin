# Forked Brainstorm Output Template

Loaded by `ce-replan-beta`'s Phase 4a. The skill writes the forked brainstorm to `docs/brainstorms/` using the filename and frontmatter conventions documented here. The original brainstorm is **never** edited or deleted.

## Filename pattern

```
docs/brainstorms/YYYY-MM-DD-<topic>-rebrainstorm-requirements.md
```

- `YYYY-MM-DD` — today's date.
- `<topic>` — kebab-cased; matches the original brainstorm's topic so the relationship is visible at a glance. When the original was, e.g., `2026-05-04-cora-v2-briefed-requirements.md`, the fork is `2026-05-06-cora-v2-briefed-rebrainstorm-requirements.md`.
- `-rebrainstorm-` infix disambiguates when multiple revolutions happen on the same day (the date alone is not enough).

## Frontmatter contract

```
---
date: YYYY-MM-DD
topic: <kebab-case-topic>
revision: <integer, supersedes-rev + 1>
supersedes: <repo-relative path to the original brainstorm>
---
```

- `revision:` increments by 1 from the source's `revision:`. If the original lacks the field, treat it as `revision: 1` and write `revision: 2` here.
- `supersedes:` names the immediately-prior brainstorm (the one this fork re-brainstormed against), not the chain root. Walking the chain of `supersedes:` links provides full history.

## Section order

The forked brainstorm follows the standard `ce-brainstorm` requirements-doc structure with one new section unique to forks (`## Discarded Requirements`).

```markdown
---
date: YYYY-MM-DD
topic: <kebab-case-topic>
revision: <integer>
supersedes: <repo-relative path>
---

# <Topic Title>

## Summary

[1-3 line forward-looking gloss of the new requirements shape. Not a diff against the prior revision — a self-contained summary of what the requirements say now.]

---

## Re-Grounded Problem Frame

[Backward-looking, situational. Re-derived from PR discussion language and learnings, NOT inherited from the original brainstorm's framing. Names the moment of pain, what the user thought before, and what changed in their understanding.]

---

## Actors

[If the original had Actors, carry them forward. Each carries an `[unchanged from rev N]` or `[revised from rev N]` marker. New actors get the next-unused A-ID.]

- A1. **<Name or role>** [unchanged from rev 1]: <what they do>
- A2. **<Name or role>** [revised from rev 1]: <new wording>

---

## Key Flows

[Same disposition discipline as Actors. Discarded flows move to a `## Discarded Flows` section if any exist.]

- F1. **<Flow name>** [unchanged from rev 1]
  - **Trigger:** ...
  - **Steps:** ...
  - **Outcome:** ...
  - **Covered by:** R1, R2

---

## Requirements

[Every requirement carries an `[unchanged from rev N]` or `[revised from rev N]` marker inline. Discarded requirements do **not** appear here — they move to `## Discarded Requirements` below. R-IDs are stable: gaps are preserved.]

**<Group header>**

- R1. <wording> [unchanged from rev 1]
- R2. <wording> [unchanged from rev 1]
- R4. <new wording> [revised from rev 1]. Was: <original wording>. Why revised: <one-line learning that drove the change>.
  
*(Note R3 absent — moved to Discarded Requirements.)*

**<Another group header>**

- R7. <wording> [new in rev 2]

---

## Discarded Requirements

[Each entry preserves the original R-ID (so the gap in the active list above is documented), the original wording, and a one-line reason for discard. The `[discarded from rev N]` marker indicates which revision the requirement was last active in.]

- R3. <original wording> [discarded from rev 1]. Why discarded: <one-line reason>.
- R5. <original wording> [discarded from rev 1]. Why discarded: <one-line reason>.

---

## Acceptance Examples

[Acceptance examples follow the same disposition discipline. Carried-forward AEs keep their AE-ID with `[unchanged from rev N]` or `[revised from rev N]` markers; discarded AEs move to a `## Discarded Acceptance Examples` section if any exist.]

- AE1. **Covers R1, R2.** Given ..., when ..., outcome. [unchanged from rev 1]

---

## Success Criteria

[Forward-looking, fresh prose. Not necessarily annotated — these are statements about the new shape, not carried-forward items.]

- ...

---

## Scope Boundaries

[Apply the disposition rules where boundaries carry forward. New exclusions tied to discarded requirements are noted as `[new in rev 2]`.]

- <Carried-forward boundary> [unchanged from rev 1]
- <New boundary> [new in rev 2]

### Deferred to Follow-Up Work

- <Plan-local follow-up>

---

## Key Decisions

[Forward-looking — the decisions that shape the new revision. Not annotated with disposition; the rationale lines say enough.]

- <Decision>: <Rationale, with reference to the learning when relevant>

---

## Dependencies / Assumptions

- ...

---

## Outstanding Questions

### Resolved Before Planning

[Questions that the new learnings or the re-brainstorm phase resolved. Note the resolution.]

- ...

### Deferred to Planning

[Questions still open; the re-plan phase or its downstream work will address them.]

- ...
```

## Discipline checks

Before writing the forked brainstorm to disk, verify:

- Every original R-ID is accounted for: present in the active Requirements list (with `[unchanged]` / `[revised]` marker) **or** in `## Discarded Requirements` (with `[discarded]` marker). No silent drops.
- New R-IDs continue from the highest used original ID + 1; no reuse of discarded IDs.
- Frontmatter `supersedes:` is the immediately-prior brainstorm (path verified to exist on disk). `revision:` is exactly one higher than the source's.
- Re-grounded problem frame uses user discussion language, not paraphrase of the original brainstorm.
- Filename matches `docs/brainstorms/YYYY-MM-DD-<topic>-rebrainstorm-requirements.md`.
- Topic in the filename matches the original (or the latest fork in the chain). Walking the chain via `supersedes:` should reach the chain root cleanly.
- All paths in the doc are repo-relative, never absolute.

## Special cases

- **Original had no R-IDs** (legacy fallback per `references/rebrainstorm-workflow.md` § Legacy fallback). Surface the derived IDs in the synthesis before writing the fork; once confirmed, treat them as if they had been there all along.
- **No new requirements.** A forked brainstorm with only `[unchanged]` and `[revised]` markers — and possibly `[discarded]` items — is valid. The `## Discarded Requirements` section may be present without new R-IDs above. Surface this in the synthesis so the user knows the fork is purely a refinement.
- **Many discards.** A forked brainstorm where most original R-IDs are discarded is a signal that the topic itself may have shifted. Surface in the synthesis; consider whether the user wanted to start a fresh brainstorm rather than fork. Default behavior: still write the fork; don't second-guess the discipline.

## Non-goals for this template

- Producing a plan. The plan is downstream; this template only covers the brainstorm fork.
- Capturing implementation details (file paths, exact code shapes). Those belong in the plan; the brainstorm stays at the requirements layer.
- Preserving an unstable diff against the original. The fork is the new source of truth at this revision; the original is preserved separately by reference, not by interleaved annotation.
