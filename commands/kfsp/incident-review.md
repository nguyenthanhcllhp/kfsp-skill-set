---
name: kfsp:incident-review
description: Post-incident review template — structured analysis of bugs, crashes, or production issues with root cause, blast radius, and preventive actions
argument-hint: "<incident-description>"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Incident Review — Phân tích sự cố sau khi xảy ra.

Khi có bug, crash, hoặc production issue, Incident Review giúp:
1. **Timeline** — Chuyện gì xảy ra, theo thứ tự
2. **Root Cause** — Nguyên nhân gốc (5 Whys)
3. **Blast Radius** — Ảnh hưởng trên 4 bề mặt
4. **Fix** — Sửa ngay + sửa triệt để
5. **Prevention** — Làm gì để không tái phát
6. **Lesson** — Bài học rút ra

**KHÔNG blame cá nhân** — focus vào system/process.

**Khi nào dùng:**
- Sau bug production
- Sau crash report từ Crashlytics/Sentry
- Sau user complaint nghiêm trọng
- Sau data loss/corruption
- Sau security incident
- Sau khi YOLO phase thất bại (như đã xảy ra!)
</objective>

<instructions>
## Input
`$ARGUMENTS` = mô tả sự cố.
Nếu rỗng, hỏi: "Mô tả sự cố: gì xảy ra, khi nào, ai phát hiện?"

## Step 1: Incident Timeline

Build chronological timeline:
```
[Thời gian] [Sự kiện]
─────────────────────
[T-X]        Nguyên nhân gốc đã tồn tại từ...
[T-0]        Sự cố xảy ra / được phát hiện
[T+X]        Hành động đầu tiên
[T+Y]        Fix deployed
[T+Z]        Confirmed resolved
```

## Step 2: Impact Assessment on 4 Surfaces

### 🔵 User Impact
- Bao nhiêu user bị ảnh hưởng?
- User experience bị thay đổi thế nào?
- Pro vs Free: ai bị nặng hơn?
- Có user data bị mất/corrupted?
- User có biết không? (silent fail vs visible error)

### 🔴 Backend Impact
- API nào bị ảnh hưởng?
- Data integrity: data có bị sai/mất?
- WebSocket connection: bị disconnect?
- Auth/token: user bị logout?

### 🟡 Service Impact
- 3rd party service nào bị ảnh hưởng?
- TradingView chart: bị blank/wrong?
- Analytics: data bị gap?
- Push notifications: bị miss?

### 🟢 Internal Impact
- Component nào bị cascade fail?
- State management: state bị corrupted?
- Build: CI/CD bị break?
- Tests: test nào should have caught this?

## Step 3: Root Cause Analysis (5 Whys)

```
Why 1: Tại sao sự cố xảy ra?
  → [direct cause]
Why 2: Tại sao [direct cause] xảy ra?
  → [deeper cause]
Why 3: Tại sao [deeper cause] xảy ra?
  → [systemic cause]
Why 4: Tại sao [systemic cause] không được phát hiện sớm?
  → [detection gap]
Why 5: Tại sao [detection gap] tồn tại?
  → [process/tool gap]
```

## Step 4: Fixes

### Immediate Fix (Hotfix)
- What: [sửa gì]
- How: [sửa thế nào]
- Risk: [hotfix có rủi ro gì thêm]

### Permanent Fix (Root Cause)
- What: [sửa triệt để]
- How: [cần refactor/redesign gì]
- Timeline: [bao lâu]

### Rollback Plan (nếu hotfix fail)
- Steps to rollback
- Data recovery steps (if needed)

## Step 5: Prevention

### Tests to Add
```
- [ ] Unit test: [test case mô tả]
- [ ] Widget test: [test case mô tả]
- [ ] Integration test: [test case mô tả]
```

### Monitoring to Add
```
- [ ] Alert: [cảnh báo cần setup]
- [ ] Dashboard: [metric cần track]
- [ ] Log: [log entry cần thêm]
```

### Process to Change
```
- [ ] Code review: [checklist item mới]
- [ ] Pre-commit: [check mới cho kfsp:pre-commit]
- [ ] Sentinel: [baseline mới cho kfsp:sentinel]
```

### Skill Updates
Check: Nên update skill nào?
- `/kfsp:sweep` → thêm check pattern mới?
- `/kfsp:guard` → thêm detection rule?
- `/kfsp:pre-commit` → thêm validation?
- `/kfsp:surface-test` → thêm contract check?

## Step 6: Lesson Learned

Summarize in 1-2 sentences:
"Chúng ta đã học được rằng [lesson]. Từ nay [new practice]."

## Output Format

```markdown
# 🚨 KFSP Incident Review
**Incident:** [short title]
**Severity:** 🔴 Critical / 🟡 Major / 🟢 Minor
**Date:** [when discovered]
**Status:** 🔧 Investigating / ✅ Resolved / 🔄 Monitoring

## Timeline
| Time | Event |
|------|-------|
| T-X | [root cause existed since] |
| T-0 | [incident occurred/discovered] |
| T+X | [first action] |
| T+Y | [fix deployed] |

## Impact Assessment
| Surface | Impact | Severity |
|---------|--------|----------|
| 🔵 User | [description] | 🔴/🟡/🟢 |
| 🔴 Backend | [description] | 🔴/🟡/🟢 |
| 🟡 Services | [description] | 🔴/🟡/🟢 |
| 🟢 Internal | [description] | 🔴/🟡/🟢 |

## Root Cause (5 Whys)
1. [why 1]
2. [why 2]
3. [why 3]
4. [why 4]
5. [why 5] ← **Root cause**

## Fixes
### Immediate
[what was done]

### Permanent
[what needs to be done]

## Prevention
### Tests to Add
- [ ] [test 1]
- [ ] [test 2]

### Monitoring
- [ ] [monitor 1]

### Process Changes
- [ ] [process 1]

### Skill Updates
- [ ] Update `/kfsp:X` — add [check]

## Lesson Learned
> "[One-line lesson]"

## Action Items
| # | Action | Owner | Due | Status |
|---|--------|-------|-----|--------|
| 1 | [action] | [who] | [when] | ⬜ |
```

## Append to Debug Log
After generating report, remind user to also append key info to:
`Product/kfsp_flutter_migration/docs/06_DEBUG_LOG.md`
</instructions>
