---
name: kfsp:build-verify
description: Post-build automated verification — flutter analyze, flutter test, runtime checks. Gate before handing build to PM for testing.
argument-hint: "[--quick] [--full]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Build Verify — Kiểm tra tự động SAU KHI build, TRƯỚC KHI giao PM test.

**Vấn đề giải quyết:** Agent build xong → giao Thanh test luôn → Thanh gặp lỗi cơ bản
(analyze fail, test crash, import broken). Lãng phí thời gian PM.

**Build Verify là GATE:**
- Agent PHẢI chạy build-verify TRƯỚC khi tạo test brief cho Thanh
- Nếu FAIL → agent fix → chạy lại → chỉ PASS mới giao PM

**Workflow:**
```
Code changes → build-verify → [PASS] → test-brief → Thanh test
                    ↓
                  [FAIL] → agent fix → build-verify (lại)
```

**Modes:**
- `--quick` (~30s): analyze + basic checks only
- Default (~1-2 min): analyze + test + runtime checks
- `--full` (~3-5 min): + build iOS + screenshot comparison
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `--quick` → quick mode
- `--full` → full mode (includes build)
- Rỗng → default mode

## Source Path Detection
```bash
for path in "/tmp/dev_flutter_kfsp" "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter_dev"; do
  [ -d "$path/lib" ] && SOURCE="$path" && break
done
```

## CHECK 1: 🔍 Static Analysis (GATE — must pass)

```bash
cd "$SOURCE"
flutter analyze 2>&1
```

**Criteria:**
- 0 errors → ✅ PASS
- Errors found → ❌ FAIL (list errors, agent must fix)
- Warnings/info → ⚠️ WARN (acceptable, list count)

## CHECK 2: 🧪 Test Suite (GATE — must pass)

```bash
cd "$SOURCE"
flutter test 2>&1
```

**Criteria:**
- All tests pass → ✅ PASS
- Any test fails → ❌ FAIL (list failures)
- No tests exist → ⚠️ WARN (acceptable for now, note in report)

## CHECK 3: 📦 Import Health

```bash
# Check for broken imports (files that import non-existent files)
grep -rn "^import 'package:flutter_kfsp_app/" "$SOURCE/lib/" --include="*.dart" | while IFS=: read -r file line content; do
  imported_path=$(echo "$content" | grep -oP "import 'package:flutter_kfsp_app/(.*?)'" | sed "s/import 'package:flutter_kfsp_app\///;s/'//")
  if [ -n "$imported_path" ] && [ ! -f "$SOURCE/lib/$imported_path" ]; then
    echo "❌ BROKEN IMPORT: $file:$line → $imported_path"
  fi
done
```

## CHECK 4: 🎨 Design Token Compliance

```bash
# Hardcoded colors in feature files (excluding token definition files)
HC_COLORS=$(grep -rn "Color(0x" "$SOURCE/lib/" --include="*.dart" | grep -v "kfsp_colors.dart\|_test.dart\|app_theme.dart\|kfsp_theme" | wc -l | tr -d ' ')

# Hardcoded spacing
HC_SPACING=$(grep -rn "EdgeInsets\.\(all\|only\|symmetric\)([0-9]" "$SOURCE/lib/" --include="*.dart" | grep -v "kfsp_spacing.dart\|_test.dart" | wc -l | tr -d ' ')

echo "Hardcoded colors: $HC_COLORS, spacing: $HC_SPACING"
```

**Criteria:**
- New hardcoded values in changed files → ⚠️ WARN
- Accept existing tech debt (tracked separately)

## CHECK 5: 🌙 Dark Mode Coverage

```bash
# Files with Colors.white or Colors.black (potential dark mode issues)
DARK_ISSUES=$(grep -rn "Colors\.white\|Colors\.black" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart\|kfsp_colors\|app_theme\|//.*Colors" | wc -l | tr -d ' ')
echo "Potential dark mode issues: $DARK_ISSUES"
```

## CHECK 6: 🔗 Route Integrity

```bash
# All routes resolve to existing widget classes
grep -n "builder:\|pageBuilder:" "$SOURCE/lib/"**/app_router.dart 2>/dev/null | head -30
# Check: no builder references a non-existent screen
```

## CHECK 7: 📱 Socket/API Integration (REAL DATA MODE)

```bash
# Verify socket lifecycle in features that use it
grep -rn "SocketClient\|emitWithAck\|emit.*join\|emit.*leave" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart"
# Check: every join has matching leave
# Check: dispose methods exist
```

## CHECK 8: 🏗️ Build (only --full mode)

