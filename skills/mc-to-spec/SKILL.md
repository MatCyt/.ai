---
name: mc-to-spec
description: >-
  Convert the current planning conversation into a spec plus individual issue files under
  .scratch/<feature>/, executable either step-by-step with review gates or independently
  in parallel. Extract every decision from the conversation, verify references against the repository, resolve open decisions with the user BEFORE writing. A fresh session with zero conversation memory must be able to
  execute any ready issue from the spec + issue file alone.
disable-model-invocation: true
user-invocable: true
argument-hint: "[optional: feature slug; default: derived from the conversation]"
---

# mc-to-spec — conversation → spec + issues

A planning conversation produces decisions and context that evaporate when the session
ends. This skill captures both into artifacts that fully replace the conversation.
Success test: **a fresh session with zero conversation memory can execute any ready
issue correctly from `spec.md` + its issue file alone.**

This skill produces the plan only — do not implement anything.

## Process

### 1. Harvest the conversation

Re-read the whole conversation and extract, exhaustively:

- Every decision — explicit ("use X") and implicit (approved by moving on) — with its
  one-line why.
- Rejected alternatives where the rejection wasn't obvious, so a future session doesn't
  reintroduce them.
- Constraints and invariants: behavior that must not change, things the user said to
  leave alone.
- Every file, pattern, or document referenced as an example or source of truth.
- Anything discussed but left unresolved.

### 2. Verify against the repository

The conversation may be wrong about the code. Verify every referenced path and symbol
exists (fix them in the artifacts — a wrong path strands the executor). Find the
verification command: `make verify` if a Makefile defines it, else
`uv run ruff check && uv run pytest && uv run mypy && uv run deptry`. Read convention
docs (AGENTS.md etc.) if not already in context.

### 3. Close the gaps — before writing

List every unresolved decision and ambiguity. Ask the user to resolve them now
(batched). Only locked decisions go into the artifacts. If the user defers one, record
it in the spec under **Open questions**, naming the issues it blocks.

### 4. Write the artifacts

Write to `.scratch/specs/<feature-slug>/`:
spec.md
issues/01-<slug>.md # numbered in dependency order, blockers first
issues/02-<slug>.md

**spec.md** — everything shared across issues; issues point here instead of repeating it:

    # <Feature> — spec

    <One paragraph: what is being built, where, why.>

    ## Decisions (locked)
    <Settled statements, one-line rationale. Include non-obvious rejections.
    Phrase as locked ("No new table"), never as open comparisons.>

    ## Invariants
    <Rules binding every issue: behavior that must not change, conventions
    that override defaults.>

    ## Reference map
    <Pointers, not pasted code: "dual-write pattern: services/orders-api/writers.py".
    Every path verified.>

    ## Out of scope
    <For the feature as a whole.>

    ## Issues
    <One line each: number, title, blocked-by — the only overview of the whole plan.>

    ## Open questions
    <Only if deferred in step 3; name the issues each blocks.>

**issues/NN-slug.md** — one issue per file:

    # NN — <title>

    Read `../spec.md` first. Decisions, invariants, and references there bind this issue.

    **Blocked by:** <numbers, or "none — ready">
    **Status:** ready

    ## What to build
    <The behavior this issue delivers, end to end.>

    ## Files
    <Created / modified. Verified paths for early issues; pattern names + reference-map
    pointers for issues deep in the chain, where paths may rot before execution.>

    ## Out of scope
    <What this issue must NOT touch, especially tempting adjacent work.>

    ## Definition of Done
    <Objectively checkable, including the verification command.>

    ## Review focus [mechanical|new-logic]
    <What the reviewer should scrutinize, or what evidence lets them skim.>

## Issue design rules

- **Vertical slices**: each issue is one behavior plus its tests, leaving the repo
  green. Never "all modules, then all tests".
- **Order by dependency, size by review load** — target a diff reviewable in 10–15
  minutes. An issue must also fit a single fresh context window.
- **Isolate new business logic** in small issues; **batch mechanical work** into larger
  ones — the review-focus line tells the reviewer why skimming is safe.
- **Blocking edges carry the state.** An issue is workable when every issue in its
  "Blocked by" line is done — that's the whole scheduling rule, for you or for
  parallel agents. Only add an edge when the dependency is real (the issue needs
  code another issue creates); an unnecessary edge forbids parallel work for no
  reason.

## Writing rules

- Write for an executor with zero conversation memory. No sentence may point at the
  conversation ("as discussed").
- Never copy secret values.
- Keep it short: every sentence locks a decision, states an invariant, points at a
  reference, or defines work.

## 5. Self-check, then hand off

For each issue, simulate a fresh reader with only spec.md + that file: executable
without guessing? If not, move the missing knowledge into the spec or the issue.
Then report: the directory path, the issue list one line each, any open questions, and
how to start ("point an executor at `.scratch/<feature>/issues/01-….md`").