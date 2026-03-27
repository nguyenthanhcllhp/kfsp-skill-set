# 🥋 KFSP Skill Set v4.0

**7 skills cho agent-driven product development.** Gộp từ 17 skills v3 — ít lệnh hơn, đủ 5 tầng, không trùng lặp.

## Quick Start

Mở chat mới → gõ:
```
/kfsp:start
```
Agent sẽ tự động: git check, tạo `.changes.json`, mở Live Guide, nhắc bật Terminal.

## 7 Skills

| # | Lệnh | Mục đích | Modes |
|---|------|----------|-------|
| 1 | `/kfsp:start` | Mở phiên — git check, drift detect, tạo sync file, nhắc dual-session | — |
| 2 | `/kfsp:check` | Rà soát — health score, blast radius, token audit, source sync | `--quick` `--full` `--tokens` `--sync` |
| 3 | `/kfsp:test` | Kiểm thử — verify build, tạo brief PM, log bugs | `--verify` `--brief` `--bug` `--bugs` |
| 4 | `/kfsp:docs` | Tài liệu — status, update từ .changes.json, đóng phase | `--status` `--update` `--close-phase` |
| 5 | `/kfsp:gate` | Cổng chặn — pre-commit 23 checks, pre-release 12 gates | `--commit` `--release` |
| 6 | `/kfsp:log` | Ghi chép — decisions, incidents 5-Whys, search precedent | `--log` `--incident` `--search` `--digest` |
| 7 | `/kfsp:risk` | Rủi ro — pre-mortem, contracts 4 bề mặt, regression | `--before` `--contracts` `--regression` |

## Lifecycle

```
/kfsp:start → /kfsp:risk --before → /gsd:plan-phase
    → Code → /kfsp:check --quick
    → Build → /kfsp:test --verify → /kfsp:test --brief → PM test → /kfsp:test --bug
    → /kfsp:gate --commit → git commit (sau build OK)
    → /kfsp:check --full → /kfsp:docs --close-phase → /kfsp:log --log
    → /kfsp:risk --contracts → /kfsp:gate --release
```

## Dual-Session (Rule 20)

2 phiên Claude Code chạy song song, phân vùng file:

| 💻 Desktop | 🖥️ Terminal |
|-----------|------------|
| Code: `lib/`, `test/` | Docs: `docs/`, `test_cases/`, `QA/` |
| `start`, `check`, `test --verify`, `gate --commit` | `docs`, `test --brief`, `test --bug`, `risk`, `log` |

**CẤM** 2 phiên cùng sửa 1 file.

## Sync System (Rule 21)

Desktop ghi `.changes.json` sau mỗi checkpoint → Terminal đọc + cập nhật docs/specs/testcases.

```
Desktop code → ghi .changes.json (BẮT BUỘC)
    → Thanh chuyển sang Terminal
    → Terminal đọc .changes.json → cập nhật docs → đánh dấu processed
    → Cuối phiên: 0 unprocessed = đồng bộ 100%
```

## Phủ 5 tầng

| Tầng | Skills |
|------|--------|
| T1 An toàn | `start`, `check`, `log` |
| T2 Code | `check`, `gate` |
| T3 QA | `test`, `risk`, `gate` |
| T4 UX | `check --tokens` (convention) |
| T5 Business | `docs`, `risk` |

## Tài liệu HTML

| File | Nội dung |
|------|---------|
| `KFSP_Skill_v4_Workflow.html` | Flow, merge map, lifecycle, dual-session diagram |
| `KFSP_Sync_System_v4.html` | Thiết kế sync system chi tiết |
| `KFSP_Live_Guide.html` | Live guide phiên làm việc (auto-open khi start) |

## Response Format (Rule 8)

Mỗi response có code change PHẢI có 4 sections:

```
🥋 Skills đã chạy: check --quick
📝 Docs ảnh hưởng: 04_FEATURES.md — cần thêm spec X
🎨 Convention check: ✅ Tiếng Việt | ✅ Tokens | ✅ Number format
🔄 Sync → Terminal: 2 entries .changes.json, chạy /kfsp:docs --update
```

## Cài đặt

1. Clone repo vào project:
```bash
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git Product/kfsp-skill-set
```

2. Symlink commands vào Claude Code:
```bash
ln -s Product/kfsp-skill-set/commands ~/.claude/commands
```

3. Mở chat mới → `/kfsp:start`

## Changelog

Xem [CHANGELOG.md](CHANGELOG.md) để biết lịch sử thay đổi.

## Lịch sử phiên bản

| Version | Ngày | Thay đổi chính |
|---------|------|----------------|
| v4.0 | 2026-03-27 | Gộp 17→7 skills, Sync System, Dual-Session, Live Guide |
| v3.3 | 2026-03-20 | Architecture doc, Operations Guide, Interactive diagram |
| v3.2 | 2026-03-15 | Orchestration, Phase-as-Project, Dev Journal compliance |
| v3.0 | 2026-03-13 | +4 skills (test-brief, bug-log, build-verify, session-start) |