```bash
cd "$SOURCE"
flutter build ios --debug --simulator 2>&1
```

**Criteria:**
- Build succeeds → ✅ PASS
- Build fails → ❌ FAIL (agent must fix)

## Output Format

```markdown
# ✅ KFSP Build Verification Report
**Date:** [timestamp]
**Source:** [path]
**Mode:** Quick / Default / Full

## Gate Results

| # | Check | Status | Details |
|---|-------|--------|---------|
| 1 | 🔍 Static Analysis | ✅/❌ | X errors, Y warnings |
| 2 | 🧪 Test Suite | ✅/❌/⚠️ | X/Y tests pass |
| 3 | 📦 Import Health | ✅/❌ | X broken imports |
| 4 | 🎨 Design Tokens | ✅/⚠️ | X hardcoded colors, Y spacing |
| 5 | 🌙 Dark Mode | ✅/⚠️ | X potential issues |
| 6 | 🔗 Route Integrity | ✅/❌ | X orphan routes |
| 7 | 📱 Real Data | ✅/⚠️ | Socket lifecycle OK/issues |
| 8 | 🏗️ Build (full) | ✅/❌ | Build time: Xs |

## Overall: [PASS ✅ / FAIL ❌]

### If FAIL:
**Blockers to fix before PM test:**
1. [error 1] — fix: [suggestion]
2. [error 2] — fix: [suggestion]

### If PASS:
**Ready for PM testing.** Run `/kfsp:test-brief` to generate test checklist.
```

## CHECK 9: 🎬 Feature Completeness — 6 Dimensions (2026-03-13+)

For each NEW or MODIFIED feature in this build, verify:

```bash
# Check: Animation present (transitions, loading states)
grep -rn "AnimatedContainer\|SlideTransition\|FadeTransition\|shimmer\|CircularProgressIndicator" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart"

# Check: Actions present (GestureDetector, onTap, navigation)
grep -rn "onTap\|onPressed\|GestureDetector\|context\.go\|context\.push" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart"

# Check: Error handling (try/catch, error states)
grep -rn "catch\|error\|ErrorWidget\|retry" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart"
```

**6 Dimensions Checklist:**
- [ ] Animation — transitions, loading shimmer, feedback visual (NOT static)
- [ ] Action — tap, drill-down, navigate, breadcrumb, swipe handlers
- [ ] Logic — business rules, edge cases, error/empty/loading states, Pro/Free gate
- [ ] Customer Insight — feature solves what user pain point?
- [ ] Value — Pro vs Free differentiation clear?
- [ ] UX/UI Test — dark/light mode, device test, design parity

## CHECK 10: 🇻🇳 Vietnamese Language — Diacritics (2026-03-13+)

```bash
# Scan for Vietnamese text WITHOUT diacritics (common patterns)
grep -rn "'Tat ca'\|'Ngan hang'\|'Co phieu'\|'Nganh'\|'Khoi luong'\|'Gia tri'\|'Von hoa'\|'Dinh gia'\|'Bieu do'\|'Thi truong'" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart\|// \|///\|heat_map_model.dart"
```

**Criteria:**
- User-facing text (labels, titles, buttons, tooltips) MUST use Vietnamese WITH diacritics
- Code comments/variable names: no-diacritics OK
- API field names: no-diacritics OK (server-defined)
- Enum labels shown to users → MUST have diacritics

## CHECK 11: 🔢 Number Formatting — Dấu chấm/Phẩy (2026-03-15+)

**Gate:** ⚠️ Warning — must review before shipping financial data.

**Rule:** Tất cả số hiển thị PHẢI dùng:
- Dấu `.` cho thập phân (1.56%)
- Dấu `,` cho phần nghìn (1,000,000)
- Kiểm tra như chính tả tiếng Việt có dấu — bắt buộc trước khi ship

**Check:**
```bash
# Tìm toStringAsFixed dùng cho số lớn (nên dùng NumberFormat)
grep -rn "toStringAsFixed" lib/ --include="*.dart" | grep -v "_test.dart" | grep -v "perChange\|percent"

# Tìm số hiển thị thiếu NumberFormat
grep -rn "\.toString()" lib/ --include="*.dart" | grep -v "_test.dart" | grep -v "// "
```

**Criteria:**
- Dùng `NumberFormat` từ package `intl` — KHÔNG dùng `toStringAsFixed` cho số lớn (giá, KL, GT, vốn hoá)
- `toStringAsFixed` CHỈ OK cho phần trăm nhỏ (perChange, percent)
- Ví dụ đúng: `26,000` (không phải `26000`), `1,572.6 Tỷ` (không phải `1572.6 Tỷ`)
- Phải hỏi Thanh confirm đơn vị khi không chắc chắn (Triệu/Tỷ/nghìn đồng/%)
- Áp dụng cho: hiển thị (labels, cards, charts), input fields, placeholder text, live rendering

