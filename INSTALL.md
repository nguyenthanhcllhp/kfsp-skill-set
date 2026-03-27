# KFSP Skill Set v2.0 — Installation Guide

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed and running
- A project open in Claude Code

## What Gets Installed

| Component | Location | Count | Purpose |
|-----------|----------|-------|---------|
| Slash Commands | `~/.claude/commands/kfsp/` | 16 files | Quick manual checks (`/kfsp:sweep myfile.dart`) |
| Agents | `~/.claude/agents/` | 15 files | Autonomous subprocess agents (`kfsp-guard`, `kfsp-sweep`...) |

**Why both?** Commands run inline in your conversation (quick checks). Agents spawn as subprocesses with their own context window (deep scans, parallel execution, orchestrator-triggered).

---

## Installation

### One-Line Install (Recommended)

```bash
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git /tmp/kfsp-skill-set \
  && mkdir -p ~/.claude/commands/kfsp/ ~/.claude/agents/ \
  && cp -r /tmp/kfsp-skill-set/commands/kfsp/ ~/.claude/commands/kfsp/ \
  && cp /tmp/kfsp-skill-set/agents/kfsp/*.md ~/.claude/agents/ \
  && rm -rf /tmp/kfsp-skill-set \
  && echo "✅ KFSP Skill Set v2.0 installed (commands + agents)."
```

### Manual Install (from cloned repo)

```bash
# 1. Clone
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git
cd kfsp-skill-set

# 2. Install slash commands (16 files)
mkdir -p ~/.claude/commands/kfsp/
cp -r commands/kfsp/ ~/.claude/commands/kfsp/

# 3. Install agents (15 files)
mkdir -p ~/.claude/agents/
cp agents/kfsp/*.md ~/.claude/agents/

# 4. (Optional) Copy reference docs
cp references/INTERACTION_SURFACE_MAP.md your-project/docs/
```

---

## Verification

### Step 1: Check file counts

```bash
# Slash commands — should show 16
ls ~/.claude/commands/kfsp/ | wc -l

# Agents — should show 15 kfsp-* files
ls ~/.claude/agents/kfsp-*.md 2>/dev/null | wc -l
```

### Step 2: Verify slash commands work

In Claude Code, type:
```
/kfsp:help
```

You should see the complete guide with 15 skills across 5 tiers.

### Step 3: Verify agents are available

In Claude Code, when you use the Agent tool, the following agent types should be available:
- `kfsp-guard`, `kfsp-sweep`, `kfsp-ux-parity`, `kfsp-pre-commit`
- `kfsp-doc-pilot`, `kfsp-pre-mortem`, `kfsp-sentinel`, `kfsp-surface-test`
- `kfsp-release-gate`, `kfsp-dev-journal`, `kfsp-sync-check`, `kfsp-post-phase`
- `kfsp-incident-review`, `kfsp-ux-audit`, `kfsp-product-health`

### Expected file listing

```
~/.claude/commands/kfsp/
├── help.md
├── guard.md
├── dev-journal.md
├── incident-review.md
├── sweep.md
├── pre-commit.md
├── sync-check.md
├── post-phase.md
├── surface-test.md
├── sentinel.md
├── release-gate.md
├── ux-parity.md
├── ux-audit.md
├── doc-pilot.md
├── pre-mortem.md
└── product-health.md

~/.claude/agents/
├── kfsp-guard.md
├── kfsp-sweep.md
├── kfsp-ux-parity.md
├── kfsp-pre-commit.md
├── kfsp-doc-pilot.md
├── kfsp-pre-mortem.md
├── kfsp-sentinel.md
├── kfsp-surface-test.md
├── kfsp-release-gate.md
├── kfsp-dev-journal.md
├── kfsp-incident-review.md
├── kfsp-sync-check.md
├── kfsp-post-phase.md
├── kfsp-ux-audit.md
└── kfsp-product-health.md
```

---

## Post-Install Setup

### Initialize Dev Journal

```
/kfsp:dev-journal --init
```

Creates the journal directory structure in your project for recording decisions and incidents.

### Install Companion Tools (KHUYẾN NGHỊ MẠNH)

KFSP hoạt động TỐT NHẤT khi kết hợp với GSD + Ralph. Xem [`ORCHESTRATION.md`](ORCHESTRATION.md).

| Tool | Purpose | Check | Install |
|------|---------|-------|---------|
| [GSD](https://github.com/gsd-build/get-shit-done) | Plan → Execute → Verify | `/gsd:help` | See GSD repo |
| [Ralph Loop](https://github.com/frankbria/ralph-claude-code) | Autonomous iteration loops | `ralph --help` | See Ralph repo |

**Verify:** Chạy `/kfsp:session-start` — tự detect tất cả.

---

## Updating

```bash
# Pull latest
cd /path/to/kfsp-skill-set && git pull

# Re-install both commands AND agents
cp -r commands/kfsp/ ~/.claude/commands/kfsp/
cp agents/kfsp/*.md ~/.claude/agents/
```

## Uninstalling

```bash
# Remove slash commands
rm -rf ~/.claude/commands/kfsp/

# Remove agents
rm -f ~/.claude/agents/kfsp-*.md
```

---

## Troubleshooting

### Skills don't appear as slash commands
- Restart Claude Code session (close and reopen)
- Verify files exist: `ls ~/.claude/commands/kfsp/`
- Check permissions: `chmod 644 ~/.claude/commands/kfsp/*.md`

### Agents not spawning
- Verify files exist: `ls ~/.claude/agents/kfsp-*.md`
- Check YAML frontmatter is valid (each file starts with `---`)
- Restart Claude Code session

### Skills can't find source code
- Skills use path detection — edit each skill to match your project structure
- Default paths are KFSP-specific (Vietnamese stock app)

### "File has not been read yet" error
- Normal behavior — skills read files on demand during execution
