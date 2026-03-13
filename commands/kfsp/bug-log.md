---
name: kfsp:bug-log
description: Structured bug logging for PM testing — log bugs with steps, severity, screenshots. Track resolution across sessions.
argument-hint: "[--log <description>] [--list] [--resolve <bug-id>] [--stats]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - TodoWrite
---
<objective>
KFSP Bug Log — Ghi chép và theo dõi bug từ PM testing.

**Vấn đề giải quyết:** Thanh test → tìm bug → ghi đâu? Hiện tại ghi rời rạc,
không có format chuẩn, không track resolution, context bị mất giữa sessions.

**Bug Log giải quyết:**
1. Format chuẩn cho mọi bug report
2. Track trạng thái: Open → In Progress → Resolved → Verified
3. Link bug → build report (biết bug từ build nào)
4. Link bug → fix commit (biết đã sửa khi nào)
5. Statistics: bao nhiêu bug open, severity breakdown

**File location:** `Product/kfsp_flutter_migration/docs/test_logs/BUG_LOG.md`

**Modes:**
- `--log <description>`: Log bug mới
- `--list [--open|--resolved|--all]`: Liệt kê bugs
- `--resolve <bug-id>`: Đánh dấu bug đã fix
- `--stats`: Thống kê tổng quan
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `--log <desc>` → tạo bug entry mới
- `--list` → liệt kê bugs (default: open only)
- `--resolve <id>` → mark bug as resolved
- `--stats` → show statistics
- Rỗng → `--list --open` + `--stats`

## File Path
```
BUG_LOG="Product/kfsp_flutter_migration/docs/test_logs/BUG_LOG.md"
```

Ensure directory exists:
```bash
mkdir -p "Product/kfsp_flutter_migration/docs/test_logs/"
```

## Mode: --log (New Bug)

### Generate Bug ID
Format: `BUG-YYYYMMDD-NN` (e.g., `BUG-20260313-01`)
```bash
TODAY=$(date +%Y%m%d)
# Count existing bugs today
EXISTING=$(grep -c "BUG-$TODAY" "$BUG_LOG" 2>/dev/null || echo 0)
NEXT=$((EXISTING + 1))
BUG_ID="BUG-$TODAY-$(printf '%02d' $NEXT)"
```

### Ask PM for Details
Nếu description ngắn, hỏi thêm:
1. **Ở screen nào?** (Dashboard, Market, Tools, Onboarding, etc.)
2. **Steps to reproduce?** (cụ thể từng bước)
3. **Expected vs Actual?**
4. **Severity?** Critical / High / Medium / Low
5. **Screenshot?** (path nếu có)
6. **Dark mode / Light mode / Both?**

### Bug Entry Format
Append to BUG_LOG:

```markdown
---

### BUG-YYYYMMDD-NN: [Short title]
- **Status:** 🔴 Open
- **Severity:** Critical / High / Medium / Low
- **Screen:** [screen name]
- **Mode:** Dark / Light / Both
- **Build:** [build report date, nếu biết]
- **Reported:** YYYY-MM-DD
- **Resolved:** —

**Steps to reproduce:**
1. [step 1]
2. [step 2]
3. [step 3]

**Expected:** [what should happen]
**Actual:** [what actually happens]

**Screenshot:** [path or "không có"]

**Fix:** — (cập nhật khi resolve)
```

## Mode: --list

### Show Bugs
```bash
# Parse BUG_LOG.md → extract bug entries
grep -A2 "^### BUG-" "$BUG_LOG"
```

Display as table:
```markdown
| Bug ID | Title | Severity | Screen | Status | Reported |
|--------|-------|----------|--------|--------|----------|
| BUG-20260313-01 | [title] | 🔴 High | Dashboard | 🔴 Open | 2026-03-13 |
```

Filter options:
- `--open`: Only 🔴 Open + 🟡 In Progress
- `--resolved`: Only 🟢 Resolved + ✅ Verified
- `--all`: Everything

## Mode: --resolve

### Update Bug Status
Find bug by ID → update:
- `Status: 🔴 Open` → `🟢 Resolved`
- `Resolved: —` → `Resolved: YYYY-MM-DD`
- `Fix: —` → `Fix: [commit hash or description]`

### Cross-reference
- Add bug ID to build report that contains the fix
- Add bug ID to `06_DEBUG_LOG.md` if it led to a new pattern/lesson

## Mode: --stats

### Generate Statistics
```markdown
## 📊 Bug Statistics
**Total:** X bugs
**Open:** X (🔴 Critical: X, High: X, Medium: X, Low: X)
**In Progress:** X
**Resolved:** X
**Verified:** X

### By Screen
| Screen | Open | Resolved | Total |
|--------|------|----------|-------|

### By Build
| Build | Bugs Found | Bugs Fixed | Net |
|-------|-----------|-----------|-----|

### Trend
- Week 1: +X bugs, -X resolved
- Week 2: +X bugs, -X resolved
```

## BUG_LOG.md File Structure

```markdown
# 🐛 KFSP Bug Log

> Bug tracker cho PM testing. Mỗi bug có ID duy nhất, severity, steps to reproduce.
> Agent dùng file này để track resolution xuyên sessions.

## Quick Stats
- **Open:** X | **Resolved:** X | **Total:** X
- **Last updated:** YYYY-MM-DD

---

## Open Bugs

### BUG-YYYYMMDD-NN: [title]
...

---

## Resolved Bugs

### BUG-YYYYMMDD-NN: [title]
...
```

## Integration Rules

1. **Sau mỗi build** → check BUG_LOG cho open bugs liên quan → ghi vào build report
2. **Fix bug** → `--resolve` + ghi vào `06_DEBUG_LOG.md` nếu là incident
3. **Trước release** → `--stats` phải show 0 Critical, 0 High open bugs
4. **Memory update** → nếu bug pattern mới → ghi vào `dev-journal`
</instructions>
