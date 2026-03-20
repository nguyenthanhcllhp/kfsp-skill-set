# KFSP Skill Set — Architecture Guide

> How 20 skills work together to run any project — from product development to marketing campaigns.
>
> **Version:** 1.0 | **Updated:** 2026-03-20 | **Owner:** Thanh Nguyen

---

## TABLE OF CONTENTS

1. [Philosophy: Everything is a Project](#1-philosophy-everything-is-a-project)
2. [The 5-Tier Protection Model](#2-the-5-tier-protection-model)
3. [6 Stages of a Project](#3-6-stages-of-a-project)
4. [Test & QA Architecture](#4-test--qa-architecture)
5. [The Build Loop (Controlled)](#5-the-build-loop-controlled)
6. [Crosscut Skills (Event-Triggered)](#6-crosscut-skills-event-triggered)
7. [20 Skills — Role, Output, Cross-Domain Examples](#7-20-skills--role-output-cross-domain-examples)
8. [Cross-Domain Translation Table](#8-cross-domain-translation-table)
9. [Agent Architecture](#9-agent-architecture)
10. [The 4 Interaction Surfaces](#10-the-4-interaction-surfaces)
11. [Modular Training Guide](#11-modular-training-guide)

---

## 1. Philosophy: Everything is a Project

Whether you're building an app, running a marketing campaign, or organizing an event — any work with a goal is a **project**. And every project, no matter how big or small, needs to answer the same 6 questions:

```
1. What do we want to achieve?          (Goals)
2. What's in scope, what's not?         (Scope)
3. What do we deliver when done?        (Deliverables)
4. How do we know it's "done right"?    (Verification Criteria)
5. Who's involved?                      (Stakeholders)
6. What do we need?                     (Resources)
```

### The Fractal Principle

These questions repeat at EVERY level:

```
Project (Roadmap)
  └── Phase (a sub-project — same 6 questions)
        └── Task (a smaller project — same 6 questions)
              └── Sub-task (same pattern)
```

**Product example:**
- Roadmap → Phase "Market Screen" → Task "Build Heatmap" → Sub-task "Gradient color logic"

**Marketing example:**
- Campaign plan → Phase "Launch Week" → Task "Email sequence" → Sub-task "Subject line A/B test"

**HR example:**
- Hiring plan → Phase "Q2 Recruitment" → Task "Interview process" → Sub-task "Technical assessment"

### Why this matters

Skills protect at EVERY level. The same `pre-mortem` (risk forecast) works for:
- A 6-month product roadmap
- A single Sprint task
- A marketing campaign
- An office move

The framework is the same. Only the vocabulary changes.

---

## 2. The 5-Tier Protection Model

```
         ┌─────────────────────────────────────────┐
    T5   │  Is this VALUABLE?                       │  ← Business Impact
         ├─────────────────────────────────────────┤
    T4   │  Can users actually USE it?              │  ← User Experience
         ├─────────────────────────────────────────┤
    T3   │  Does the deliverable MATCH the spec?    │  ← Verification
         ├─────────────────────────────────────────┤
    T2   │  Are assets/code INTACT?                 │  ← Integrity
         ├─────────────────────────────────────────┤
    T1   │  Is the team IN CONTROL?                 │  ← Safety
         └─────────────────────────────────────────┘
```

**Read bottom-up:** If T1 fails (lost control), T2-T5 are meaningless. If T2 fails (broken code), you can't verify (T3). If T3 fails (wrong deliverable), UX is bad (T4) and there's no value (T5).

### What each tier prevents

| Tier | Risk | Without it | With it |
|------|------|-----------|---------|
| **T1 Safety** | Lost control, forgotten decisions, repeated mistakes | Agent goes rogue, same bugs return | Every decision tracked, anomalies caught early |
| **T2 Integrity** | Broken imports, out-of-sync copies, hardcoded values | Build fails, docs lie, copies diverge | Clean commits, synced sources, valid code |
| **T3 Verification** | Deliverable doesn't match spec | PM discovers gaps manually, late | Automated spec compliance, structured PM testing |
| **T4 UX** | Design drift, accessibility gaps | Users struggle, design intent lost | Measured parity, heuristic-based quality |
| **T5 Business** | Wrong priorities, missing docs, unforeseen risks | Ship wrong things, no user guide, surprises | Risk forecast, docs ready, health dashboard |

---

## 3. 6 Stages of a Project

Every project goes through 6 stages. Skills protect at each stage.

```
🚀 START → 📐 PLAN → 🔨 DEV LOOP → 📦 BUILD & TEST ⇄ (loop) → 🏁 CLOSE → ✅ SHIP
```

### Stage 1: START — "What's the current state?"

**Purpose:** Ensure the team has accurate context BEFORE doing anything.

| Skill | Tier | Does what |
|-------|------|----------|
| `session-start` | T1 | Checks git status, code-doc drift, tool availability. Reports what changed since last session. |

### Stage 2: PLAN — "What are we building, what could go wrong?"

**Purpose:** Forecast risks + generate test cases + expected changelog FROM THE SPEC, before any code is written. This is where verification begins.

| Skill | Tier | Does what |
|-------|------|----------|
| `pre-mortem` | T5 | "If this fails completely — why?" Risk matrix across 4 surfaces + mitigation plan + Go/No-Go. |
| `dev-journal` | T1 | Records the decision and its reasoning. Searchable when similar situations arise later. |

**Critical output from this stage:**
- Test cases (derived from spec) — written as actual test files BEFORE code
- Expected changelog — what files should change and how
- Acceptance criteria — how PM will verify each deliverable

### Stage 3: DEV LOOP — "Code → Check → Commit, repeat"

**Purpose:** Write code, verify it doesn't break anything, commit clean.

| Skill | Tier | Does what |
|-------|------|----------|
| `sweep` | T2 | After every change: scans blast radius — what else is affected? |
| `ux-parity` | T4 | After UI changes: compares code vs approved design (Figma/prototype). Parity score %. |
| `pre-commit` | T2 | Before every commit: no hardcoded values, no secrets, imports valid, tests pass. |

### Stage 4: BUILD & TEST — "AI checks automatically, PM verifies on device"

**Purpose:** Bridge the gap between spec and deliverable. Compare actual vs expected.

| Skill | Tier | Does what |
|-------|------|----------|
| `build-verify` | T3 | AI runs 16 automated checks + compares actual changelog vs expected changelog (from Plan stage). |
| `test-brief` | T3 | Generates PM test checklist — items DERIVED FROM SPEC, not from memory. HTML with pass/fail buttons. |
| `bug-log` | T3 | When PM finds issues: structured logging with severity + tracking until resolution. |

**The spec bridge:**
```
Plan (expected)  ──→  Build (actual)  ──→  Compare  ──→  Match? Ship. Deviate? Fix.
```

### Stage 5: CLOSE — "Is everything complete before we ship?"

**Purpose:** Ensure nothing is left behind. Docs, sync, contracts, gate.

| Skill | Tier | Does what |
|-------|------|----------|
| `post-phase` | T2 | 6-step completion checklist: verify, regression, docs, sync, sweep, archive. |
| `sync-check` | T2 | All copies of the same source match? No drift between code, docs, design? |
| `doc-pilot` | T5 | Which docs need updating? Is the user guide ready? |
| `release-gate` | T3 | Final gate: 12-16 checks, score /100, Go/No-Go decision. |
| `surface-test` | T3 | Tests "contracts" between 4 interaction surfaces. |
| `product-health` | T5 | 8-dimension dashboard for PM: UX, tech, docs, business, legal, ops, data, growth. |

### Stage 6: SHIP ✅

All gates passed, all docs updated, all tests green. Ship with confidence.

---

## 4. Test & QA Architecture

### The core problem

Without a spec-to-verification pipeline, testing is:
- Based on "what the developer remembers" (not the spec)
- Discovered by PM manually (expensive, demoralizing)
- Not tracked systematically (same bugs return)

### The solution: Spec is the single source of truth

```
                SPEC (source of truth)
                    ↓                ↓
         ┌──────────┘                └──────────┐
         ↓                                      ↓
    Test Cases                           Expected Changelog
    (before code)                        (what should change)
         ↓                                      ↓
    ┌─────────┐                          ┌──────────────┐
    │   DEV   │ ────── code ──────────→  │    BUILD     │
    │ (TDD)   │                          │              │
    └─────────┘                          └──────┬───────┘
                                                │
                                         Actual Changelog
                                         (from git diff)
                                                │
                                    ┌───────────┼───────────┐
                                    ↓                       ↓
                              COMPARE                  Test Brief
                         actual vs expected          (from SPEC items)
                                    ↓                       ↓
                              Compliance              PM Verify
                              Report                  on Device
```

### Who checks what? (AI vs Human)

| Check type | AI automated | PM manual | When |
|-----------|-------------|----------|------|
| Unit tests (logic, format, parser) | ✅ Write + run | — | Before code (TDD) |
| Static analysis (compile, imports) | ✅ flutter analyze | — | Before commit |
| Design token compliance | ✅ Scan hardcoded values | — | build-verify |
| Spec compliance | ✅ Actual vs expected changelog | — | build-verify |
| Dark mode visual | ⚠️ Scan code patterns | ✅ Visual on device | build-verify + PM test |
| Language/locale check | ✅ Scan strings | ✅ Read on device | build-verify + PM test |
| Number formatting | ✅ Scan formatters | ✅ View on device | build-verify + PM test |
| UI layout / visual | ⚠️ ux-parity (code scan) | ✅ Compare with design | PM test |
| Navigation / user flow | ❌ Can't tap simulator | ✅ Tap through screens | PM test |
| Real data / live API | ⚠️ Check lifecycle code | ✅ View during market hours | PM test |
| Regression | ✅ sentinel (code-level) | ✅ Regression items in test-brief | After refactor |
| Performance | ⚠️ Build time check | ✅ Scroll smoothness, load time | Before release |

### 3 levels of test brief

| Level | Scope | Items | When | PM effort |
|-------|-------|-------|------|----------|
| **Build test** | Only changes in this build | 5-10 | Every build | 5-10 min |
| **Phase test** | All items in the phase | 20-30 | End of phase | 30-60 min |
| **Release test** | Full regression, all features | 100+ | Before release | 2-4 hours |

**Smart scoping:** Previously passed items show as grayed out. PM only tests NEW or CHANGED items. Previously failed items show as highlighted.

### Bug flow

```
PM taps "Fail" → bug-log creates BUG-YYYYMMDD-NN → AI reads → AI fixes → regression test
→ re-build → test-brief updates (only fixed items highlighted) → PM re-tests 1-2 items → Pass → Close
```

**Loop limit:** If the same issue loops > 3 times → mandatory `incident-review` (root cause analysis).

---

## 5. The Build Loop (Controlled)

```
                    ┌────────────────────────────┐
                    │                            │
     📐 PLAN ────→ 🔨 DEV ────→ 📦 BUILD ─────→ 🏁 CLOSE
                    ↑                  │
                    │    ┌─────────────┘
                    │    │ FAIL or BUG
                    │    ↓
                    └── 🔄 FIX (with bug ID or fail reason)
```

**Controls:**
1. Every loop-back must have a **bug ID** (from `bug-log`) or **fail reason** (from `build-verify`)
2. Re-build only tests **items that were fixed** + **regression check** — not everything
3. Loop > 3 times on same issue → `incident-review` (root cause analysis) is mandatory
4. Each loop is tracked: Loop 1/3, Loop 2/3, Loop 3/3 → escalation

---

## 6. Crosscut Skills (Event-Triggered)

These skills don't belong to a specific stage. They're triggered by events in the main flow.

| Skill | Tier | Triggered when | What it does |
|-------|------|---------------|-------------|
| `guard` | T1 | After every task, periodically, after large changes | Health score /100. If < 70 → stop and investigate before continuing. |
| `dev-journal` | T1 | Every decision, incident, or learning | Records as searchable "precedent". Like case law — look up when similar situation arises. |
| `sentinel` | T3 | Before/after risky refactors | Takes "photo BEFORE" → change → compares "photo AFTER" → detects regressions. |
| `incident-review` | T1 | Serious incidents, build loop > 3x, wrong data | 5 Whys root cause analysis + blast radius + prevention plan. |
| `ux-audit` | T4 | Periodically, before release, after redesign | 10 Nielsen heuristic scores /30 + accessibility + domain-specific checks. |
| `help` | T1 | When you forget which skill does what | Shows all 20 skills + workflows + setup guide. |

---

## 7. 20 Skills — Role, Output, Cross-Domain Examples

### 🚀 START (1 skill)

#### `session-start` — The morning check

| | |
|---|---|
| **Tier** | T1 Safety |
| **Does** | Checks current state before work begins: git, docs, memory, tools |
| **Output** | Report: git status + drift detected + tools available + next steps |
| **When** | Start of EVERY session |
| **Product** | Git status, code-doc drift, memory update, tool detection |
| **Marketing** | Campaign status, budget snapshot, pending approvals, KPI check |
| **General** | Current state + what changed since last time + ready to work? |

---

### 📐 PLAN (2 skills)

#### `pre-mortem` — The risk forecast

| | |
|---|---|
| **Tier** | T5 Business |
| **Does** | "If this fails completely — why?" Asked BEFORE you start, not after |
| **Output** | Risk matrix + mitigation plan + **test cases** + **expected changelog** + Go/No-Go |
| **When** | Before every new feature, phase, or task |
| **Product** | User impact, API break, 3rd party risk, component cascade, financial data accuracy |
| **Marketing** | Wrong audience, bad timing, weak creative, channel mismatch, budget waste |
| **General** | Forecast risks across 4 interaction surfaces + plan mitigations |

#### `dev-journal` — The long-term memory

| | |
|---|---|
| **Tier** | T1 Safety |
| **Does** | Records decisions, incidents, and learnings as searchable "precedents" |
| **Output** | Journal entries + index + tags + weekly digest |
| **When** | Every decision (Plan stage), every incident (crosscut), every learning |
| **Product** | Technical decisions, bug postmortems, architecture choices |
| **Marketing** | Campaign learnings, A/B test results, audience insights, vendor evaluations |
| **General** | "Case law" — look up when you encounter a similar situation |

---

### 🔨 DEV LOOP (3 skills)

#### `sweep` — The blast radius scanner

| | |
|---|---|
| **Tier** | T2 Integrity |
| **Does** | After every change: what else is affected? Scans dependencies, imports, routes, tests, docs |
| **Output** | Files impacted + severity (critical/warn/info) + fix suggestions |
| **When** | After EVERY code/asset change |
| **Product** | Imports, routes, providers, design tokens, tests, docs related to changed file |
| **Marketing** | Change brand color → check website, social profiles, email templates, print, partner assets |
| **General** | Impact analysis: change 1 thing → what else breaks? |

#### `ux-parity` — The design truth-meter

| | |
|---|---|
| **Tier** | T4 UX |
| **Does** | Compares actual deliverable vs approved design. Measures "faithfulness" to spec |
| **Output** | Parity score % + violations list (colors, spacing, typography, layout) |
| **When** | After every UI/creative change |
| **Product** | Flutter code vs Figma design — spacing, colors, typography drift |
| **Marketing** | Final creative vs approved mockup, delivered campaign vs approved brief |
| **General** | How "true" is the deliverable to the approved spec? |

#### `pre-commit` — The last gate before locking in

| | |
|---|---|
| **Tier** | T2 Integrity |
| **Does** | Validation gate before committing. Catches basic errors before they cause damage |
| **Output** | Pass/fail per check + violations detail |
| **When** | Before EVERY commit/submit |
| **Product** | No hardcoded colors, no secrets, valid imports, tests pass, journal entries exist |
| **Marketing** | No typos in copy, correct links, proper UTM tags, approved by legal, budget allocated |
| **General** | Validation gate before locking in changes |

---

### 📦 BUILD & TEST (3 skills)

#### `build-verify` — The AI pre-check

| | |
|---|---|
| **Tier** | T3 Verification |
| **Does** | AI runs 16 automated checks + compares actual changelog vs expected changelog from spec |
| **Output** | 16 gates pass/fail + **Spec Compliance report** (actual vs expected) + blockers |
| **When** | After EVERY build, BEFORE handing to PM/reviewer |
| **Product** | flutter analyze, tests, imports, design tokens, dark mode, locale, numbers + spec match |
| **Marketing** | Spell check, link validation, image quality, brand match, responsive preview + brief match |
| **General** | Automated checks + spec compliance. Filter basic errors so humans don't waste time |

#### `test-brief` — The human verification guide

| | |
|---|---|
| **Tier** | T3 Verification |
| **Does** | Creates a clear checklist FOR THE HUMAN VERIFIER. Items derived FROM SPEC, not from memory |
| **Output** | Interactive HTML file: items + expected behavior + Pass/Fail/Skip buttons + bug form |
| **When** | After build-verify passes |
| **Product** | PM tests on device: "Tap X → expect to see Y". Dark mode, Vietnamese text, navigation |
| **Marketing** | Reviewer checks: creative matches brief? CTA correct? Landing page loads? Tracking works? |
| **General** | Clear verification guide traceable to the original spec. Human only tests what AI can't |

#### `bug-log` — The issue tracker

| | |
|---|---|
| **Tier** | T3 Verification |
| **Does** | Structured issue logging with tracking until resolution. Triggers loop back to Dev |
| **Output** | BUG_LOG.md: BUG-YYYYMMDD-NN + severity + status + resolution stats |
| **When** | When PM/reviewer finds an issue |
| **Product** | Bug: steps to reproduce, severity, screenshots, resolution |
| **Marketing** | Issue: creative revision request, copy change, targeting adjustment, budget reallocation |
| **General** | Track from discovery → fix → verify → close. Each loop needs a bug ID |

---

### 🏁 CLOSE & RELEASE (6 skills)

#### `post-phase` — The completion checklist

| | |
|---|---|
| **Tier** | T2 Integrity |
| **Does** | End-of-phase checklist: deliverables complete? Docs updated? Source synced? Regression passed? |
| **Output** | 6-step completion report + project charter check + journal compliance |
| **When** | End of every phase/sprint/campaign flight |
| **Product** | Code synced, docs updated, regression checked, phase archived |
| **Marketing** | Results documented, learnings captured, assets archived, budget reconciled |

#### `sync-check` — The copy matcher

| | |
|---|---|
| **Tier** | T2 Integrity |
| **Does** | Multiple places store the same information — are they all identical? |
| **Output** | Drift list between copies + fix commands |
| **When** | End of phase, or when drift is suspected |
| **Product** | Source code copies, docs vs code, design files vs implementation |
| **Marketing** | Messaging across channels, brand asset versions, translation consistency |

#### `doc-pilot` — The documentation autopilot

| | |
|---|---|
| **Tier** | T5 Business |
| **Does** | Tracks which docs need updating. Ensures user guides are ready before public launch |
| **Output** | Outdated docs list + feature coverage + update priorities + user guide gap analysis |
| **When** | End of phase, before release, after major changes |
| **Product** | User guide ready before store submission? API docs current? Feature specs match code? |
| **Marketing** | Campaign report ready for stakeholders? Content calendar updated? Knowledge base current? |

#### `release-gate` — The final gate

| | |
|---|---|
| **Tier** | T3 Verification |
| **Does** | Comprehensive pre-release check: 12-16 gates, score /100, Go/No-Go decision |
| **Output** | Score /100 + gate results + READY / CAUTION / NOT READY |
| **When** | Before every public release/launch |
| **Product** | Performance, security, accessibility, store compliance, all tests pass |
| **Marketing** | Legal review, brand compliance, tracking setup, budget approved, launch timeline confirmed |

#### `surface-test` — The contract checker

| | |
|---|---|
| **Tier** | T3 Verification |
| **Does** | Tests "contracts" between interaction surfaces — are components communicating correctly? |
| **Output** | 4 surfaces health + contract violations + missing contracts |
| **When** | Before release, after API/integration changes |
| **Product** | User↔App, App↔Server, App↔3rd Party, Component↔Component |
| **Marketing** | Brand↔Customer, Marketing↔Sales, Marketing↔Partners, Channel↔Channel |

#### `product-health` — The PM dashboard

| | |
|---|---|
| **Tier** | T5 Business |
| **Does** | 8-dimension health dashboard for project leadership. Overall score /80 |
| **Output** | 8 scores (UX, Tech, Docs, Business, Legal, Ops, Data, Growth) + gap analysis |
| **When** | Periodically (weekly/monthly), before milestone reviews |
| **Product** | UX quality, tech debt, docs coverage, store readiness, compliance |
| **Marketing** | Reach, engagement, conversion, ROI, brand sentiment, team bandwidth, budget utilization |

---

### 🔄 CROSSCUT (5+1 skills)

#### `guard` — The health doctor

| | |
|---|---|
| **Tier** | T1 Safety |
| **Does** | Scores project health /100. If < 70 → stop, investigate before continuing |
| **Output** | Health score /100 + breakdown (5 categories) + warnings + trends vs previous |
| **When** | After every task, periodically, after large changes |

#### `sentinel` — The before/after camera

| | |
|---|---|
| **Tier** | T3 Verification |
| **Does** | Takes "snapshot BEFORE" → change happens → compares "snapshot AFTER" → finds regressions |
| **Output** | Baseline snapshot, or regression report (before vs after diff) |
| **When** | Before/after large refactors or risky changes |

#### `incident-review` — The root cause analyst

| | |
|---|---|
| **Tier** | T1 Safety |
| **Does** | After serious incidents: 5 Whys root cause, blast radius on 4 surfaces, prevention plan |
| **Output** | Timeline + root cause + fixes (immediate + permanent) + prevention checklist + lesson |
| **When** | Serious incidents, build loop > 3x on same issue, wrong financial data |

#### `ux-audit` — The UX evaluator

| | |
|---|---|
| **Tier** | T4 UX |
| **Does** | Evaluates user experience against 10 proven heuristics (Nielsen) + accessibility |
| **Output** | 10 heuristic scores /3 each + total /30 + accessibility audit + recommendations |
| **When** | Periodically, before release, after redesign |

#### `help` — The quick reference

| | |
|---|---|
| **Tier** | T1 Safety |
| **Does** | Shows all 20 skills + workflow chains + setup guide |
| **When** | When you forget which skill does what |

---

## 8. Cross-Domain Translation Table

The framework is domain-agnostic. Only the vocabulary changes.

| Concept | Product Dev | Marketing | Sales | HR | General |
|---------|------------|-----------|-------|----|---------|
| Spec/Brief | Feature spec, PRD | Creative brief, Campaign brief | Sales playbook, Pitch deck | Job description, Process doc | Project brief |
| Build | App build, Deploy | Campaign launch, Content publish | Proposal sent, Demo delivered | Candidate pipeline, Training session | Deliverable |
| Test | Unit/Widget/E2E test | A/B test, QA review | Demo feedback, Win/loss analysis | Interview assessment, 360 review | Verification |
| Bug | Software bug | Creative revision, Copy error | Objection, Deal blocker | Process gap, Compliance issue | Issue/Defect |
| Regression | Feature broken by new code | Metric drop after change | Conversion drop after pitch change | Hiring quality drop after process change | Side effect |
| Design token | Color, spacing, typography | Brand colors, fonts, logo specs | Pitch template, Email template | Offer letter template, Policy format | Brand standards |
| Sprint/Phase | Dev sprint | Campaign flight | Sales quarter | Hiring sprint, Training cycle | Project phase |
| Release | App store submission | Campaign go-live | Product launch | Policy rollout, New hire start date | Public launch |
| Dark mode | UI theme variant | Accessibility variant | Market segment variant | Department-specific variant | Context variant |

---

## 9. Agent Architecture

### Distributed Model — No Single Orchestrator

KFSP uses a distributed model where different systems can spawn skill agents:

```
┌─────────────────────────────────────────────────────────┐
│                    WHO SPAWNS SKILLS?                     │
├─────────────────────────────────────────────────────────┤
│  GSD Executor  ───→ guard after each task               │
│  Ralph Loop    ───→ guard after each iteration          │
│  Human (/kfsp) ───→ Slash command runs inline           │
│  Claude Code   ───→ Spawns agents in parallel           │
└─────────────────────────────────────────────────────────┘
```

**Key insight:** KFSP skill agents are **read-only analysts**. They READ code/assets, SCAN patterns, and PRODUCE reports. They do NOT write code. The orchestrator (GSD/Ralph/human) writes code and calls KFSP agents to verify.

### 3 companion tools

| Tool | Role | How it pairs with KFSP |
|------|------|----------------------|
| [GSD](https://github.com/gsd-build/get-shit-done) | 🧠 The Brain — Planning & execution | GSD handles *what to build*, KFSP ensures *it's built safely* |
| [Ralph Loop](https://github.com/frankbria/ralph-claude-code) | 🦶 The Legs — Autonomous iteration | Ralph handles *execution loops*, KFSP guard verifies after each loop |
| KFSP | 🥋 The Armor — Protection | KFSP wraps around GSD/Ralph as quality shield |

---

## 10. The 4 Interaction Surfaces

Every project has 4 surfaces where components interact. Testing these "contracts" prevents integration failures.

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

**Cross-domain mapping:**

| Surface | Product Dev | Marketing | Sales |
|---------|------------|-----------|-------|
| **Front** | User ↔ App UI | Customer ↔ Campaign | Prospect ↔ Sales pitch |
| **Back** | App ↔ API Server | Campaign ↔ CRM/Analytics | Sales ↔ CRM/Pipeline |
| **Lateral** | App ↔ 3rd Party (Firebase, TradingView) | Campaign ↔ Ad Platforms (Meta, Google) | Sales ↔ Partners |
| **Internal** | Component ↔ Component | Channel ↔ Channel | Team ↔ Team |

---

## 11. Modular Training Guide

This architecture guide can be split into 7 training modules for onboarding, seminars, or NotebookLM content generation.

| Module | Content | Audience | Duration | NotebookLM Output |
|--------|---------|----------|----------|-------------------|
| 01 | Philosophy: Everything is a Project (fractal model) | All | 30 min | Podcast overview |
| 02 | 5-Tier Protection Model | PM, PO | 45 min | Flashcards |
| 03 | 6 Stages of a Project | PM, PO, Dev | 60 min | Mindmap |
| 04 | Test & QA Architecture | QA, PM | 60 min | Video walkthrough |
| 05 | 20 Skills Deep Dive | Dev, AI agent | 90 min | Reference cards |
| 06 | Cross-Domain Application | Marketing, Sales, HR | 45 min | Slide deck |
| 07 | Case Study: Real Project | All | 60 min | Podcast case study |

### Using with NotebookLM

Upload `ARCHITECTURE.md` + `OPERATIONS_GUIDE.md` as sources to NotebookLM. Ask it to generate:
- **Podcast:** 2-person discussion about the framework
- **Flashcards:** Key concepts for quick review
- **Study guide:** Quiz-style learning
- **FAQ:** Common questions from new PMs

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-20 | Initial release — full architecture guide with cross-domain examples |

---

## Related Documents

| Document | Location | Purpose |
|----------|----------|---------|
| README.md | `kfsp-skill-set/` | Quick start + installation |
| OPERATIONS_GUIDE.md | `kfsp-skill-set/` | Detailed operations manual |
| ORCHESTRATION.md | `kfsp-skill-set/` | GSD + Ralph integration |
| architecture-diagram.html | `kfsp-skill-set/docs/` | Interactive visual guide |
| INTERACTION_SURFACE_MAP.md | `kfsp-skill-set/references/` | 36 interaction types |
