---
name: codex-review
description: Run a code review using the OpenAI Codex CLI (`codex exec`) headlessly from within Claude Code, then respond to each Codex finding with a short specific commentary. Use this whenever the user invokes `/codex-review <prompt>`, wants to "run codex", "use codex to review", or wants Codex to look at their code. The prompt passed by the user is forwarded to Codex as-is.
disable-model-invocation: true
---

# Codex Code Review

Run `codex exec` non-interactively with the user's prompt, display its output, then add a specific one-liner per finding as a second opinion.

## Available models (from `~/.codex/models_cache.json`)

| Slug | Name |
|------|------|
| `gpt-5.5` | GPT-5.5 — strongest, default in this project |
| `gpt-5.4` | GPT-5.4 |
| `gpt-5.4-mini` | GPT-5.4-Mini — faster, lighter |
| `gpt-5.3-codex` | GPT-5.3-Codex |

Reasoning effort levels (all models): `low` | `medium` (default) | `high` | `xhigh`

---

## Step 1: Ask — one AskUserQuestion call, 2 questions

**Question 1** — "Which Codex model?" (header: `"Model"`)
- `gpt-5.5` — GPT-5.5, strongest (Recommended)
- `gpt-5.4` — GPT-5.4
- `gpt-5.4-mini` — GPT-5.4-Mini, faster
- `gpt-5.3-codex` — GPT-5.3-Codex

**Question 2** — "Thinking effort?" (header: `"Effort"`)
- `medium` — balanced (Recommended)
- `low` — fast
- `high` — deeper reasoning
- `xhigh` — maximum depth, slowest

---

## Step 2: Run the command

Use the user's original prompt (the args passed to `/codex-review`) verbatim. Assign it to a variable to avoid quoting issues:

```bash
PROMPT="<user's original prompt>"
codex exec --sandbox read-only --ephemeral \
  -c model="<model>" \
  -c model_reasoning_effort="<effort>" \
  "$PROMPT"
```

If codex is not found in PATH: `/home/mat/.nvm/versions/node/v24.13.0/bin/codex`

---

## Step 3: Show output

Display the Codex output under **Codex review:**

If the output is empty or the command exited non-zero, report the error and stop. Don't fabricate a review.

---

## Step 4: Claude's take

Read the Codex output carefully. For **each distinct finding** Codex raised, write one item — **1–2 sentences max**:
- Respond to that specific point: do you agree, and why? Add any nuance relevant to this codebase.
- State severity: critical / minor / style
- Suggest a quick fix if Codex didn't provide one

Ground every item in what Codex actually said — don't add generic advice.

**Claude's take:**
1. …
2. …
