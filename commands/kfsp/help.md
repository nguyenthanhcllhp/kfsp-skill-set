---
name: kfsp:help
description: Show all KFSP proprietary skills — 17 skills across 5 tiers for agent-driven product development
allowed-tools:
  - Read
---
<objective>
Hiển thị danh sách tất cả KFSP skills, tier map, workflow, và portable setup guide.
</objective>

<instructions>
Output the following directly:

```markdown
# 🥋 KFSP Skill Set v3.2
**17 skills · 5 tầng bảo vệ · Human + Agent compatible**

> Bộ skill độc quyền cho phát triển sản phẩm end-to-end.
> Thiết kế để: human dùng được, agent swarm an toàn, reusable cho project mới.

---

## 🏛️ Mô hình 5 tầng bảo vệ

```
┌─────────────────────────────────────────────────────────┐
│  T5 🎯 BUSINESS IMPACT                                  │
│  Sản phẩm đạt mục tiêu? Docs cho user/store ready?     │
│  Skills: doc-pilot, pre-mortem                          │
├─────────────────────────────────────────────────────────┤
│  T4 👤 USER EXPERIENCE                                   │
│  User bị ảnh hưởng tiêu cực không?                      │
│  Skills: ux-parity, doc-pilot                           │
├─────────────────────────────────────────────────────────┤
│  T3 🧪 QUALITY ASSURANCE                                 │
│  Tests pass? Regression? Contracts đúng?                │
│  Skills: surface-test, sentinel, release-gate,          │
│          test-brief, bug-log, build-verify              │
├─────────────────────────────────────────────────────────┤
│  T2 🔧 CODE INTEGRITY                                    │
│  Blast radius? Dependencies? Tokens? Sync?              │
│  Skills: sweep, pre-commit, sync-check, post-phase      │
├─────────────────────────────────────────────────────────┤
│  T1 🛡️ AGENT SAFETY                                      │
│  Agent làm đúng? Không vượt quyền? Context giữ chặt?   │
│  Skills: guard, dev-journal                             │
└─────────────────────────────────────────────────────────┘
```

---

## 📋 Tất cả 17 Skills

### T5 🎯 BUSINESS IMPACT + T4 👤 USER EXPERIENCE

| Skill | Mô tả | Tốc độ |
|-------|--------|--------|
| `/kfsp:doc-pilot [--status\|--what-changed\|--hdsd-gap\|--checklist]` | 📚 Hoa tiêu tài liệu — biết ghi gì vào đâu, HDSD ready trước store | 1-3 phút |
| `/kfsp:pre-mortem <mô-tả>` | 🔮 Dự báo rủi ro 4 bề mặt TRƯỚC khi build | 2-3 phút |
| `/kfsp:ux-parity [screen] [--quick]` | 🎨 So sánh UI thực tế vs Figma/tokens — design drift | 30s-2 phút |

### T3 🧪 QUALITY ASSURANCE

| Skill | Mô tả | Tốc độ |
|-------|--------|--------|
| `/kfsp:surface-test [front\|back\|lateral\|internal\|all] [--quick]` | 🧪 Test contracts 4 bề mặt tương tác | 1-5 phút |
| `/kfsp:sentinel [--baseline\|--compare\|--quick]` | 📸 Regression: snapshot trước → so sánh sau | 30s-3 phút |
| `/kfsp:release-gate [--quick] [--store-check]` | 🚦 Gate trước release: 10 gates, score /100 | 2-8 phút |
| `/kfsp:test-brief [build-report] [--from-changes]` | 📋 Tạo test checklist cho PM — test gì, kỳ vọng gì, cách test | 1-2 phút |
| `/kfsp:bug-log [--log\|--list\|--resolve\|--stats]` | 🐛 Log bug từ PM testing, track resolution xuyên sessions | 30s-1 phút |
| `/kfsp:build-verify [--quick\|--full]` | ✅ Gate tự động trước khi giao PM — analyze, test, checks | 30s-5 phút |

### T2 🔧 CODE INTEGRITY

| Skill | Mô tả | Tốc độ |
|-------|--------|--------|
| `/kfsp:sweep [file-or-area]` | 🔍 Blast radius sau thay đổi code | 1-2 phút |
| `/kfsp:pre-commit` | ✅ Validation gate trước commit | ~30 giây |
| `/kfsp:sync-check [--fix]` | 🔄 Đồng bộ source copies + docs | ~1 phút |
| `/kfsp:post-phase <N>` | 📋 Checklist hoàn thành phase | 2-3 phút |

### T1 🛡️ AGENT SAFETY + 🔄 CROSS-CUTTING

| Skill | Mô tả | Tốc độ |
|-------|--------|--------|
| `/kfsp:guard [--deep] [--report-only]` | 🛡️ Guardian: health score + agent safety | 1-5 phút |
| `/kfsp:dev-journal [--log\|--search\|--digest\|--init]` | 📔 Nhật ký "án lệ" — decisions, incidents, learnings | 1-5 phút |
| `/kfsp:incident-review <mô-tả>` | 🚨 Post-incident: 5 Whys + prevention + skill updates | 5-10 phút |
| `/kfsp:session-start [--quick\|--skip-pull]` | 🚀 Session startup: git check, code-doc drift, memory update | 1-3 phút |

### Utility

| Skill | Mô tả |
|-------|--------|
| `/kfsp:help` | 📖 Hiển thị guide này |

---

## 🔗 Phối hợp với GSD + Ralph

KFSP Skill Set hoạt động TỐT NHẤT khi kết hợp với **GSD** (planning) và **Ralph Loop** (auto-execution).

| Hệ thống | Vai trò | Check |
|-----------|---------|-------|
| **KFSP** | 🛡️ Bảo vệ (17 skills, 5 tầng) | `ls ~/.claude/commands/kfsp/ \| wc -l` ≥ 17 |
| **GSD** | 📋 Plan → Execute → Verify | `/gsd:help` |
| **Ralph** | 🔄 Auto-loop cho task phức tạp | `which ralph` |

**Tài liệu chi tiết:** Đọc `ORCHESTRATION.md` trong repo này.

**Quick reference:**
- GSD plan → include KFSP pre-mortem output (risks + test cases)
- GSD execute → spawn KFSP sweep + guard sau mỗi task
- Ralph loop → KFSP guard sau mỗi iteration (score < 70 = break)
- Mỗi commit → KFSP pre-commit gate
- Mỗi build → KFSP build-verify + test-brief

---

## ⏱️ Khi nào chạy skill nào?

### Pyramid: Thường xuyên nhất = nhanh nhất

```
Mỗi session mới (~2p):   session-start
Mỗi commit (~30s):      pre-commit
Mỗi thay đổi (~2p):     sweep
Mỗi feature (~3p):      pre-mortem (trước) → ux-parity (sau)
Mỗi build session:      build-verify → [PASS] → test-brief → giao PM
PM tìm bug:             bug-log --log → track → bug-log --resolve
Mỗi phase (~10p):       post-phase → doc-pilot → sync-check
Mỗi release (~15p):     sentinel → release-gate → surface-test → doc-pilot --checklist
Khi có sự cố:           incident-review → dev-journal --log incident
Sau mỗi decision:       dev-journal --log decision
Định kỳ (tuần):         guard --deep → doc-pilot --status → bug-log --stats
Agent tự động:          guard (sau mỗi task)
```

### Workflow chuỗi

**🆕 Feature mới:**
pre-mortem → [dev] → sweep → ux-parity → doc-pilot --what-changed → dev-journal --log decision

**🔧 Bug fix:**
bug-log --list → [fix] → sweep → pre-commit → commit → bug-log --resolve → dev-journal --log incident

**📦 Build session xong:**
build-verify → [PASS] → test-brief → giao PM test → PM test → bug-log (nếu có bug)

**🚀 Release:**
sentinel --baseline → [dev] → sentinel --compare → release-gate → surface-test all → doc-pilot --checklist → bug-log --stats (0 critical/high)

**📦 Phase complete:**
post-phase N → doc-pilot --status → sync-check → guard → dev-journal --log learning

**🤖 Agent tự động:**
[agent task] → guard → nếu OK tiếp, nếu FAIL → stop + incident-review

---

## 🚀 Portable Setup — Cài cho Project Mới

### Bước 1: Cài tools nền
```bash
# GSD (Get Shit Done) — https://github.com/gsd-build/get-shit-done
# Ralph Loop — https://github.com/frankbria/ralph-claude-code
```
> ⚠️ Đọc `ORCHESTRATION.md` để biết cách 3 hệ thống phối hợp.

### Bước 2: Cài domain skills (tùy project)
```bash
# Ví dụ Flutter project:
# Copy flutter-patterns, flutter-development skills vào .claude/skills/
```

### Bước 3: Cài KFSP Skill Set
```bash
# Copy toàn bộ folder commands/kfsp/ vào project mới:
cp -r ~/.claude/projects/[old-project]/commands/kfsp/ \
      ~/.claude/projects/[new-project]/commands/kfsp/

