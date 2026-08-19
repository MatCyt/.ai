---
name: mc-code-review
description: Review the changes since a fixed point (commit, branch, tag, or merge-base) along two axes — Standards (does the code follow this repo's documented coding standards?) and Spec (does the code match what the originating issue/spec asked for?). Runs both reviews in parallel sub-agents and reports them side by side. Use when the user wants to review a branch, a PR, work-in-progress changes, or asks to "review since X".
user-invocable: true
argument-hint: "[optional: path to the spec file under .scratch/]"
---
 
Two-axis review of the diff between `HEAD` and a fixed point the user supplies:
 
- **Standards** — does the code conform to this repo's documented coding standards?
- **Spec** — does the code faithfully implement the originating issue / spec?
Both axes run as **parallel sub-agents** so they don't pollute each other's context; this skill aggregates their findings. The axes stay separate because a change can pass one and fail the other (right code, wrong thing — or right thing, wrong code); merging them lets one mask the other.
 
## Process
 
### 1. Pin the fixed point and gate
 
The fixed point is whatever the user said — SHA, branch, tag, `main`, `HEAD~5`. If unspecified, ask.
 
- Confirm it resolves: `git rev-parse <fixed-point>`.
- Capture once: `git diff <fixed-point>...HEAD` (three-dot) and `git log <fixed-point>..HEAD --oneline`. Empty diff → stop and report.

### 2. Identify the spec source
In order:
 
1. A path the user passed as an argument.
2. Issue references in commit messages.
3. Ask the user. If there is no spec, the Spec sub-agent skips and the report says "no spec available".

### 3. Identify the standards sources
 
Collect whichever exist: `AGENTS.md`, `ARCHITECTURE.md`. Note any testing exemptions in `ARCHITECTURE.md`; they override the testing defaults and must not be flagged.
 
On top of repo documents, the Standards axis carries this **smell baseline** (Fowler, _Refactoring_ ch.3), pre-tuned to this repo's conventions. Two rules bind it: a documented repo standard always wins, and each smell is a labelled judgement call ("possible Feature Envy"), never a hard violation — with one exception marked below.
 
- **Mysterious Name** — name doesn't reveal what it does or holds. → rename; if no honest name comes, the design's murky.
- **Duplicated Code** — same logic shape in more than one hunk or file. → extract the shared shape.
- **Feature Envy** — a function working mostly with another module's data. → move it next to the data.
- **Data Clumps** — the same few fields/params always travelling together. → bundle into a dataclass.
- **Primitive Obsession** — only when a primitive crosses a boundary or carries business meaning (repo policy endorses primitives for simple local values; boundary data is where the Pydantic rule applies). → give the concept its own small type.
- **Repeated Switches** — same `if`-cascade on the same type recurring across the change. → one shared dict-dispatch map (not class polymorphism — repo prefers small functions over class hierarchies).
- **Shotgun Surgery** — one logical change forcing scattered edits across many files. → gather into one module.
- **Divergent Change** — one module edited for several unrelated reasons. → split by responsibility.
- **Speculative Generality** — **hard finding, not a judgement call**: it restates the documented "avoid overengineering and unrequested abstractions" rule. → delete; inline until a real need shows.
- **Message Chains** — long `a.b().c().d()` navigation through your own object graph. → hide the walk behind one method. **Suppress for fluent/builder APIs** — dataframe chains (Polars, PySpark), query builders, and similar are idiomatic, not smells.
- **Middle Man** — a function/class that mostly delegates onward. → cut it. **Exempt entrypoints** — `main.py` is orchestration-only by design and should mostly delegate.
- **Refused Bequest** — an implementer ignoring most of what it inherits. → composition over inheritance. (Rare here; keep as tripwire.)
### 4. Spawn both sub-agents in parallel
 
**Standards sub-agent** — give it: the diff command and commit list; the standards-source contents from step 3; the tooling-enforced exclusion list; any ARCHITECTURE.md testing exemptions; the smell baseline pasted in full (it has no other access to it). Brief: "Report, per file/hunk where relevant, (a) every violation of a documented standard — cite file + rule; (b) any baseline smell — name it, quote the hunk. Documented-standard breaches and Speculative Generality are hard findings; other smells are judgement calls; a documented standard overrides the baseline. Skip anything on the tooling-enforced list and anything covered by a testing exemption. Under 400 words."
 
**Spec sub-agent** — give it: the diff command and commit list; the spec path or contents. Brief: "Report: (a) spec requirements missing or partial; (b) behaviour not asked for (scope creep); (c) requirements implemented but apparently wrong. Quote the spec line for each finding. Under 400 words."
 
If no spec, skip the Spec sub-agent and note it in the report.
 
### 5. Aggregate
 
Present both reports verbatim (or lightly cleaned) under `## Standards` and `## Spec`. Do **not** merge or rerank across axes. End with one line: findings per axis and the worst issue *within each axis* — no single winner across axes.