## CHECK 12: 📋 Test Case Coverage (2026-03-15+)

**Gate:** ⚠️ Warning — modified screens MUST have test cases in Master Test Registry.

```bash
REGISTRY="Product/kfsp_flutter_migration/docs/test_cases/master_test_registry.html"
if [ -f "$REGISTRY" ]; then
  TC_COUNT=$(grep -c '"id":' "$REGISTRY" 2>/dev/null || echo "0")
  echo "📋 Master Test Registry: $TC_COUNT test cases"
else
  echo "⚠️ Master Test Registry NOT FOUND"
fi
```

**Check per modified screen:**
For each screen/feature modified in this build:
1. Find test cases with matching screen prefix (e.g., `MK-` for Market, `HM-` for Heatmap)
2. If no test cases exist → ⚠️ FLAG: "Screen [X] modified but has 0 test cases"
3. If test cases exist but are outdated → ⚠️ FLAG: "Test cases may need updating"

**Output row:**
| 12 | 📋 Test Coverage | ✅/⚠️ | X screens covered, Y missing |

## CHECK 13: 📋 Project Charter Alignment (2026-03-15+)

**Gate:** ⚠️ Warning — build deliverables SHOULD map to Project Charter.

```bash
PHASE=$(grep -r "Phase hiện tại" memory/flutter-migration.md 2>/dev/null | grep -oP "P\d+" | head -1)
CHARTER="Product/kfsp_flutter_migration/docs/build_reports/${PHASE}_project_charter.md"
if [ -f "$CHARTER" ]; then
  echo "✅ Charter found — verify deliverables match"
  grep -A 20 "Deliverables" "$CHARTER" | head -20
else
  echo "⚠️ No Project Charter — build not mapped to project goals"
fi
```

## CHECK 14: 📔 Dev Journal Entries (2026-03-15+)

**Gate:** ⚠️ Warning — code changes SHOULD have corresponding journal entries.

```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
MONTH=$(date +%Y-%m)
ENTRIES=$(find "$JOURNAL_DIR/$MONTH/" -name "*.md" 2>/dev/null | wc -l | tr -d ' ')
echo "📔 Dev journal entries (${MONTH}): $ENTRIES"
[ "$ENTRIES" -eq 0 ] && echo "⚠️ No journal entries — decisions not documented"
```

**Severity:** ⚠️ Warning — nhắc agent ghi journal, không block build.

## CHECK 15: 🧪 Unit Test First — TDD Gate (2026-03-19+)

**Gate:** ❌ FAIL nếu feature mới KHÔNG có unit test.

**Rule:** Mọi feature/widget MỚI hoặc SỬA ĐỔI PHẢI có unit test TRƯỚC KHI dev.

**Workflow bắt buộc:**
```
1. Viết test (expected behavior) → test FAIL (chưa có code)
2. Viết code → test PASS
3. Refactor → test vẫn PASS
```

**Check:**
```bash
# Đếm test files vs source files cho features đã thay đổi
TESTS=$(find "$SOURCE/test" -name "*_test.dart" 2>/dev/null | wc -l | tr -d ' ')
SRC=$(find "$SOURCE/lib/screens" -name "*.dart" 2>/dev/null | wc -l | tr -d ' ')
echo "🧪 Tests: $TESTS files, Source: $SRC files (ratio: $(echo "scale=1; $TESTS * 100 / $SRC" | bc)%)"
```

**Criteria:**
- Feature mới PHẢI có ít nhất 1 test file tương ứng
- Widget test cho UI, unit test cho logic/model/provider
- Agent KHÔNG ĐƯỢC báo "hoàn thành" feature mà chưa có test
- Test phải cover: happy path + edge cases (null, empty, error)

## CHECK 16: 📦 Build Deliverable Gate (2026-03-19+)

**Gate:** ❌ FAIL nếu build KHÔNG có deliverable cho PM.

**Rule:** Mọi build PHẢI có deliverable rõ ràng giao cho PM, bao gồm:

1. **Bảng thay đổi** — liệt kê CỤ THỂ đã làm gì (files, widgets, logic)
2. **Screenshot/evidence** — chụp simulator hoặc mô tả kết quả
3. **So sánh trước/sau** — nếu sửa UI: before vs after
4. **Bugs đã biết** — liệt kê rõ, không giấu
5. **Cần test gì** — chỉ rõ cho PM biết cần kiểm tra mục nào

