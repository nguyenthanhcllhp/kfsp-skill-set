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

## Ready for Phase [N+1]: YES ✅ / NO ❌
[if NO, list blockers]
```
</instructions>
