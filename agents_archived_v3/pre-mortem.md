---
name: kfsp:pre-mortem
description: Pre-mortem risk forecast before building — predict what can go wrong across 4 interaction surfaces (user, backend, services, internal) and plan mitigations
argument-hint: "<feature-or-change-description>"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Pre-Mortem — Dự báo rủi ro TRƯỚC KHI build/sửa/nâng cấp.

Thay vì hỏi "cái gì có thể sai?" SAU KHI xong, Pre-Mortem hỏi TRƯỚC:
"Giả sử feature này thất bại hoàn toàn — vì lý do gì?"

Phân tích trên 4 bề mặt tương tác:
1. **User Impact** — User bị ảnh hưởng gì? UX regression?
2. **Backend Risk** — API/data bị break? Format thay đổi?
3. **Service Risk** — 3rd party service bị ảnh hưởng?
4. **Internal Risk** — Component nào bị cascade break?

**Khi nào dùng:**
- Trước khi bắt đầu feature mới
- Trước khi refactor lớn
- Trước khi upgrade dependency
- Trước khi đổi API contract
- Trước khi đổi design tokens

**Orchestration:**
- Chạy TRƯỚC `/gsd:plan-phase` → output risks + test cases đưa vào plan
- Ralph: chạy trước `ralph-enable` để define scope + risks
- Output test cases → thêm vào Master Test Registry
- Chi tiết: xem `ORCHESTRATION.md` trong kfsp-skill-set repo
</objective>

<instructions>
## Input
`$ARGUMENTS` = mô tả feature/thay đổi sắp làm.
Nếu rỗng, hỏi user: "Bạn định làm gì? (feature mới, sửa bug, refactor, upgrade?)"

## Step 1: Understand the Change

Read relevant docs:
- `Product/kfsp_flutter_migration/docs/13_DEPENDENCY_IMPACT_MAP.md`
- `Product/kfsp_flutter_migration/docs/04_FEATURE_SPECS.md`
- `Product/kfsp_flutter_migration/docs/02_ARCHITECTURE_GUIDE.md`

Identify:
- Files likely to be modified
- Dependencies of those files (from Impact Map)
- Existing tests that cover these areas

## Step 1.5: Feature Completeness Forecast (6 Dimensions — 2026-03-13+)

Before analyzing risks, verify the PLAN includes all 6 dimensions:

| # | Dimension | Planned? | Notes |
|---|-----------|----------|-------|
| 1 | **Animation** | ✅/❌ | Transitions, loading, feedback visual? |
| 2 | **Action** | ✅/❌ | All tappable elements have handlers? Drill-down? Navigate? |
| 3 | **Logic** | ✅/❌ | Business rules, edge cases, error/empty states? |
| 4 | **Customer Insight** | ✅/❌ | What user pain point does this solve? |
| 5 | **Value** | ✅/❌ | Pro vs Free? Competitive advantage? |
| 6 | **UX/UI Test** | ✅/❌ | Design parity plan? Dark/light mode? |

Also check:
- **Vietnamese text:** All user-facing strings use proper diacritics (có dấu)?

If any dimension is ❌ → flag as risk and add to mitigation plan.

## Step 2: Pre-Mortem on 4 Surfaces

For each surface, imagine the change is DONE and FAILED:

### 🔵 Surface 1: User Impact
Ask:
- "User mở app → thấy gì khác?" → Có bất ngờ không?
- "User đang dùng feature X → bị break không?"
- "Pro user vs Free user → gate có đúng không?"
- "User flow A→B→C → bước nào có thể stuck?"
- "Push notification / deep link → còn hoạt động?"

Score: Impact Level (🔴 High / 🟡 Medium / 🟢 Low) × Probability (H/M/L)

### 🔴 Surface 2: Backend Risk
Ask:
- "API call nào bị thay đổi?" → Response format khác?
- "Mock data có phản ánh đúng real API không?"
- "Token/Auth flow có bị ảnh hưởng?"
- "WebSocket events có bị break?"
- "Database migration cần không?"

### 🟡 Surface 3: Service Risk
Ask:
- "TradingView chart có bị ảnh hưởng?"
- "Firebase push còn hoạt động?"
- "Analytics events có bị thiếu/sai?"
- "Social login flow có thay đổi?"
- "3rd party SDK version compatibility?"

### 🟢 Surface 4: Internal Risk
Use dependency map to check:
- "Theme cascade: kfsp_colors → theme_colors → theme → extensions → 45+ files?"
- "Router: route name change → bao nhiêu nơi reference?"
- "Provider: state type change → bao nhiêu widget rebuild?"
- "Shared widget: prop change → bao nhiêu consumer?"

## Step 3: Risk Matrix

Compile all risks into matrix:

```
| # | Risk | Surface | Impact | Probability | Score | Mitigation |
|---|------|---------|--------|-------------|-------|------------|
| 1 | ... | 🔵 User | 🔴 High | 🟡 Medium | 6 | ... |
```

Score = Impact (H=3, M=2, L=1) × Probability (H=3, M=2, L=1)
- Score 6-9: 🔴 MUST mitigate before starting
- Score 3-5: 🟡 SHOULD mitigate
- Score 1-2: 🟢 ACCEPT risk, monitor

## Step 4: Mitigation Plan

For each 🔴 and 🟡 risk:
1. **Preventive action** — What to do BEFORE building
2. **Detection method** — How to DETECT if risk materializes
3. **Rollback plan** — How to UNDO if things go wrong
4. **Test to write** — Specific test case to prevent regression

## Step 5: Go/No-Go Decision

```
Total risks: X
🔴 Critical: X (all must have mitigation)
🟡 Medium: X (>50% should have mitigation)
🟢 Low: X (accepted)

DECISION: ✅ GO / ⚠️ GO WITH CONDITIONS / 🛑 NO-GO
```

If NO-GO: explain why and suggest alternative approach.

## Output Format

```markdown
# 🔮 KFSP Pre-Mortem Report
**Feature/Change:** [description]
**Date:** [timestamp]
**Decision:** ✅ GO / ⚠️ GO WITH CONDITIONS / 🛑 NO-GO

## Change Summary
[What will be changed, files affected, estimated blast radius]

## Risk Matrix
| # | Risk | Surface | Impact | Prob | Score | Mitigation |
|---|------|---------|--------|------|-------|------------|

## Top 3 Risks to Watch
1. [most dangerous]
2. [second]
3. [third]

## Pre-Build Checklist
- [ ] [preventive action 1]
- [ ] [preventive action 2]
- [ ] [test to write before building]

## Rollback Plan
[step-by-step rollback if things go wrong]

## Post-Build Verification
- [ ] [what to check after done]
```
</instructions>