**Agent KHÔNG ĐƯỢC:**
- Build xong rồi hỏi "anh muốn em làm gì tiếp"
- Build xong chỉ nói "build thành công" mà không giao deliverable
- Chờ PM hỏi mới báo cáo

**Agent PHẢI:**
- TỰ ĐỘNG giao deliverable ngay sau build
- Format rõ ràng: bảng, bullet points, screenshots
- Ghi rõ "Cần anh test: ..." ở cuối

## CHECK 17: 📋 Spec Compliance — So sánh Build vs Plan (2026-03-20+)

**Gate:** ⚠️ Warning — mọi build PHẢI được so sánh với plan/spec gốc.

**Vấn đề giải quyết:** Agent build xong nhưng KHÔNG AI so sánh kết quả với spec ban đầu.
Deliverables bị thiếu, bị lệch, hoặc làm thừa mà không ai biết.

**Quy trình:**
```
PLAN/SPEC (source of truth)
    ↓
Liệt kê deliverables kỳ vọng (từ plan)
    ↓
So sánh với deliverables thực tế (từ code/git)
    ↓
Spec Compliance Report
```

**Check:**
```bash
# 1. Tìm plan file gần nhất
PLAN=$(find .planning/ -name "PLAN.md" -path "*phase*" 2>/dev/null | sort -r | head -1)
if [ -z "$PLAN" ]; then
  PLAN=$(find . -name "*plan*.md" -o -name "*charter*.md" 2>/dev/null | sort -r | head -1)
fi

if [ -n "$PLAN" ]; then
  echo "📋 Plan found: $PLAN"
  # Extract deliverables/tasks from plan
  grep -E "^\s*-\s*\[" "$PLAN" 2>/dev/null | head -20
  echo "---"
  # Compare with git changes
  echo "📦 Files changed in this build:"
  git diff --name-only HEAD~5..HEAD -- lib/ 2>/dev/null | head -30
else
  echo "⚠️ No plan file found — build not mapped to spec"
fi
```

**Spec Compliance Report format:**
```markdown
## 📋 Spec Compliance Report

| # | Plan Item | Status | Evidence |
|---|-----------|--------|----------|
| 1 | [deliverable từ plan] | ✅ Done / ⚠️ Partial / ❌ Missing | [file hoặc commit] |
| 2 | ... | ... | ... |

**Coverage:** X/Y items completed (Z%)
**Unplanned work:** [liệt kê việc làm thêm ngoài plan]
**Missing:** [liệt kê deliverables trong plan nhưng chưa làm]
```

**Criteria:**
- Coverage ≥ 80% → ✅ PASS
- Coverage 50-79% → ⚠️ WARN (liệt kê missing items)
- Coverage < 50% → ❌ FAIL (build không đáp ứng spec)
- Có unplanned work → ⚠️ NOTE (ghi nhận, không block)

**Cross-domain translation:**
- Product dev: "Plan item" = feature/screen/widget từ GSD plan
- Marketing: "Plan item" = campaign element/content piece từ brief
- Sales: "Plan item" = qualification criteria từ sales playbook
- HR: "Plan item" = process step từ hiring plan

## Integration Rules

1. **GATE:** Build verify MUST pass before creating test-brief
2. **Auto-trigger:** Chạy sau mỗi rsync (before build on Desktop)
3. **Quick mode:** Dùng trong development loop (sau mỗi code change nhỏ)
4. **Full mode:** Dùng trước khi giao PM (build + verify)
5. **Chain:** `build-verify → [PASS] → test-brief → bug-log`
6. **Feature completeness:** Every feature must pass 6-dimension check (animation, action, logic, insight, value, UX)
7. **Vietnamese text:** All user-facing text must use proper diacritics
8. **Simulator testing:** Agent CANNOT self-navigate iOS Simulator. After build+install: ASK Thanh to tap/navigate → screenshot → verify → send self-test report
9. **Git remote safety:** NEVER push/pull/rename any git remote without Thanh's explicit permission. NEVER assume GitHub = GitLab. Always confirm: remote name + URL + branch → wait for OK
10. **Unit Test First:** Mọi feature mới PHẢI có unit test TRƯỚC KHI viết code. Test → Code → Refactor.
11. **Build Deliverable:** Mọi build PHẢI tự động giao deliverable cho PM — bảng thay đổi, screenshots, bugs, cần test gì. KHÔNG chờ PM hỏi.
</instructions>
