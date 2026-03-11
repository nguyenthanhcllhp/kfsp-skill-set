# 🥋 KFSP Skill Set

**15 production-grade skills for agent-driven product development.**

Build apps from zero to store with confidence — for both human teams and AI agent swarms.

```
┌─────────────────────────────────────────────────────────┐
│  T5 🎯 BUSINESS IMPACT                                  │
│  doc-pilot · pre-mortem                                 │
├─────────────────────────────────────────────────────────┤
│  T4 👤 USER EXPERIENCE                                   │
│  ux-parity · doc-pilot                                  │
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

KFSP Skill Set provides **5 layers of protection** that catch these problems across **4 interaction surfaces** (user-facing, backend, third-party services, internal components).

---

## Quick Start

### 1. Install

```bash
# Clone or download
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git

# Copy commands to your project's Claude config
cp -r kfsp-skill-set/commands/kfsp/ \
  ~/.claude/projects/<your-project-hash>/commands/kfsp/

# Or install globally (available in all projects)
cp -r kfsp-skill-set/commands/kfsp/ \
  ~/.claude/commands/kfsp/
```

### 2. Initialize

```bash
# In Claude Code:
/kfsp:dev-journal --init    # Start your developer journal
/kfsp:doc-pilot --status    # Baseline your documentation
/kfsp:help                  # See all commands
```

### 3. Develop!

```bash
/kfsp:pre-mortem "Add user authentication"   # Forecast risks
# ... write code ...
/kfsp:pre-commit                              # Validate before commit
/kfsp:sweep auth_controller.dart              # Check blast radius
/kfsp:doc-pilot --what-changed auth           # Update docs
```

---

## All 13 Skills

### T5 🎯 Business Impact + T4 👤 User Experience

| Skill | What it does | When to use |
|-------|-------------|-------------|
| `/kfsp:product-health` | **NEW** PM's comprehensive dashboard: 8 dimensions (UX, Tech, Docs, Business, Legal, Ops, Data, Growth). Score /80 with gap analysis. | Monthly or before major launch |
| `/kfsp:doc-pilot` | Tracks which docs need updating. Knows what info goes into which file. Ensures user guide is ready before store submission. | After every feature change |
| `/kfsp:pre-mortem` | Before building anything, imagines it already failed. Identifies risks across all 4 interaction surfaces. Produces Go/No-Go decision. | Before starting new feature |
| `/kfsp:ux-audit` | **NEW** UX evaluation using Nielsen 10 heuristics + accessibility + user flow analysis. Designed for PMs with no UX background. Templates included. | After building screens, before release |
| `/kfsp:ux-parity` | Compares actual UI against design specs. Finds hardcoded colors, wrong spacing, typography drift. | After building screens |

### T3 🧪 Quality Assurance

| Skill | What it does | When to use |
|-------|-------------|-------------|
| `/kfsp:surface-test` | Tests interaction contracts across 4 surfaces: user↔app, app↔backend, app↔services, component↔component. | Before releases |
| `/kfsp:sentinel` | Takes baseline snapshot before changes, compares after. Detects regressions in files, routes, providers, tokens. | Before/after refactoring |
| `/kfsp:release-gate` | 10 comprehensive gates (build, security, design, UX, tests, deps, surfaces, docs, store, rollback). Scores /100 with Go/No-Go. | Before submitting to store |

### T2 🔧 Code Integrity

| Skill | What it does | When to use |
|-------|-------------|-------------|
| `/kfsp:sweep` | Analyzes blast radius after code changes. Checks design tokens, routes, providers, widgets, tests, docs. | After every code change |
| `/kfsp:pre-commit` | Validates code before commit: no hardcoded values, no secrets, no broken imports, file size limits. | Before every commit |
| `/kfsp:sync-check` | Verifies all source copies are in sync, docs match code reality. | Weekly or before handoff |
| `/kfsp:post-phase` | Post-milestone checklist: update docs, sync sources, run regression, archive artifacts. | After completing a phase |

### T1 🛡️ Agent Safety

| Skill | What it does | When to use |
|-------|-------------|-------------|
| `/kfsp:guard` | Continuous guardian. Checks structural integrity, design system fidelity, cross-cutting concerns, spec alignment, agent safety. Health score /100. | After agent runs any task |
| `/kfsp:dev-journal` | Developer journal — records decisions, incidents, learnings as "precedents" (like case law). Searchable by future agents/humans. | After every important decision |
| `/kfsp:incident-review` | Post-incident analysis: timeline, 5 Whys root cause, blast radius on 4 surfaces, prevention plan, skill updates needed. | After any bug or crash |

### Utility

| Skill | What it does |
|-------|-------------|
| `/kfsp:help` | Shows complete guide with tier map, workflows, and portable setup instructions |

---

## The 4 Interaction Surfaces

Every app interacts in 4 directions. KFSP skills monitor all of them:

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

Each surface has **contracts** (expected behavior), **standards** (thresholds), **monitors** (continuous checks), and **tests** (verification).

See `references/INTERACTION_SURFACE_MAP.md` for the complete map with 36 interaction types.

---

## Workflows

### Frequency Pyramid

Run the most frequent checks fastest:

```
Every commit (~30s):      /kfsp:pre-commit
Every change (~2min):     /kfsp:sweep
Every feature (~5min):    /kfsp:pre-mortem → dev → /kfsp:ux-parity
Every phase (~10min):     /kfsp:post-phase → /kfsp:doc-pilot
Every release (~15min):   /kfsp:release-gate → /kfsp:surface-test
On demand:                /kfsp:incident-review, /kfsp:dev-journal
Weekly:                   /kfsp:guard --deep
```

### Chain Workflows

**New Feature:**
```
pre-mortem → [develop] → sweep → ux-parity → doc-pilot --what-changed → dev-journal --log decision
```

**Bug Fix:**
```
[fix] → sweep → pre-commit → commit → dev-journal --log incident
```

**Release:**
```
sentinel --baseline → [develop] → sentinel --compare → release-gate → surface-test all → doc-pilot --checklist
```

**Agent Autonomous:**
```
[agent task] → guard → if OK continue, if FAIL → stop + incident-review
```

---

## Companion Tools (Recommended)

KFSP Skill Set works standalone, but is designed to pair with:

### 🔧 GSD (Get Shit Done) — Planning & Execution Framework

> Spec-driven development: plan → tasks → execute → verify

- **What:** 33 commands for project planning, phase management, and atomic execution
- **Why pair:** GSD handles *what to build*, KFSP skills ensure *it's built safely*
- **Install:** See [GSD GitHub](https://github.com/get-shit-done/gsd) or check if pre-installed:
  ```bash
  # Test if GSD is available:
  /gsd:help
  ```
- **Key commands:** `/gsd:new-project`, `/gsd:plan-phase`, `/gsd:execute-phase`, `/gsd:verify-work`

### 🔄 Ralph Loop — Autonomous Iteration

> AI runs in a loop until task is done. Git is memory between loops.

- **What:** Continuous self-referential AI loops with monitoring dashboard
- **Why pair:** Ralph handles *autonomous execution*, KFSP guard verifies after each loop
- **Install:** See [Ralph Loop](https://github.com/anthropics/ralph-loop) or:
  ```bash
  # Test if Ralph is available:
  /ralph-loop --help
  ```
- **Key commands:** `ralph --monitor`, `ralph-enable`

### 🦋 Domain Skills (Optional)

Add domain-specific skills based on your tech stack:

| Tech | Skill | Source |
|------|-------|--------|
| Flutter | `flutter-patterns`, `flutter-development` | Built-in or custom |
| React Native | `react-native-patterns` | Custom |
| Next.js | `nextjs-patterns` | Community |
| Python | `python-patterns` | Community |

### Combined Workflow

```
┌─────────────────────────────────────────────┐
│  PLAN  │  GSD: /gsd:plan-phase              │
│        │  KFSP: /kfsp:pre-mortem            │
├────────┼────────────────────────────────────┤
│  BUILD │  GSD: /gsd:execute-phase           │
│   or   │  KFSP: /kfsp:pre-commit (each)    │
│  Ralph │  Ralph: ralph --monitor            │
│        │  KFSP: /kfsp:guard (each loop)     │
├────────┼────────────────────────────────────┤
│ VERIFY │  GSD: /gsd:verify-work             │
│        │  KFSP: /kfsp:release-gate          │
│        │  KFSP: /kfsp:surface-test all      │
│        │  KFSP: /kfsp:doc-pilot --checklist │
└────────┴────────────────────────────────────┘
```

---

## Adapting for Your Project

KFSP skills are designed to be project-agnostic. To adapt:

### 1. Source Path Detection

Every skill auto-detects your source code location. Edit the path list in each skill:

```bash
# Default (KFSP Flutter):
for path in "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter" "Product/source"; do
  [ -d "$path/lib" ] && SOURCE="$path" && break
