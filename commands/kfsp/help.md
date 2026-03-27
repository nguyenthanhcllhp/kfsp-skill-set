---
name: kfsp:help
description: "Show all KFSP skills v4.0 — 7 skills with modes, sync system, dual-session workflow"
allowed-tools:
  - Read
---

# /kfsp:help — KFSP Skill Set v4.0

## 7 Skills

| # | Lệnh | Mục đích | Modes |
|---|------|----------|-------|
| 1 | `/kfsp:start` | Mở phiên — git check, drift detect, .changes.json, dual-session | — |
| 2 | `/kfsp:check` | Rà soát — health, blast radius, tokens, sync, convention | `--quick` `--full` `--tokens` `--sync` |
| 3 | `/kfsp:test` | Kiểm thử — verify build, brief PM, log bugs | `--verify` `--brief` `--bug` `--bugs` |
| 4 | `/kfsp:docs` | Tài liệu — status, update từ .changes.json, đóng phase | `--status` `--update` `--close-phase` |
| 5 | `/kfsp:gate` | Cổng chặn — pre-commit (23 checks), pre-release (12 gates) | `--commit` `--release` |
| 6 | `/kfsp:log` | Ghi chép — decisions, incidents, 5-Whys, search | `--log` `--incident` `--search` `--digest` |
| 7 | `/kfsp:risk` | Rủi ro — pre-mortem, contracts 4 bề mặt, regression | `--before` `--contracts` `--regression` |

## Phủ 5 tầng

| Tầng | Skills |
|------|--------|
| T1 An toàn | `start`, `check`, `log` |
| T2 Code | `check`, `gate` |
| T3 QA | `test`, `risk`, `gate` |
| T4 UX | `check --tokens` (convention) |
| T5 Business | `docs`, `risk` |

## Sync System (Rule 21)

```
Desktop ghi .changes.json → Terminal đọc → cập nhật docs/specs/TC
```

File: `docs/logs/.changes.json` (1 file / 1 phiên)

## Lifecycle

```
start → risk --before → plan → code → check --quick
→ build (flutter run/build) → test --verify → test --brief
→ gate --commit → git commit
→ PM test → test --bug
→ check --full → docs --close-phase → log --log
→ risk --contracts → risk --regression → gate --release
```

## Dual-Session (Rule 20)

| Desktop | Terminal |
|---------|----------|
| Code: `lib/`, `test/` | Docs: `docs/`, `test_cases/`, `QA/` |
| `start`, `check`, `gate --commit`, `test --verify` | `docs`, `test --brief`, `test --bug`, `risk`, `log` |

**CẤM** 2 phiên cùng sửa 1 file.

## Tài liệu thiết kế

- Flow chi tiết: `Product/kfsp-skill-set/KFSP_Skill_v4_Workflow.html`
- Sync System: `Product/kfsp-skill-set/KFSP_Sync_System_v4.html`
- Live Guide: `Product/kfsp-skill-set/KFSP_Live_Guide.html`
