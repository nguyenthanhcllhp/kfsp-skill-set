# 🥋 KFSP Skill Set v2.0

**15 production-grade skills for agent-driven product development.**

Build apps from zero to store with confidence — for both human teams and AI agent swarms.

## What's New in v2.0

### From Skill Specs → Agent Format

**v1.0** listed skills as slash command specs — instructions that Claude reads and follows inline.

**v2.0** converts every skill into a **standalone agent** (`.claude/agents/` format) alongside the slash commands. Why?

| Problem with v1.0 | How v2.0 solves it |
|---|---|
| Skills run inline in main conversation — consume context window | Agents spawn as **subprocesses** with their own context |
| Skills can't be composed or orchestrated | Agents can be **spawned by other agents** (e.g., GSD executor spawns kfsp-guard) |
| Skills must be manually triggered by user typing `/kfsp:sweep` | Agents can be **auto-triggered** by orchestrators after every code change |
| Skills share context with dev work — get confused | Agents get **fresh context** — focused on their single job |
| No parallelism — one skill at a time | Multiple agents can run **in parallel** (sweep + ux-parity simultaneously) |

**Bottom line:** Slash commands = quick manual checks. Agents = autonomous protection layer that runs without human reminding.

### Package Structure

```
kfsp-skill-set/
├── README.md                          ← You are here
├── INSTALL.md                         ← Installation + verification guide
├── commands/kfsp/                     ← 16 slash command files (/kfsp:*)
│   ├── help.md
│   ├── guard.md                       ← T1: Continuous guardian
│   ├── dev-journal.md                 ← T1: Developer journal
│   ├── incident-review.md             ← T1: Post-incident review
│   ├── sweep.md                       ← T2: Blast radius analysis
│   ├── pre-commit.md                  ← T2: Commit validation
│   ├── sync-check.md                  ← T2: Source sync
│   ├── post-phase.md                  ← T2: Phase completion
│   ├── surface-test.md                ← T3: 4-surface contract testing
│   ├── sentinel.md                    ← T3: Regression detection
│   ├── release-gate.md                ← T3: Pre-release gate
│   ├── ux-parity.md                   ← T4: Design-code parity
│   ├── ux-audit.md                    ← T4: UX heuristic evaluation
│   ├── doc-pilot.md                   ← T4-T5: Documentation autopilot
│   ├── pre-mortem.md                  ← T5: Risk forecast
│   └── product-health.md              ← T5: PM health dashboard
├── agents/kfsp/                       ← 15 agent files (kfsp-*.md) ← NEW in v2.0
│   ├── kfsp-guard.md
│   ├── kfsp-sweep.md
│   ├── kfsp-ux-parity.md
│   ├── kfsp-pre-commit.md
│   ├── kfsp-doc-pilot.md
│   ├── kfsp-pre-mortem.md
│   ├── kfsp-sentinel.md
│   ├── kfsp-surface-test.md
│   ├── kfsp-release-gate.md
│   ├── kfsp-dev-journal.md
│   ├── kfsp-incident-review.md
│   ├── kfsp-sync-check.md
│   ├── kfsp-post-phase.md
│   ├── kfsp-ux-audit.md
│   └── kfsp-product-health.md
├── references/
│   └── INTERACTION_SURFACE_MAP.md
└── examples/
    └── README.md
```

---

## 5-Tier Protection Model

```
┌─────────────────────────────────────────────────────────┐
│  T5 🎯 BUSINESS IMPACT                                  │
│  doc-pilot · pre-mortem · product-health                │
├─────────────────────────────────────────────────────────┤
│  T4 👤 USER EXPERIENCE                                   │
│  ux-parity · ux-audit · doc-pilot                       │
├─────────────────────────────────────────────────────────┤
│  T3 🧪 QUALITY ASSURANCE                                 │
│  surface-test · sentinel · release-gate                 │
├─────────────────────────────────────────────────────────┤
│  T2 🔧 CODE INTEGRITY                                    │
│  sweep · pre-commit · sync-check · post-phase           │
├─────────────────────────────────────────────────────────┤
│  T1 🛡️ AGENT SAFETY                                      │
│  guard · dev-journal · incident-review                  │
└─────────────────────────────────────────────────────────┘
```

---

## Why?

When AI agents build your product, things can go wrong silently:
- **Hallucination** — agent makes up data or ignores constraints
- **Regression** — new feature breaks old ones
- **Context loss** — agent forgets decisions from 3 conversations ago
- **Drift** — code diverges from design, docs fall behind
- **Silent failure** — contracts between services break without error

