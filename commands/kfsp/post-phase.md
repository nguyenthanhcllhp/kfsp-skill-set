---
name: kfsp:post-phase
description: Post-phase completion checklist — update docs, sync source copies, run regression check, archive phase artifacts
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Post-Phase Checklist — Chạy sau khi hoàn thành một phase trong Clean Slate Rebuild.

Đảm bảo:
1. Docs được cập nhật reflect đúng trạng thái mới
2. Source copies được đồng bộ
3. Regression test cho các phase trước vẫn pass
4. Known issues list cập nhật
5. Migration package sẵn sàng handoff bất kỳ lúc nào

**Quy tắc KFSP:** "Mỗi phase = 1 lần build & test trên thiết bị thật."

**Orchestration:**
- Chạy SAU khi GSD verify-work xong (hoặc thay thế nếu không có GSD)
- TRƯỚC khi GSD complete-milestone
- Guard score cuối PHẢI ≥ 70 để close phase
- Chi tiết: xem `ORCHESTRATION.md` trong kfsp-skill-set repo
</objective>

<instructions>
## Input
`$ARGUMENTS` = phase number (0, 1, 2, 3, 4)
Nếu không có argument → hỏi user phase nào.

## Phase Context

Đọc plan để hiểu phase goals:
```
Product/kfsp_flutter_migration/docs/11_CLEAN_SLATE_PLAN.md
```

| Phase | Goal |
|-------|------|
| 0 | App build + 6 tabs bottom nav |
| 1 | Onboarding 5 slides → Dashboard |
| 2 | Dashboard 11 sections + sidebar + tools |
| 3 | TradingView chart 30 mã VN30 |
| 4 | FA 13 tabs + stock detail |

## Step 0.5: 📋 Project Charter Verification (BẮT BUỘC — 2026-03-15+)

**GATE:** Phase KHÔNG ĐƯỢC close nếu chưa có Project Charter hoàn chỉnh 6/6 chiều.

```bash
CHARTER="Product/kfsp_flutter_migration/docs/build_reports/P${PHASE}_project_charter.md"
if [ -f "$CHARTER" ]; then
  echo "✅ Charter found: $CHARTER"
  # Verify all 6 dimensions present
  for dim in "Goals" "Scope" "Deliverables" "Success Criteria" "Stakeholders" "Resources"; do
    grep -q "$dim" "$CHARTER" && echo "  ✅ $dim" || echo "  ❌ MISSING: $dim"
  done
else
  echo "❌ NO PROJECT CHARTER — must create before closing phase"
  echo "File expected: $CHARTER"
fi
```

**6 chiều bắt buộc:**

| # | Chiều | Verify |
|---|-------|--------|
| 1 | **Goals** | Mục tiêu cụ thể, đo lường được — ĐÃ ĐẠT? |
| 2 | **Scope** | In scope / out of scope — có scope creep không? |
| 3 | **Deliverables** | Screens, widgets, APIs — đã giao đủ? |
| 4 | **Success Criteria** | Pass/fail — kết quả? |
| 5 | **Stakeholders** | PM approved? Dev confirmed? |
| 6 | **Resources** | Effort thực tế vs estimate? |

**Nếu thiếu charter → TẠO NGAY trước khi tiếp tục close phase.**

## Step 0.6: 📔 Dev Journal Compliance Check

```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
MONTH=$(date +%Y-%m)
# Check for phase-start entry
grep -rl "phase-start\|Phase.*Start" "$JOURNAL_DIR/$MONTH/" 2>/dev/null | head -5
# Check for decision entries during this phase
grep -rl "decision\|Decision" "$JOURNAL_DIR/$MONTH/" 2>/dev/null | wc -l
# Check for any entries at all
ENTRIES=$(find "$JOURNAL_DIR/$MONTH/" -name "*.md" 2>/dev/null | wc -l)
echo "Dev journal entries this month: $ENTRIES"
```

**Required entries for phase close:**
- [ ] `phase-start` entry (with charter summary)
- [ ] At least 1 `decision` entry per major technical choice
- [ ] `bug-fix` entries for all bugs found and fixed
- [ ] `pm-feedback` entries for all Thanh feedback
- [ ] `phase-complete` entry (results vs plan) — CREATE NOW

