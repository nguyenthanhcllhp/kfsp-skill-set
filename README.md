# 🥋 KFSP Skill Set v3.2

**17 production-grade skills for agent-driven product development.**

Build apps from zero to store with confidence — for both human teams and AI agent swarms.

## What's New in v3.2

### v3.2 (2026-03-15)
- **ORCHESTRATION.md** — Master integration guide: KFSP + GSD + Ralph workflow
- **Phase-as-Project Rule** — Mỗi phase = 1 project với 6 chiều bắt buộc
- **Dev Journal Compliance** — 8-layer enforcement across all skills
- **Number Formatting Rule** — Dấu `.` thập phân, `,` phần nghìn (financial app)

### v3.0 (2026-03-13)
- +3 skills: `test-brief`, `bug-log`, `build-verify` → PM testing workflow
- +1 skill: `session-start` → Session initialization with drift detection
- **Feature Completeness Rule** — 6 dimensions per feature
- **Vietnamese Diacritics Rule** — All user-facing text must have diacritics
- **Data Accuracy Rule** — Zero tolerance for "approximate" in financial data
- **Simulator Testing Rule** — Agent must ask PM to navigate, take screenshots
- **Git Remote Safety Rule** — Never push without explicit permission

### v2.0
- Converted skills from inline specs to **standalone agent** format (`.claude/agents/`)
- Agent subprocess isolation, parallel execution, auto-trigger by orchestrators

### Package Structure

