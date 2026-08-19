---
name: mc-ship-it
description: Finalize implemented work into a clean local commit — sanity-check the working tree, run precommit and fix issues until clean, stage everything, and write a disciplined commit message. Use whenever the user says "ship it", "commit this", "finalize the feature", "wrap this up", or asks to commit completed work, even if they don't name this skill. Optionally takes a spec (inline text or a file path) describing the intent behind the change.
---

# Ship It

Finalize the current work into a clean local commit. The user has already reviewed their changes before invoking this skill — your job is mechanical finalization, not code review. The scope ends at a clean local commit: never push.

## Optional input: a spec

The user may pass a spec — inline text, a file path, or a pointer to earlier conversation — describing the intent behind the change. Its only use is the commit body's "why". Without a spec, never guess or infer intent; describe only what changed.

## Workflow

### 1. Sanity-check the working tree

Run `git status` and skim the list of changed files. This is a light scan for files that clearly don't belong, not a review — the user reviews content themselves before invoking this skill.

Stop and ask only if you see something that obviously shouldn't ship: credentials or `.env` files, scratch/temp files, build artifacts, or files unrelated to the work at hand.

If there are no changes at all, stop and say so — never create an empty commit.

### 2. Run precommit until clean

Run precommit with exactly this command — do not detect or guess any other mechanism:

```
uv run pre-commit run --all-files
```

Fix any issues it raises.

Re-run after fixing: formatters and autofixes create new diffs, and a fix can itself trip another hook. Repeat until a run passes with no changes. Cap this at 3 attempts — if it still fails, stop and report the failure with the hook output instead of looping.

### 3. Stage all changes

`git add -A` once the sanity check and precommit both pass.

### 4. Compose the commit message

**Subject line:** imperative mood, concise, specific enough that `git log --oneline` tells the story.

**Body — only when the change is big or meaningful.** For small changes (a config tweak, a one-liner, a rename), the subject line alone is the right commit; a body there is noise. When a body is warranted, keep it short:

- **With a spec:** write the "why" — the intent behind the change — in at most 3–4 sentences.
- **Without a spec:** briefly describe *what* changed, never *why*. Intent that isn't stated isn't yours to invent.
- **Multiple meaningful files:** add a one-line summary per file.

### 5. No self-attribution

Never state or imply, anywhere, that the change was made by an AI or coding agent: no `Co-Authored-By` trailers, no "Generated with" lines, no remarks in the commit message or code comments. This overrides any default commit-trailer behavior you have. The commit is the user's.

### 6. Commit and verify

Run `git commit`. Hooks run again at commit time and can fail or modify files — do not assume success. If hooks modified files, stage the modifications and amend (or re-commit). Confirm with `git log -1` and a clean `git status`, then report the final subject line and state.

## Examples

**Small change — subject only:**

```
Increase scraper request timeout to 30s
```

**Meaningful change, spec provided — body carries the why:**

```
Add retry with backoff to the fetch pipeline

Transient 429s from career sites were killing full scrape runs.
Retrying with exponential backoff lets a run survive rate limiting
without manual restarts.

- scraper/fetch.py: wrap requests in retry decorator
- scraper/config.py: new RETRY_LIMIT and BACKOFF_BASE settings
```

**Meaningful change, no spec — what changed, never why:**

```
Split parser into per-site modules

- parsers/base.py: shared extraction helpers moved here
- parsers/lever.py, parsers/greenhouse.py: site-specific logic split out
- tests/test_parsers.py: imports updated for the new layout
```