# Hoặc nếu đã publish lên registry:
# /install kfsp-skill-set
```

### Bước 4: Init project
```bash
/kfsp:dev-journal --init          # Khởi tạo journal
/kfsp:doc-pilot --status          # Baseline tài liệu
/gsd:new-project                  # Khởi tạo roadmap
```

### Bước 5: Develop!
```
/gsd:plan-phase 1                 # Plan
/kfsp:pre-mortem "Phase 1 features"  # Risk forecast
/gsd:execute-phase 1              # Execute
/kfsp:post-phase 1                # Verify + docs
/kfsp:guard                       # Health check
```

### Adapt cho project khác:
- Đổi source path detection trong mỗi skill
- Đổi feature list / HDSD mapping trong doc-pilot
- Đổi design token paths trong ux-parity
- Đổi architecture references trong guard/sweep
- Journal + incident-review + pre-mortem → dùng ngay, không cần sửa

---

## 🤖 Tại sao bộ skill này tốt cho Agent Team/Swarm?

| Vấn đề Agent | Skill giải quyết |
|---------------|-----------------|
| **Hallucination** (bịa thông tin) | `guard` verify mọi thay đổi có evidence |
| **Context loss** (mất bối cảnh) | `dev-journal` lưu precedent, agent mới search được |
| **Scope creep** (vượt phạm vi) | `pre-mortem` define rõ scope trước, `guard` check sau |
| **Regression** (phá feature cũ) | `sentinel` detect before/after, `sweep` blast radius |
| **Drift** (lệch design/spec) | `ux-parity` check design, `sync-check` check docs |
| **Silent failure** (lỗi ngầm) | `surface-test` verify contracts 4 bề mặt |
| **No docs** (không tài liệu) | `doc-pilot` nhắc + biết ghi gì vào đâu |
| **No learning** (không học kinh nghiệm) | `dev-journal` ghi, `incident-review` phân tích |
| **Uncontrolled** (mất kiểm soát) | 5 tầng bảo vệ + guard sau mỗi task |

---

## 📖 Tham khảo

| Tài liệu | Nội dung |
|-----------|----------|
| `docs/14_INTERACTION_SURFACE_MAP.md` | Contracts 4 bề mặt (36 interactions) |
| `docs/13_DEPENDENCY_IMPACT_MAP.md` | Dependency graph + blast radius |
| `docs/12_TESTING_LIFECYCLE.md` | 7 giai đoạn testing |
| `docs/08_HANDOFF_STATUS.md` | Trạng thái hiện tại |
| `journal/INDEX.md` | Dev Journal index |
| `HDSD_KFSP/SUMMARY.md` | User Guide table of contents |
```
</instructions>
