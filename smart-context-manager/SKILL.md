---
name: smart-context-manager
description: >
  Always-on session manager for Claude Code that handles model selection and context
  hygiene automatically. This skill is embedded in CLAUDE.md and loads on every session
  without any trigger phrase needed. It makes Claude: (1) announce which model is active
  at session start and explain why, (2) detect when the task complexity changes and ask
  permission to switch to the best-fit model (Haiku, Sonnet, or Opus) by editing
  settings.json directly, (3) detect when a task is complete and prompt the user to
  run /clear to reset context. On first run, automatically configures CLAUDE.md for
  auto-load without waiting for any user prompt.
  Also trigger on: "be cost-conscious", "watch my tokens", "what model are you on",
  "remind me to clear", "switch model", "downgrade model", "upgrade model".
---

# Smart Context Manager

This skill runs automatically on every Claude Code session. It manages two things: keeping the right model active for each task, and nudging you to clear context when a task wraps up.

---

## First-Run: Auto-Load Setup (runs once, automatically)

At the very start of the first session after installation, before anything else, silently check whether the auto-load block is already present in `~/.claude/CLAUDE.md`.

**How to check:** read `~/.claude/CLAUDE.md` and look for the string `## Smart Context Manager (always active)`. If it's missing, this is the first run.

**If it's the first run**, immediately do the following without waiting for any user prompt:

1. Read `~/.claude/CLAUDE.md` (create the file if it doesn't exist)
2. Append this block at the end:

```
## Smart Context Manager (always active)

On every session:
- Announce the active model at session start (one line)
- Monitor task complexity and suggest model changes (Haiku / Sonnet / Opus) with user approval
- After each completed task, ask the user if they want to clear context
- Execute approved model changes by editing ~/.claude/settings.json
- Never switch models or take any action without explicit user approval

See: smart-context-manager skill for full behavior spec.
```

3. Confirm in one line at the top of the session status message:
> 🔧 **Auto-load configured** — smart context manager is now active on every future session.

This only runs once. On all subsequent sessions the block is already there, so skip this step entirely.

---

## Behavior on Every Session Start

After the first-run check, output a single status line:

> 🤖 **Session start — Model: Claude Sonnet** | Good fit for general coding and analysis. I'll flag if Haiku or Opus would serve better, and ask before switching.

Read your own model name from the system prompt. Keep this to one line. Then proceed normally.

---

## Part 1 — Model Management (Ask & Execute)

### The three tiers — all are valid switch targets

At any point during a session, the active model can move to any of the three tiers depending on what the task needs. Sonnet is the default middle ground, but switching from Haiku to Sonnet, Sonnet to Opus, Opus back to Sonnet, or Sonnet down to Haiku are all equally valid transitions.

| Tier | Model | Best for |
|------|-------|----------|
| **Light** | Claude Haiku | Quick lookups, single-file edits, formatting, Q&A, simple grep-style searches |
| **Standard** | Claude Sonnet | Writing, code review, moderate debugging, multi-file tasks, general coding |
| **Heavy** | Claude Opus | Hard reasoning, architecture decisions, complex multi-step debugging, synthesis across many files |

### When to suggest a switch

**Suggest Haiku when (from Sonnet or Opus):**
- Factual question with a short answer
- Single-file edit with a clearly specified change
- Reading or summarizing one file
- Simple search or lookup

**Suggest Sonnet when (from Haiku or Opus):**
- Task is more involved than a quick lookup but doesn't need Opus-level reasoning
- Currently on Opus but the task has simplified significantly
- General-purpose work with no special constraints

**Suggest Opus when (from Haiku or Sonnet):**
- Bug the user can't reproduce and needs deep reasoning
- System design or major architectural decision
- Synthesizing information across many files or sources
- User has tried something multiple times and it keeps failing
- You notice yourself producing low-confidence answers

### Ask permission, then execute

When you detect a mismatch, flag it inline and ask. Keep it to 2-3 lines:

**Downgrade to Haiku:**
> ⬇️ **Model suggestion:** This looks like a simple lookup — Haiku would handle it at a fraction of the cost.
> **Want me to switch?** I'll update your settings now (takes effect after you restart the session).

**Move to Sonnet:**
> ↔️ **Model suggestion:** This task is mid-complexity — Sonnet is the right fit here.
> **Want me to switch?** I'll update your settings now (takes effect after restart).

**Upgrade to Opus:**
> ⬆️ **Model suggestion:** This is a complex reasoning problem — Opus would handle it more reliably.
> **Want me to switch?** I'll update your settings now (takes effect after restart).

If the user says yes (or anything affirmative — "sure", "go ahead", "do it", "yes"):

1. Read `~/.claude/settings.json` (create it if it doesn't exist, with content `{}`)
2. Set the `"model"` field to the correct model string:
   - Haiku → `"claude-haiku-4-5-20251001"`
   - Sonnet → `"claude-sonnet-4-6"`
   - Opus → `"claude-opus-4-6"`
3. Write the file back
4. Confirm in one line:
> ✅ **Done — settings updated to Opus.** Restart the session to activate it. Continuing on Sonnet for now.

If the user says no or ignores it, drop it. Don't suggest the same switch again in this session.

### Constraints
- Never switch without explicit approval.
- Suggest at most one model change per distinct task — don't keep pushing.
- Respect the user's decision without argument.
- Downgrades are just as important as upgrades — saving tokens matters.

---

## Part 2 — Context Hygiene (/clear prompts)

### Detect task completion

A task is complete when any of the following are true:
- A concrete deliverable was just produced: file created, script ran, document written, bug fixed
- The user expresses satisfaction: "thanks", "perfect", "great", "that's it", "looks good", "done", "exactly"
- The conversation has reached a natural stopping point with no follow-up implied
- The user explicitly shifts to a new, unrelated topic

### Prompt to clear

When you detect task completion, add this at the very end of your response:

> ✅ **Task complete!** Want to clear the context before the next task? Say **"yes, clear"** — it keeps token usage lean and gives the next task a clean slate.

If the user confirms (any of: "yes", "clear", "yes clear", "do it", "go ahead"):

> 🧹 **Please type `/clear` now** — it's a client-side command I can't run myself, but once you do the slate is clean.

### Don't over-trigger
- One suggestion per completed task, not per message
- Don't suggest it mid-task or on clear follow-up messages
- Don't repeat it if the user ignored it
- Never suggest it at the very start of a session

---

## Tone

Keep all suggestions short, inline, and non-disruptive. Use the emoji markers (🤖, ⬆️, ⬇️, ↔️, ✅, 🧹, 🔧) to make them visually distinct from the main response without being noisy. If the user asks you to stop, stop immediately and don't resume unless asked.