KFSP Skill Set provides **5 layers of protection** across **4 interaction surfaces** (user-facing, backend, third-party services, internal components).

---

## Quick Start

### Install (one command)

```bash
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git /tmp/kfsp-skill-set \
  && cp -r /tmp/kfsp-skill-set/commands/kfsp/ ~/.claude/commands/kfsp/ \
  && cp /tmp/kfsp-skill-set/agents/kfsp/*.md ~/.claude/agents/ \
  && rm -rf /tmp/kfsp-skill-set \
  && echo "✅ KFSP Skill Set v2.0 installed (commands + agents)."
```

### Verify

```bash
# Check slash commands (should show 16 files)
ls ~/.claude/commands/kfsp/ | wc -l

# Check agents (should show 15 kfsp-* files)
ls ~/.claude/agents/kfsp-*.md | wc -l

# In Claude Code — type /kfsp: and see dropdown
/kfsp:help
```

See `INSTALL.md` for detailed installation + verification guide.

### Use

```bash
/kfsp:pre-mortem "Add user authentication"   # Forecast risks BEFORE building
# ... write code ...
/kfsp:sweep auth_controller.dart              # Check blast radius AFTER changes
/kfsp:ux-parity login                         # Verify design match
/kfsp:pre-commit                              # Validate BEFORE committing
/kfsp:doc-pilot --what-changed                # Track doc updates needed
```

---

## All Skills

### T5 🎯 Business Impact + T4 👤 User Experience

| Skill | Command | Agent | What it does |
|-------|---------|-------|-------------|
| Product Health | `/kfsp:product-health` | `kfsp-product-health` | PM's 8-dimension dashboard. Score /100. |
| Doc Pilot | `/kfsp:doc-pilot` | `kfsp-doc-pilot` | Tracks which docs need updating. Ensures user guide ready before store. |
| Pre-Mortem | `/kfsp:pre-mortem` | `kfsp-pre-mortem` | Imagines feature already failed. Risks across 4 surfaces. |
| UX Audit | `/kfsp:ux-audit` | `kfsp-ux-audit` | Nielsen 10 heuristics + accessibility + domain checks. |
| UX Parity | `/kfsp:ux-parity` | `kfsp-ux-parity` | Compares actual UI vs design specs. Finds hardcoded colors, spacing drift. |

### T3 🧪 Quality Assurance

| Skill | Command | Agent | What it does |
|-------|---------|-------|-------------|
| Surface Test | `/kfsp:surface-test` | `kfsp-surface-test` | Tests contracts across 4 surfaces. |
| Sentinel | `/kfsp:sentinel` | `kfsp-sentinel` | Baseline before changes, compare after. Detects regressions. |
| Release Gate | `/kfsp:release-gate` | `kfsp-release-gate` | 10 gates, score /100, Go/No-Go decision. |

### T2 🔧 Code Integrity

| Skill | Command | Agent | What it does |
|-------|---------|-------|-------------|
| Sweep | `/kfsp:sweep` | `kfsp-sweep` | Blast radius after code changes. Tokens, routes, providers, tests, docs. |
| Pre-Commit | `/kfsp:pre-commit` | `kfsp-pre-commit` | No hardcoded values, no secrets, no broken imports. |
| Sync Check | `/kfsp:sync-check` | `kfsp-sync-check` | All source copies in sync, docs match code. |
| Post-Phase | `/kfsp:post-phase` | `kfsp-post-phase` | Phase completion checklist: docs, sync, regression. |

### T1 🛡️ Agent Safety

| Skill | Command | Agent | What it does |
|-------|---------|-------|-------------|
| Guard | `/kfsp:guard` | `kfsp-guard` | Health score /100. Structure, design system, cross-cutting, safety. |
| Dev Journal | `/kfsp:dev-journal` | `kfsp-dev-journal` | Records decisions/incidents as searchable precedents. |
| Incident Review | `/kfsp:incident-review` | `kfsp-incident-review` | 5 Whys root cause, blast radius, prevention plan. |

---

## Commands vs Agents — When to Use Which

