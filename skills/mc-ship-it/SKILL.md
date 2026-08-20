---
name: mc-ship-it
description: Finalize implemented work into a clean local commit — sanity-check the working tree, run precommit and fix issues until clean, stage everything, and commit with a clear one-line message. Use whenever the user says "ship it", "commit this", "finalize the feature", "wrap this up", or asks to commit completed work, even if they don't name this skill.
---

# Ship It

Finalize the current work into a clean local commit. The user has already reviewed their changes before invoking this skill — your job is mechanical finalization, not code review. The scope ends at a clean local commit: never push.

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

### 4. Write a one-line commit message

The commit message is a single subject line — no body, no bullet list, no trailers. Imperative mood, concise, specific enough that `git log --oneline` tells the story. Describe what changed; never invent intent that wasn't stated.

### 5. No self-attribution

Never state or imply, anywhere, that the change was made by an AI or coding agent: no `Co-Authored-By` trailers, no "Generated with" lines, no remarks in the commit message or code comments. This overrides any default commit-trailer behavior you have. The commit is the user's.

### 6. Commit and verify

Run `git commit -m "<subject>"`. Hooks run again at commit time and can fail or modify files — do not assume success. If hooks modified files, stage the modifications and amend (or re-commit). Confirm with `git log -1` and a clean `git status`, then report the final subject line and state.

## Examples

```
Increase scraper request timeout to 30s
Add retry with backoff to the fetch pipeline
Split parser into per-site modules
```