```
kfsp-skill-set/
├── README.md                          ← You are here
├── INSTALL.md                         ← Installation + verification guide
├── ORCHESTRATION.md                   ← GSD + Ralph integration (BẮT BUỘC đọc)
├── commands/kfsp/                     ← 20 slash command files (/kfsp:*)
│   ├── help.md
│   ├── guard.md                       ← T1: Continuous guardian
│   ├── dev-journal.md                 ← T1: Developer journal
│   ├── incident-review.md             ← T1: Post-incident review
│   ├── session-start.md               ← T1: Session initialization
│   ├── sweep.md                       ← T2: Blast radius analysis
│   ├── pre-commit.md                  ← T2: Commit validation
│   ├── sync-check.md                  ← T2: Source sync
│   ├── post-phase.md                  ← T2: Phase completion
│   ├── surface-test.md                ← T3: 4-surface contract testing
│   ├── sentinel.md                    ← T3: Regression detection
│   ├── release-gate.md                ← T3: Pre-release gate
│   ├── test-brief.md                  ← T3: PM test checklist (HTML)
│   ├── bug-log.md                     ← T3: Bug tracking
│   ├── build-verify.md                ← T3: Post-build verification gate
│   ├── ux-parity.md                   ← T4: Design-code parity
│   ├── ux-audit.md                    ← T4: UX heuristic evaluation
│   ├── doc-pilot.md                   ← T4-T5: Documentation autopilot
│   ├── pre-mortem.md                  ← T5: Risk forecast
│   └── product-health.md              ← T5: PM health dashboard
├── agents/kfsp/                       ← 15 agent files (kfsp-*.md)
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
│  test-brief · bug-log · build-verify                    │
├─────────────────────────────────────────────────────────┤
│  T2 🔧 CODE INTEGRITY                                    │
│  sweep · pre-commit · sync-check · post-phase           │
├─────────────────────────────────────────────────────────┤
│  T1 🛡️ AGENT SAFETY                                      │
│  guard · dev-journal · incident-review · session-start  │
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
- **Shortcuts** — agent takes approximations in financial data

KFSP Skill Set provides **5 layers of protection** across **4 interaction surfaces** (user-facing, backend, third-party services, internal components).

---

## Quick Start

### Install (one command)

```bash
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git /tmp/kfsp-skill-set \
  && mkdir -p ~/.claude/commands/kfsp/ \
  && cp /tmp/kfsp-skill-set/commands/kfsp/*.md ~/.claude/commands/kfsp/ \
  && mkdir -p ~/.claude/agents/ \
  && cp /tmp/kfsp-skill-set/agents/kfsp/*.md ~/.claude/agents/ 2>/dev/null \
  && echo "✅ KFSP Skill Set v3.2 installed ($(ls ~/.claude/commands/kfsp/*.md | wc -l | tr -d ' ') commands)."
```

### Verify

```bash
# Check slash commands (should show 20 files)
ls ~/.claude/commands/kfsp/ | wc -l

# In Claude Code — type /kfsp: and see dropdown
/kfsp:help
```

See `INSTALL.md` for detailed installation + verification guide.

### Use

```bash
/kfsp:session-start                                # Start every session here
/kfsp:pre-mortem "Add user authentication"         # Forecast risks BEFORE building
# ... write code ...
/kfsp:sweep auth_controller.dart                   # Check blast radius AFTER changes
/kfsp:ux-parity login                              # Verify design match
/kfsp:pre-commit                                   # Validate BEFORE committing
/kfsp:build-verify                                 # Gate BEFORE handing to PM
/kfsp:test-brief                                   # Generate PM test checklist (HTML)
/kfsp:doc-pilot --what-changed                     # Track doc updates needed
```

---

## All 17 Skills

### T5 🎯 Business Impact + T4 👤 User Experience

| Skill | Command | What it does |
|-------|---------|-------------|
| Product Health | `/kfsp:product-health` | PM's 8-dimension dashboard. Score /100. |
| Doc Pilot | `/kfsp:doc-pilot` | Tracks which docs need updating. Ensures user guide ready before store. |
| Pre-Mortem | `/kfsp:pre-mortem` | Imagines feature already failed. Risks across 4 surfaces. 6-dimension check. |
| UX Audit | `/kfsp:ux-audit` | Nielsen 10 heuristics + accessibility + domain checks. |
| UX Parity | `/kfsp:ux-parity` | Compares actual UI vs design specs. Finds hardcoded colors, spacing drift. |

### T3 🧪 Quality Assurance

| Skill | Command | What it does |
|-------|---------|-------------|
| Surface Test | `/kfsp:surface-test` | Tests contracts across 4 interaction surfaces. |
| Sentinel | `/kfsp:sentinel` | Baseline before changes, compare after. Detects regressions. |
| Release Gate | `/kfsp:release-gate` | 12 gates, score /100, Go/No-Go. Charter + journal check. |
| Test Brief | `/kfsp:test-brief` | Generates HTML test checklist for PM after each build. |
| Bug Log | `/kfsp:bug-log` | Structured bug logging from PM testing. Track resolution. |
| Build Verify | `/kfsp:build-verify` | 14 automated checks. Gate before handing build to PM. |

### T2 🔧 Code Integrity

| Skill | Command | What it does |
|-------|---------|-------------|
| Sweep | `/kfsp:sweep` | Blast radius after code changes. Tokens, routes, providers, tests, docs. |
| Pre-Commit | `/kfsp:pre-commit` | No hardcoded values, no secrets, no broken imports. Journal check. |
| Sync Check | `/kfsp:sync-check` | All source copies in sync, docs match code. |
| Post-Phase | `/kfsp:post-phase` | Phase completion: charter 6/6, journal compliance, docs, sync. |

### T1 🛡️ Agent Safety

| Skill | Command | What it does |
|-------|---------|-------------|
| Guard | `/kfsp:guard` | Health score /100. 10 checks: structure, design, cross-cutting, charter, journal. |
| Dev Journal | `/kfsp:dev-journal` | Records decisions/incidents as searchable precedents. 10 entry types. |
| Incident Review | `/kfsp:incident-review` | 5 Whys root cause, blast radius, prevention plan. |
| Session Start | `/kfsp:session-start` | Git check, tool detection, code-doc drift, charter/journal status. |

---

## Enforcement Rules (v3.0+)

These rules are embedded across multiple skills for automatic enforcement:

| Rule | Enforced by | Since |
|------|-------------|-------|
| **Feature Completeness** (6 dimensions) | pre-mortem, build-verify, guard | v3.0 |
| **Vietnamese Diacritics** | build-verify, guard, pre-commit | v3.0 |
| **Data Accuracy** (zero tolerance) | All skills — CLAUDE.md level | v3.0 |
| **Simulator Testing** (ask PM) | build-verify, test-brief | v3.0 |
| **Git Remote Safety** | pre-commit, guard, session-start | v3.0 |
| **Phase-as-Project** (6 dimensions) | post-phase, guard, session-start, release-gate | v3.2 |
| **Dev Journal Compliance** | 8 skills — full chain | v3.2 |
| **Number Formatting** | build-verify, pre-commit | v3.2 |
| **Test Case Coverage** | build-verify, pre-commit, release-gate | v3.2 |

### 8-Layer Enforcement Chain

```
session-start → pre-mortem → pre-commit → build-verify →
test-brief → post-phase → guard → release-gate
```

Every skill in the chain checks for charter, journal, and rule compliance.

---

## Workflows

### Auto-Trigger Table

| When | Run |
|------|-----|
| Session starts | `session-start` (git check, drift detect, tool detect) |
| Before feature | `pre-mortem` → plan 6 dimensions |
| After code change | `sweep` (blast radius) |
| After UI change | `ux-parity` (design match) |
| Before commit | `pre-commit` (validation gate) |
| After build | `build-verify` → `test-brief` → PM test |
| PM finds bug | `bug-log --log` → fix → `bug-log --resolve` |
| Phase complete | `post-phase N` → `guard` → `sync-check` |
| Before release | `release-gate` → `surface-test` → `doc-pilot --checklist` |
| Decision made | `dev-journal --log decision` |
| Incident | `incident-review` → `dev-journal --log incident` |

### Chain Workflows

```
New Feature:   pre-mortem → [develop] → sweep → ux-parity → build-verify → test-brief
Bug Fix:       [fix] → sweep → pre-commit → commit → bug-log --resolve
Release:       sentinel --baseline → [develop] → sentinel --compare → release-gate
Phase Close:   post-phase N → guard → sync-check → doc-pilot --status
Session:       session-start → [check drift] → [fix if needed] → start work
```

### Doc-First Response Rule

> **Every response** from the agent to the user **MUST** include:
>
> ```
> 📝 Docs đã cập nhật:
> - file_1.md — what changed
>
> 🥋 Skills đã chạy:
> - sweep — [result]
> - guard — [score]
> ```
>
> If no docs affected: `📝 Không có doc nào bị ảnh hưởng`
> If no code change: `🥋 Không có code change — không cần chạy skill`

---

## Companion Tools — GSD + Ralph Integration

| Tool | Role | How it pairs with KFSP |
|------|------|----------------------|
| [GSD](https://github.com/gsd-build/get-shit-done) | 🧠 Bộ não — Planning & execution | GSD handles *what to build*, KFSP ensures *it's built safely* |
| [Ralph Loop](https://github.com/frankbria/ralph-claude-code) | 🦶 Đôi chân — Autonomous iteration | Ralph handles *execution loops*, KFSP guard verifies after each loop |
| KFSP | 🥋 Áo giáp — Protection | KFSP wraps around GSD/Ralph as quality shield |

> **📖 Read [`ORCHESTRATION.md`](ORCHESTRATION.md)** for full integration details: detection scripts, spawn rules, enforcement layers, 5 workflow templates.

`/kfsp:session-start` auto-detects GSD + Ralph and selects the appropriate workflow mode:
- **Full Orchestration**: KFSP + GSD + Ralph
- **GSD + KFSP**: Planning + protection (no auto-loop)
- **KFSP Only**: Protection only
- **Unprotected**: ⚠️ Warning — install KFSP first

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

## Agent Architecture

### Distributed Model — No Single Orchestrator

KFSP uses a **distributed model** where different systems spawn KFSP agents:

```
┌─────────────────────────────────────────────────────────┐
│                    WHO SPAWNS KFSP AGENTS?               │
├─────────────────────────────────────────────────────────┤
│  GSD Executor  ───→ kfsp-guard after each task          │
│  Ralph Loop    ───→ kfsp-guard after each iteration     │
│  Human (/kfsp) ───→ Slash command runs inline           │
│  Claude Code   ───→ Spawns agents in parallel           │
└─────────────────────────────────────────────────────────┘
```

**Key insight:** KFSP agents are **read-only analysts** — they READ code, GREP patterns, and PRODUCE reports. They do NOT write code. The orchestrator writes code and calls KFSP agents to verify.

### Build Report Rule

Every build session MUST produce a **build report** with 9 sections:
1. Goals (from Project Charter)
2. Deliverables (status)
3. Success Criteria (results)
4. Files Changed
5. Technical Decisions
6. Test Checklist for PM
7. Skills Run (with results)
8. Known Issues
9. Next Steps

Saved to: `docs/build_reports/YYYY-MM-DD_P{N}_{description}.md`

---

## Adapting for Your Project

KFSP skills are designed to be project-agnostic. To adapt:

1. **Source paths** — Edit path detection in each skill to match your project structure
2. **Design tokens** — Edit `ux-parity.md` to point to your token files
3. **Doc registry** — Edit `doc-pilot.md` to map your documentation structure
4. **Feature list** — Edit `guard.md` to match your expected folder structure
5. **Rules** — Add project-specific rules to CLAUDE.md (data accuracy, language, etc.)

Skills that work without changes: `pre-mortem`, `dev-journal`, `incident-review`, `sentinel`, `release-gate`, `session-start`

---

## Disclaimer

**This skill set is NOT created by a domain expert.** It is distilled from personal thinking, experience, and intuition of a non-tech Product Manager building a real product with AI. The ideas, checklists, and frameworks here are generalized from a Vietnamese stock analysis app — if something feels unfamiliar, try asking AI to translate/adapt it to your context.

**You are fully responsible for your own actions.** Whether you use these skills, modify them, or ignore them — every decision and its consequences belong to the person who acts.

**This is a snapshot of work-in-progress.** Skills are updated as the product evolves.

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v3.2 | 2026-03-15 | ORCHESTRATION.md, Phase-as-Project, Dev Journal Compliance, Number Formatting |
| v3.0 | 2026-03-13 | +4 skills (17 total), 6 enforcement rules, simulator testing, git safety |
| v2.0 | 2026-03-10 | Agent format, distributed orchestration, parallel execution |
| v1.0 | 2026-03-08 | Initial 13 slash command specs |

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