| Use Case | Use Command | Use Agent |
|----------|-------------|-----------|
| Quick manual check | `/kfsp:sweep myfile.dart` | — |
| Auto-trigger after code change | — | Agent spawned by orchestrator |
| Part of GSD execute-phase | — | GSD executor spawns kfsp-guard |
| Parallel checks | — | Spawn sweep + ux-parity simultaneously |
| Deep scan needing focus | — | Agent gets full context window |
| Quick pre-commit gate | `/kfsp:pre-commit` | — |

---

## Workflows

### Frequency Pyramid

```
Every commit (~30s):      /kfsp:pre-commit
Every change (~2min):     /kfsp:sweep  (or auto: kfsp-sweep agent)
Every feature (~5min):    /kfsp:pre-mortem → dev → /kfsp:ux-parity
Every phase (~10min):     /kfsp:post-phase → /kfsp:doc-pilot
Every release (~15min):   /kfsp:release-gate → /kfsp:surface-test
On demand:                /kfsp:incident-review, /kfsp:dev-journal
Weekly:                   /kfsp:guard --deep
```

### Chain Workflows

```
New Feature:   pre-mortem → [develop] → sweep → ux-parity → doc-pilot → dev-journal
Bug Fix:       [fix] → sweep → pre-commit → commit → dev-journal
Release:       sentinel --baseline → [develop] → sentinel --compare → release-gate → surface-test
Agent Loop:    [agent task] → guard → if OK continue, if FAIL → stop + incident-review
```

---

## The 4 Interaction Surfaces

```
                    ┌──────────────┐
                    │ 🔵 FRONT     │
                    │ User ↔ App   │
                    └──────┬───────┘
                           │
      ┌────────────────────┼────────────────────┐
      │                    │                    │
┌─────┴──────┐    ┌────────┴───────┐   ┌───────┴──────┐
│ 🟡 LATERAL │    │  🟢 INTERNAL   │   │ 🟡 LATERAL   │
│ 3rd Party  │    │  Component ↔   │   │ Analytics    │
│ Services   │    │  Component     │   │ Monitoring   │
└────────────┘    └────────┬───────┘   └──────────────┘
                           │
                    ┌──────┴───────┐
                    │ 🔴 BACK      │
                    │ App ↔ Server │
                    └──────────────┘
```

See `references/INTERACTION_SURFACE_MAP.md` for the complete map with 36 interaction types.

---

## Companion Tools

| Tool | Purpose | How it pairs with KFSP |
|------|---------|----------------------|
| [GSD](https://github.com/gsd-build/get-shit-done) | Plan → Execute → Verify | GSD handles *what to build*, KFSP ensures *it's built safely* |
| [Ralph Loop](https://github.com/frankbria/ralph-claude-code) | Autonomous iteration | Ralph handles *execution loops*, KFSP guard verifies after each loop |
| Domain Skills | Tech-specific patterns | Flutter, React, Python skills for conventions |

---

## Adapting for Your Project

KFSP skills are designed to be project-agnostic. To adapt:

1. **Source paths** — Edit path detection in each skill to match your project structure
2. **Design tokens** — Edit `ux-parity.md` to point to your token files (CSS, Tailwind, etc.)
3. **Doc registry** — Edit `doc-pilot.md` to map your documentation structure
4. **Feature list** — Edit `guard.md` to match your expected folder structure

Skills that work without changes: `pre-mortem`, `dev-journal`, `incident-review`, `sentinel`, `release-gate`

---

## Disclaimer

**This skill set is NOT created by a domain expert.** It is distilled from personal thinking, experience, and intuition of a non-tech Product Manager building a real product with AI. The ideas, checklists, and frameworks here are generalized from a Vietnamese stock analysis app — if something feels unfamiliar, try asking AI to translate/adapt it to your context rather than forcing interpretation.

**You are fully responsible for your own actions.** Whether you use these skills, modify them, or ignore them — every decision and its consequences belong to the person who acts.

**This is a snapshot of work-in-progress.** Skills will be updated as the product evolves.

---

## About the Creator

**[Thanh Nguyen](https://github.com/nguyenthanhcllhp)** — [read my story here](https://github.com/nguyenthanhcllhp).

---

## License

MIT — Use freely, adapt for your projects.

---

## Credits

Created by **[Thanh Nguyen](https://github.com/nguyenthanhcllhp)** with [Claude AI](https://claude.ai) as co-developer.

Born from building [KFSP](https://kfsp.vn/home/#trang-chu).

Inspired by:
- [GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) — planning & execution framework
- [Ralph Loop](https://github.com/frankbria/ralph-claude-code) — autonomous iteration