done

# Your project (e.g., React Native):
for path in "./src" "../app/src" "$HOME/projects/myapp/src"; do
  [ -d "$path" ] && SOURCE="$path" && break
done
```

### 2. Design Token Paths

Edit `ux-parity.md` to point to your design token files:

```bash
# KFSP Flutter:
COLORS="lib/core/theme/kfsp_colors.dart"

# Your project:
COLORS="src/styles/colors.ts"  # or tailwind.config.js, etc.
```

### 3. Doc Registry

Edit `doc-pilot.md` to map your documentation structure:

```markdown
# Your project's doc mapping:
| Change | Write to |
|--------|----------|
| API change | docs/api-spec.md |
| UI change | docs/components.md |
| Feature done | CHANGELOG.md |
```

### 4. Feature List

Edit `guard.md` expected folder structure to match your project.

### Skills That Work Without Changes

These skills are already project-agnostic:
- `pre-mortem` — risk analysis framework (universal)
- `dev-journal` — journal system (universal)
- `incident-review` — post-mortem template (universal)
- `sentinel` — file/route/provider diffing (universal)
- `release-gate` — release checklist (universal)

---

## For Agent Teams / Swarms

KFSP Skill Set is specifically designed for AI agent safety:

| Problem | How KFSP Solves It |
|---------|-------------------|
| **Hallucination** | `guard` verifies every change against real codebase |
| **Context loss** | `dev-journal` stores precedents, searchable by new agents |
| **Scope creep** | `pre-mortem` defines scope, `guard` enforces it |
| **Regression** | `sentinel` snapshots before/after, `sweep` traces blast radius |
| **Design drift** | `ux-parity` checks tokens, `sync-check` verifies copies |
| **Silent failures** | `surface-test` validates contracts on all 4 surfaces |
| **Missing docs** | `doc-pilot` auto-reminds and maps info → document |
| **No learning** | `dev-journal` records, `incident-review` analyzes patterns |
| **Loss of control** | 5-tier model + `guard` after every agent task |

### Agent Swarm Setup

```bash
# Each agent in the swarm should:
1. Read /kfsp:help on startup → understand available protections
2. Run /kfsp:guard after every task → verify safety
3. Log decisions via /kfsp:dev-journal --log decision → preserve context
4. Search journal via /kfsp:dev-journal --search <topic> → leverage precedents
```

---

## Package Contents

```
kfsp-skill-set/
├── README.md                          ← You are here
├── INSTALL.md                         ← Detailed install guide
├── commands/kfsp/                     ← 16 skill files
│   ├── help.md                        ← /kfsp:help (start here)
│   ├── product-health.md              ← T5: PM's 8-dimension health dashboard
│   ├── doc-pilot.md                   ← T5: Documentation autopilot
│   ├── pre-mortem.md                  ← T5: Risk forecast
│   ├── ux-audit.md                    ← T4: UX heuristic evaluation (for non-UX PMs)
│   ├── ux-parity.md                   ← T4: Design-code parity
│   ├── surface-test.md                ← T3: 4-surface contract testing
│   ├── sentinel.md                    ← T3: Regression detection
│   ├── release-gate.md                ← T3: Pre-release gate
│   ├── sweep.md                       ← T2: Blast radius analysis
│   ├── pre-commit.md                  ← T2: Commit validation
│   ├── sync-check.md                  ← T2: Source sync
│   ├── post-phase.md                  ← T2: Phase completion
│   ├── guard.md                       ← T1: Continuous guardian
│   ├── dev-journal.md                 ← T1-T5: Developer journal
│   └── incident-review.md            ← Cross: Post-incident
├── references/                        ← Reference documents
│   └── INTERACTION_SURFACE_MAP.md     ← 4-surface contract reference
└── examples/                          ← Example outputs (coming soon)
```

---

## About the Creator

Hi, I'm **Thanh** — a non-tech product person who built this skill set with AI.

I studied **Business Administration**, not Computer Science. From 2019 to early 2025, my world was **content marketing**, Excel/Google Sheets automation, and building data pipelines from raw orders to dashboards — all self-taught. In late 2024, I discovered **n8n** and started automating workflows properly. Then AI changed everything.

I went from asking the most naive questions to Claude, to building a complete mobile app ([KungFu Stocks Pro](https://kungfustockspro.com)) with AI as my co-developer. Along the way, I realized: **the harder part isn't building — it's making sure what you build actually works, stays consistent, and doesn't break silently.**

This skill set was born from that realization. I'm not a developer, tester, or QA engineer. I'm a **Product Manager** (still working on my Google PM Certificate when I created this). But that's exactly why these skills exist — they're the "insurance policy" that lets me focus on what I do best: **business, marketing, and customer care** — while being confident that the product side is covered.

Every skill here is something I needed, couldn't find anywhere else, and built by articulating my thinking to AI until it became a reusable tool. If you find them useful, that means something to me. I'm someone who values connection and meaning — poetry, music, human stories. Knowing that my work helps someone else makes this journey worth more.

**I will keep developing new skills** — for myself first, and for anyone who finds them valuable.

---

## License

MIT — Use freely, adapt for your projects.

---

## Credits

Created by **[Thanh Nguyen](https://github.com/nguyenthanhcllhp)** — Product Manager, non-tech builder, AI-first practitioner.

Built for [KungFu Stocks Pro](https://kungfustockspro.com) with [Claude AI](https://claude.ai) as co-developer.

Inspired by:
- [GSD (Get Shit Done)](https://github.com/get-shit-done/gsd) — planning & execution framework
- [Ralph Loop](https://github.com/anthropics/ralph-loop) — autonomous iteration
- [Product-Manager-Skills](https://github.com/deanpeters/Product-Manager-Skills) — PM skill framework
