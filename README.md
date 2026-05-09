# Claude Skills

A collection of Claude Code skills for productivity and cost optimization.

---

## Skills

### [smart-context-manager](./smart-context-manager) — v1.0

Automatically manages model selection and context hygiene across every Claude Code session.

**Features:**
- Announces active model (Haiku / Sonnet / Opus) at session start
- Monitors task complexity and suggests model upgrades or downgrades — asks permission, then edits `~/.claude/settings.json` directly
- Detects task completion and prompts to clear context
- Self-configures auto-load via `~/.claude/CLAUDE.md` on first run

**Install:** Download `smart-context-manager.skill` and open it in Claude Code (Cowork or CLI).

---

## How to Install a Skill

### Option A — .skill file (recommended)
1. Download the `.skill` file from the skill's folder
2. Open Claude Code (Cowork or CLI)
3. Click **Save skill** / drag the file into the skills panel

### Option B — Manual
1. Clone this repo
2. Copy the skill folder into `~/.claude/skills/`
3. Verify it contains `SKILL.md`

---

## License

MIT
