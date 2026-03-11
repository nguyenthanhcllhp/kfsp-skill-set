# KFSP Skill Set — Installation Guide

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed and running
- A project open in Claude Code

## Installation Methods

### Method 1: Project-Level (Recommended)

Skills only available in one specific project:

```bash
# Find your project hash
ls ~/.claude/projects/
# Look for the folder matching your project path

# Copy skills
cp -r commands/kfsp/ ~/.claude/projects/<your-project-hash>/commands/kfsp/

# Verify
ls ~/.claude/projects/<your-project-hash>/commands/kfsp/
# Should show 14 .md files
```

### Method 2: Global Install

Skills available in ALL Claude Code projects:

```bash
# Copy to global commands
cp -r commands/kfsp/ ~/.claude/commands/kfsp/

# Verify
ls ~/.claude/commands/kfsp/
```

### Method 3: One-Line Install (from GitHub)

```bash
# Clone and install globally
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git /tmp/kfsp-skill-set \
  && cp -r /tmp/kfsp-skill-set/commands/kfsp/ ~/.claude/commands/kfsp/ \
  && rm -rf /tmp/kfsp-skill-set \
  && echo "✅ KFSP Skill Set installed. Run /kfsp:help to get started."
```

## Post-Install Setup

### 1. Verify installation

In Claude Code, type:
```
/kfsp:help
```

You should see the complete skill guide with 13 skills across 5 tiers.

### 2. Initialize Dev Journal

```
/kfsp:dev-journal --init
```

This creates the journal directory structure in your project.

### 3. Copy reference docs (optional)

If you want the Interaction Surface Map template:
```bash
cp references/INTERACTION_SURFACE_MAP.md your-project/docs/
```

## Installing Companion Tools

### GSD (Get Shit Done)

Check if already installed:
```
/gsd:help
```

If not installed, see: https://github.com/get-shit-done/gsd

GSD provides: `/gsd:new-project`, `/gsd:plan-phase`, `/gsd:execute-phase`, `/gsd:verify-work`

### Ralph Loop

Check if already installed:
```
/ralph-loop --help
```

If not installed, see: https://github.com/anthropics/ralph-loop

Ralph provides: `ralph --monitor`, `ralph-enable`

### Domain Skills (Optional)

For Flutter projects:
```bash
# If your project uses Flutter, add Flutter-specific skills:
# These are typically in your project's .claude/skills/ directory
```

## Adapting Skills to Your Project

See the "Adapting for Your Project" section in README.md for details on:
1. Source path detection
2. Design token paths
3. Doc registry mapping
4. Feature list customization

## Updating

```bash
# Pull latest from GitHub
cd /path/to/kfsp-skill-set && git pull

# Re-copy commands
cp -r commands/kfsp/ ~/.claude/commands/kfsp/
# Or: cp -r commands/kfsp/ ~/.claude/projects/<hash>/commands/kfsp/
```

## Uninstalling

```bash
# Project-level
rm -rf ~/.claude/projects/<your-project-hash>/commands/kfsp/

# Global
rm -rf ~/.claude/commands/kfsp/
```

## Troubleshooting

### Skills don't appear as slash commands
- Restart Claude Code session
- Verify files are in correct directory
- Check file permissions: `chmod 644 ~/.claude/commands/kfsp/*.md`

### Skills can't find source code
- Edit the source path detection in each skill
- Default paths are KFSP-specific — adapt to your project structure

### "File has not been read yet" error
- This is normal — skills read files on demand
- The skill will auto-read required files during execution