**Nếu thiếu entries → agent PHẢI bổ sung trước khi close phase.**

## Step 1: Build Verification ✅

Confirm với user:
- App đã build thành công trên simulator? (Y/N)
- Phase features đã test bằng tay? (Y/N)
- Screenshot có chưa? (optional)

Nếu chưa build → STOP. Nhắc quy tắc: "Không code tiếp khi chưa verify phase trước."

## Step 2: Regression Check 🔄

Dựa trên phase number, kiểm tra regression:

**After Phase 0:** Bottom nav 6 tabs switch ✅
**After Phase 1:** + Onboarding 5 slides + persistence
**After Phase 2:** + Dashboard 11 sections + tools grid
**After Phase 3:** + TradingView chart renders
**After Phase 4:** + FA 13 tabs + navigation from Dashboard

Tạo regression checklist và hỏi user confirm từng item.

## Step 3: Update Docs 📝

### 3a: docs/08_HANDOFF_STATUS.md
Cập nhật:
- Phase completion: ✅ cho phase vừa xong
- Progress bar percentages
- Feature list: đánh dấu features vừa implement
- Known issues: thêm/bỏ issues

### 3b: README.md
Cập nhật:
- Timeline section: thêm date + ✅ cho phase vừa xong
- Known Issues table: refresh
- Status line ở đầu file

### 3c: docs/04_FEATURE_SPECS.md
Thêm specs cho features vừa implement (nếu chưa có).

### 3d: docs/00_INDEX.md
Kiểm tra mục lục còn accurate.

## Step 4: Source Sync 🔄

```bash
# Sync working → build copy
rsync -av --delete /tmp/kfsp_flutter/ ~/Desktop/kfsp_flutter/

# Sync working → migration package
rsync -av --delete \
  --exclude='.dart_tool' \
  --exclude='.packages' \
  --exclude='build/' \
  --exclude='.flutter-plugins*' \
  /tmp/kfsp_flutter/ "Product/kfsp_flutter_migration/source/"
```

Hỏi user confirm trước khi chạy rsync.

## Step 5: Impact Sweep (Mini) 🔍

Chạy lightweight version của /kfsp:sweep cho source vừa sync:
- Design token compliance check
- Route validation
- Import validation

## Step 6: Archive Phase Artifacts 📦

Tạo hoặc update phase summary:

```markdown
## Phase [N] Summary
- **Completed:** [date]
- **Duration:** [estimated]
- **Files changed:** [count]
- **New widgets:** [list]
- **Known issues:** [list]
- **Regression:** All previous phases verified ✅
```

## Output Format

```markdown
# ✅ KFSP Post-Phase [N] Checklist
**Date:** [timestamp]
**Phase:** [N] — [description]

## Completion Status
| Step | Status | Notes |
|------|--------|-------|
| 1. Build Verified | ✅/❌ | [details] |
| 2. Regression Check | ✅/⚠️ | [N]/[N] items passed |
| 3. Docs Updated | ✅/⚠️ | [list of updates] |
| 4. Source Synced | ✅/❌ | [copies synced] |
| 5. Impact Sweep | ✅/⚠️ | [findings count] |
| 6. Phase Archived | ✅ | [summary] |

## Project Charter Status
| Chiều | Status | Notes |
|-------|--------|-------|
| Goals | ✅/❌ | [achieved?] |
| Scope | ✅/⚠️ | [scope creep?] |
| Deliverables | ✅/❌ | [all delivered?] |
| Success Criteria | ✅/❌ | [met?] |
| Stakeholders | ✅/❌ | [PM approved?] |
| Resources | ✅/⚠️ | [effort vs estimate] |

## Dev Journal Compliance
- phase-start entry: ✅/❌
- decision entries: X found
- bug-fix entries: X found
- pm-feedback entries: X found
- phase-complete entry: ✅/❌

## Ready for Phase [N+1]: YES ✅ / NO ❌
[if NO, list blockers]
```
</instructions>
