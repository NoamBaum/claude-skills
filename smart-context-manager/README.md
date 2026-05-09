# Smart Context Manager

**Version:** 1.0
**Author:** Noam Baum
**Compatibility:** Claude Code (CLI)

---

## What It Does

Smart Context Manager is a Claude Code skill that runs automatically on every session. It tackles two of the biggest sources of wasted tokens in Claude Code: using the wrong model for the task, and carrying stale context from one task into the next.

### 1. Intelligent Model Switching

At the start of every session, Claude announces which model is active and why it's a good fit. As you work, it continuously monitors task complexity and suggests switching to a more appropriate model — both upgrades and downgrades:

| Model | Best for |
|-------|----------|
| **Claude Haiku** | Quick lookups, single-file edits, simple Q&A, formatting |
| **Claude Sonnet** | General coding, writing, code review, multi-file tasks |
| **Claude Opus** | Hard reasoning, architecture decisions, complex debugging |

When a better model is detected, Claude asks your permission before doing anything. If you say yes, it updates `~/.claude/settings.json` directly — no manual editing needed. The switch takes effect on the next session restart.

### 2. Context Hygiene Prompts

When a task finishes, Claude detects it (deliverable produced, user says "thanks", topic shifts, etc.) and asks if you want to clear the context. One-word confirmation ("yes") — then you type `/clear` and the next task starts completely fresh.

> **Note:** `/clear` is a client-side command that Claude cannot execute automatically. The skill makes it as frictionless as possible — you just say yes, then type `/clear`.

### 3. Auto-Load on Every Session

On first use, the skill silently configures `~/.claude/CLAUDE.md` so the behavior is active on every future Claude Code session without any trigger phrase.

---

## Installation

### Option A — Install via `.skill` file (recommended)

1. Download `smart-context-manager.skill` from this repo
2. Open Claude Code (Cowork or CLI)
3. Click **Save skill** / drag the `.skill` file into the skills panel
4. The skill is now installed

### Option B — Manual install

1. Clone or download this repo
2. Copy the `smart-context-manager/` folder into your Claude skills directory:
   ```
   ~/.claude/skills/smart-context-manager/
   ```
3. Verify the folder contains `SKILL.md`

---

## First-Run Setup (Automatic)

On your very first session after installation, the skill will:

1. Detect it hasn't been configured yet
2. Automatically add the auto-load block to `~/.claude/CLAUDE.md`
3. Confirm with a single line:
   > 🔧 **Auto-load configured** — smart context manager is now active on every future session.

You don't need to do anything. Every session after that, it just runs.

---

## Usage

Once installed, the skill is always on. Here's what you'll see:

**Session start:**
```
🤖 Session start — Model: Claude Sonnet | Good fit for general coding and analysis.
   I'll flag if Haiku or Opus would serve better, and ask before switching.
```

**Model downgrade suggestion:**
```
⬇️ Model suggestion: This looks like a simple lookup — Haiku would handle it at a fraction of the cost.
   Want me to switch? I'll update your settings now (takes effect after restart).
```

**Model upgrade suggestion:**
```
⬆️ Model suggestion: This is a complex architectural problem — Opus would reason through it more reliably.
   Want me to switch? I'll update your settings now (takes effect after restart).
```

**Task complete:**
```
✅ Task complete! Want to clear the context before the next task? Say "yes, clear"
   — it keeps token usage lean and gives the next task a clean slate.
```

---

## How Model Switching Works

When you approve a model switch, the skill edits `~/.claude/settings.json`:

```json
{
  "model": "claude-opus-4-6"
}
```

Available model strings:
- Haiku: `claude-haiku-4-5-20251001`
- Sonnet: `claude-sonnet-4-6`
- Opus: `claude-opus-4-6`

The change takes effect the next time you start a Claude Code session.

---

## Stopping Suggestions

If you want to pause the suggestions at any point, just tell Claude:
> "Stop making model suggestions" or "Stop asking about /clear"

It will stop immediately and not resume unless you ask.

---

## Version History

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-05-09 | Initial release — model switching (Haiku/Sonnet/Opus), context clear prompts, auto-load via CLAUDE.md |
