---
name: kfsp:pre-commit
description: "Pre-commit 5-tier validation gate — 23 checks across Safety, Code, QA, UX, Business tiers. MUST pass before any commit."
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - TodoWrite
---
<objective>
KFSP Pre-Commit Gate — Bộ kiểm tra 5 tầng (23 checks) TRƯỚC MỌI commit.

Mô hình 5 tầng bảo vệ:
```
T5 🎯 Business  — Docs, deliverable, plan, feature completeness
T4 👤 UX        — Vietnamese diacritics, number formatting, UX parity
T3 🧪 QA        — Unit test, flutter test, test ratio, build-verify
T2 🔧 Code      — Design system, secrets, imports, file size, sweep, sync
T1 🛡️ Safety    — Dev journal, guard score, git remote
```

**Gate:** PASS/FAIL. BLOCK items → PHẢI fix trước khi commit. WARN items → commit OK nhưng phải lên kế hoạch fix.

## ⚠️ Checklist Protocol (BẮT BUỘC)

Agent PHẢI thực hiện 2 bước khi chạy pre-commit:

### Bước 1 — TRƯỚC KHI chạy: Gửi PM bảng checklist + expected results
```
📋 Pre-commit Checklist — [mô tả commit]
| Tier | # | Hạng mục | Expected |
|------|---|----------|----------|
| T1 🛡️ | 1 | Dev journal entries | ≥1 entry |
| T1 🛡️ | 2 | Guard score | ≥70/100 |
| T1 🛡️ | 3 | Git remote correct | PASS |
| T2 🔧 | 4 | Design system (NEW violations) | 0 new |
| T2 🔧 | 5 | Secrets & credentials | 0 |
| T2 🔧 | 6 | File size | <500 lines |
| T2 🔧 | 7 | Import validation | PASS |
| T2 🔧 | 8 | Conventions | PASS |
| T2 🔧 | 9 | Sweep ran (blast radius) | Done |
| T2 🔧 | 10 | Sync-check (source copies) | In sync |
| T3 🧪 | 11 | Unit test coverage | ≥80% logic files |
| T3 🧪 | 12 | Flutter test pass | ALL PASS |
| T3 🧪 | 13 | Test-code ratio | ≥0.3 |
| T3 🧪 | 14 | Build-verify | PASS (if built) |
| T4 👤 | 15 | Vietnamese diacritics | 0 violations |
| T4 👤 | 16 | Number formatting | PASS |
| T4 👤 | 17 | UX parity | Done (if UI changed) |
| T5 🎯 | 18 | Docs updated | All affected updated |
| T5 🎯 | 19 | Feature completeness 6D | PASS (if new feature) |
| T5 🎯 | 20 | Build report / deliverable | Created |
| T5 🎯 | 21 | Plan file updated | In sync |
| T5 🎯 | 22 | Changelog vs preprod | Generated |
| T5 🎯 | 23 | Legacy violations trend | Documented |
```

### Bước 2 — SAU KHI chạy: Lôi lại bảng + điền kết quả thực tế
```
✅/❌ Pre-commit Results — [mô tả commit]
| Tier | # | Hạng mục | Expected | Actual | Status |
|------|---|----------|----------|--------|--------|
| T1 🛡️ | 1 | Dev journal | ≥1 | 3 entries | ✅ PASS |
| T1 🛡️ | 2 | Guard score | ≥70 | 85/100 | ✅ PASS |
| ... |
→ Tổng: X/23 PASS, Y WARN, Z BLOCK
→ Verdict: ✅ OK to commit / ❌ BLOCK — fix N items first
```

Agent KHÔNG ĐƯỢC bỏ qua bước 1 hoặc bước 2. PM cần thấy CẢ HAI bảng.
</objective>

<instructions>
## Determine Source Path

Check which Flutter source exists (in order):
1. `Product/kfsp_flutter_migration/current_source/flutter_kfsp_app/` (git repo — preferred)
2. `~/Desktop/kfsp_flutter_dev/` (build copy)

Use whichever is the git repo. If neither → error.

---

## T1 🛡️ SAFETY TIER (3 checks)

### CHECK 1: 📔 Dev Journal Entries

**Severity:** ⚠️ WARN

Kiểm tra session hiện tại có ghi dev journal cho decisions/incidents không.

```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
MONTH=$(date +%Y-%m)
TODAY=$(date +%Y-%m-%d)

# Count entries this month
ENTRIES_MONTH=$(find "$JOURNAL_DIR/$MONTH/" -name "*.md" 2>/dev/null | wc -l | tr -d ' ')

# Count entries today
ENTRIES_TODAY=$(find "$JOURNAL_DIR/$MONTH/" -name "${TODAY}*" 2>/dev/null | wc -l | tr -d ' ')

echo "📔 Dev journal: $ENTRIES_TODAY entries today, $ENTRIES_MONTH this month"

# If committing significant changes, should have decision entry
CHANGED_FILES=$(git diff --name-only HEAD | wc -l | tr -d ' ')
if [ "$CHANGED_FILES" -gt 5 ] && [ "$ENTRIES_TODAY" -eq 0 ]; then
  echo "⚠️ WARN: Committing $CHANGED_FILES files but 0 journal entries today — document decisions!"
fi
```

**Rule:** Mỗi commit có >5 files → nên có ít nhất 1 dev journal entry ghi lại decisions.

### CHECK 2: 🛡️ Guard Score

**Severity:** ⚠️ WARN (score <70 → flag, không block commit nhưng cảnh báo mạnh)

Agent PHẢI đã chạy `/kfsp:guard` (hoặc guard --quick) TRƯỚC khi pre-commit.

Kiểm tra:
- Guard đã chạy trong session này chưa? (hỏi agent tự report)
- Score cuối cùng bao nhiêu /100?
- Nếu <70 → ⚠️ WARN: "Guard score thấp — có risk chưa giải quyết"
- Nếu chưa chạy → ⚠️ WARN: "Guard chưa chạy — khuyến nghị chạy trước commit"

**Agent self-report:** Liệt kê score + critical findings từ lần guard gần nhất.

### CHECK 3: 🔒 Git Remote Verification

**Severity:** 🔴 BLOCK

```bash
echo "=== Git Remote Check ==="
REMOTE_URL=$(git remote get-url origin 2>/dev/null)
echo "Remote origin: $REMOTE_URL"

# Verify correct repo
if echo "$REMOTE_URL" | grep -q "gitlab.com/phudh3/flutter_kfsp_app"; then
  echo "✅ Correct: GitLab flutter_kfsp_app"
elif echo "$REMOTE_URL" | grep -q "github.com/nguyenthanhcllhp/kfsp-skill-set"; then
  echo "✅ Correct: GitHub kfsp-skill-set"
else
  echo "🔴 BLOCK: Unexpected remote URL! Ask Thanh before push."
fi

# Check branch
BRANCH=$(git branch --show-current)
echo "Branch: $BRANCH"
```

**Rule:** Remote origin PHẢI là expected URL. Sai URL → 🔴 BLOCK. CẤM push khi Thanh chưa cho phép.

---

## T2 🔧 CODE INTEGRITY TIER (7 checks)

### CHECK 4: 🎨 Design System Compliance (GATE)

**Severity:** 🔴 BLOCK on NEW violations

#### Token Reference Files (8 files)
- `kfsp_colors.dart` — colors, radius, font sizes
- `kfsp_text_styles.dart` — TextStyle definitions
- `kfsp_spacing.dart` — EdgeInsets, padding, margin, gap
- `kfsp_animations.dart` — Duration, Curve values
- `kfsp_shadows.dart` — BoxShadow, elevation
- `kfsp_buttons.dart` — ButtonStyle definitions
- `kfsp_components.dart` — Component-level decorations (card, chip, dropdown, etc.)
- `kfsp_theme_colors.dart` — Light/dark semantic colors

#### Step 1: Get changed files
```bash
CHANGED_DART=$(git diff --name-only HEAD -- '*.dart' | grep -E "^lib/(screens|common|features|widgets)/" | grep -v "_test.dart")
if [ -z "$CHANGED_DART" ]; then
  echo "✅ No widget/screen Dart files changed — skip design system check"
fi
```

#### Step 2: Compare BEFORE vs AFTER violation count
```bash
for f in $CHANGED_DART; do
  BEFORE_COLORS=$(git show HEAD:"$f" 2>/dev/null | grep -c "Color(0x" || echo "0")
  AFTER_COLORS=$(cat "$f" | grep -c "Color(0x" || echo "0")
  DELTA=$((AFTER_COLORS - BEFORE_COLORS))
  [ "$DELTA" -gt 0 ] && echo "🔴 BLOCK: $f — +$DELTA new Color(0x (use KfspColors.*)"
done
```

(Repeat for Colors.*, fontSize:, EdgeInsets.*, BoxShadow, Duration(milliseconds:, BorderRadius.circular)

#### Blocking vs Warning Rules

| Pattern | Verdict |
|---------|---------|
| NEW `Color(0xFF...)` in screens/widgets | 🔴 BLOCK |
| NEW `Colors.*` (except transparent) | 🔴 BLOCK |
| NEW `fontSize:` with raw number | 🔴 BLOCK |
| NEW `EdgeInsets.*` with raw numbers | 🔴 BLOCK |
| NEW `BoxShadow(...)` hardcoded | 🔴 BLOCK |
| NEW `Duration(milliseconds: N)` | 🔴 BLOCK |
| NEW `BorderRadius.circular(N)` where N matches a token | 🔴 BLOCK |
| EXISTING (count unchanged/decreased) | ⚠️ WARN legacy |
| In `lib/theme/`, `lib/constants/` | ✅ ALLOWED (token definitions) |
| In `_test.dart`, `generated/`, `l10n/` | ✅ ALLOWED |

### CHECK 5: 🔒 Secrets & Credentials

**Severity:** 🔴 BLOCK

```bash
grep -rni "api_key\|apikey\|secret\|password\|bearer" lib/ --include="*.dart" | grep -v "//\|hint\|label\|AutofillHints"
grep -rn "\.env" lib/ --include="*.dart"
grep -rn "localhost\|127\.0\.0\.1\|192\.168\." lib/ --include="*.dart" | grep -v "env_config.dart"
```

### CHECK 6: 📏 File Size

**Severity:** ⚠️ WARN (>500), 🔴 BLOCK (>1000 for new files)

```bash
for f in $(git diff --name-only HEAD | grep '\.dart$'); do
  [ ! -f "$f" ] && continue
  lines=$(wc -l < "$f")
  [ "$lines" -gt 1000 ] && echo "🔴 $f: $lines lines (>1000)"
  [ "$lines" -gt 500 ] && [ "$lines" -le 1000 ] && echo "⚠️ $f: $lines lines (>500)"
done
```

### CHECK 7: 📦 Import Validation

**Severity:** 🔴 BLOCK on broken imports

```bash
grep -rn "^import.*package:flutter_kfsp_app" lib/ --include="*.dart" | while read line; do
  file=$(echo "$line" | sed "s/.*package:flutter_kfsp_app\//lib\//;s/'.*//" | sed "s/\";.*//")
  [ ! -f "$file" ] && echo "❌ Broken import: $line"
done
```

### CHECK 8: 🏗️ Conventions

**Severity:** ℹ️ INFO

- Widget files có suffix đúng (Widget/Screen/Page/Card/...)
- Provider files ở đúng folder
- Naming conventions (snake_case files, PascalCase classes)

### CHECK 9: 🔍 Sweep Ran (Blast Radius)

**Severity:** ⚠️ WARN

Agent PHẢI tự report:
- Sweep đã chạy trong session này chưa?
- Findings summary: X critical, Y warnings
- Blast radius: files affected by changes

**Rule:** Nếu thay đổi >3 files mà chưa chạy sweep → ⚠️ WARN. Nếu sweep phát hiện critical → cần address trước commit.

### CHECK 10: 🔄 Sync-Check (Source Copies)

**Severity:** ⚠️ WARN

Kiểm tra 3 bản copy đồng bộ:
```bash
# Git repo (source of truth)
SRC="Product/kfsp_flutter_migration/current_source/flutter_kfsp_app"

# Build copy
BUILD="$HOME/Desktop/kfsp_flutter_dev"

# Quick check: compare file counts
SRC_COUNT=$(find "$SRC/lib" -name "*.dart" | wc -l | tr -d ' ')
BUILD_COUNT=$(find "$BUILD/lib" -name "*.dart" 2>/dev/null | wc -l | tr -d ' ')

echo "Source: $SRC_COUNT files | Build: $BUILD_COUNT files"
[ "$SRC_COUNT" != "$BUILD_COUNT" ] && echo "⚠️ Out of sync! Run rsync before build."
```

---

## T3 🧪 QA TIER (4 checks)

### CHECK 11: 🧪 Unit Test Coverage

**Severity:** 🔴 BLOCK (logic files) / ⚠️ WARN (widget/token files)

Map changed `lib/*.dart` → expected `test/*_test.dart`:

| Source pattern | Severity nếu thiếu test |
|----------------|------------------------|
| `*model*.dart`, `*provider*.dart`, `*controller*.dart`, `*service*.dart` | 🔴 BLOCK |
| `*util*.dart`, `*helper*.dart`, `*repository*.dart`, `*notifier*.dart` | 🔴 BLOCK |
| `*_screen.dart`, `*_page.dart`, `*_widget.dart`, `*_card.dart` | ⚠️ WARN |
| `*theme*.dart`, `*colors*.dart`, `*style*.dart` | ⚠️ WARN |
| `generated/`, `l10n/` | ✅ SKIP |

```bash
for f in $(git diff --name-only HEAD | grep "^lib/.*\.dart$" | grep -v "generated/"); do
  base=$(basename "$f" .dart)
  has_test=$(find test/ -name "${base}_test.dart" -o -name "${base}_helpers_test.dart" 2>/dev/null | head -1)

  if [ -z "$has_test" ]; then
    if echo "$f" | grep -qE "(model|provider|controller|service|util|helper|repository|notifier)"; then
      echo "🔴 BLOCK — no test: $f"
    else
      echo "⚠️ WARN — no test: $f"
    fi
  else
    echo "✅ $f → $has_test"
  fi
done
```

### CHECK 12: 🧪 Flutter Test Pass

**Severity:** 🔴 BLOCK

```bash
flutter test --no-pub 2>&1
# Exit 0 = PASS, non-zero = FAIL
# FAIL → BLOCK commit, agent PHẢI fix
```

**Rule:** 100% tests PHẢI pass. 0 failures allowed.

### CHECK 13: 🧪 Test-Code Ratio

**Severity:** ⚠️ WARN

```bash
LIB_LINES=$(find lib/ -name "*.dart" ! -path "*/generated/*" -exec cat {} + 2>/dev/null | wc -l)
TEST_LINES=$(find test/ -name "*.dart" -exec cat {} + 2>/dev/null | wc -l)
RATIO=$(awk "BEGIN {printf \"%.2f\", $TEST_LINES / ($LIB_LINES > 0 ? $LIB_LINES : 1)}")
echo "Ratio: $RATIO (target ≥0.3)"
```

| Target | Loại file |
|--------|-----------|
| ≥ 0.5 | Logic (models, providers, utils) |
| ≥ 0.2 | Widgets/screens |
| ≥ 0.3 | Overall project |

### CHECK 14: ✅ Build-Verify Status

**Severity:** ⚠️ WARN (if building this session)

Agent PHẢI tự report:
- Build-verify đã chạy chưa? (chỉ cần nếu có build trong session)
- Kết quả: flutter analyze (0 errors?), flutter test (pass?), runtime checks?
- Nếu chưa build → ghi "N/A — no build this session"

---

## T4 👤 UX TIER (3 checks)

### CHECK 15: 🇻🇳 Vietnamese Diacritics

**Severity:** 🔴 BLOCK

```bash
# Tìm user-facing strings tiếng Việt không dấu
git diff HEAD -- '*.dart' | grep '^+' | grep -v '^+++' | \
  grep -iE "'(Tat ca|Ngan hang|Co phieu|Khong co|Thu lai|Dang nhap|Dang ky|Bieu do|Thi truong|Dong tien|Danh muc|Canh bao|Ghi chu|Tim kiem|Xem them|Huy|Luu|Xoa|Sua|Them|Loc|Sap xep)'"
```

**Rule:** TẤT CẢ text user-facing PHẢI có dấu đầy đủ. Không chấp nhận "Tat ca", "Co phieu".

### CHECK 16: 🔢 Number Formatting

**Severity:** ⚠️ WARN

```bash
# Tìm toStringAsFixed cho số lớn (nên dùng NumberFormat)
grep -rn "toStringAsFixed" lib/ --include="*.dart" | grep -v "_test.dart\|perChange\|percent"

# Tìm hiển thị số thiếu formatting
grep -rn "\.toString()" lib/ --include="*.dart" | grep -v "_test.dart\|debug\|log\|print"
```

**Rule:** Dấu `.` = thập phân, dấu `,` = phần nghìn. Dùng `NumberFormat` từ `intl` package.

### CHECK 17: 🎨 UX Parity Status

**Severity:** ⚠️ WARN (if UI changed)

Agent PHẢI tự report:
- Có thay đổi UI trong commit này không?
- Nếu CÓ → đã chạy `ux-parity` chưa? Kết quả?
- Nếu CÓ → đã compare với Figma/HTML prototype chưa?
- Nếu KHÔNG có UI change → ghi "N/A"

**Rule:** Mọi thay đổi UI PHẢI được verify parity với design trước khi commit.

---

## T5 🎯 BUSINESS TIER (6 checks)

### CHECK 18: 📝 Docs Updated

**Severity:** ⚠️ WARN

Kiểm tra docs bị ảnh hưởng bởi code change:

```bash
echo "=== Doc sync check ==="

# Check if any of these docs need updating based on changed files
DOCS_DIR="Product/kfsp_flutter_migration/docs"

# Feature specs
CHANGED_SCREENS=$(git diff --name-only HEAD | grep "lib/screens/" | head -5)
[ -n "$CHANGED_SCREENS" ] && echo "📝 Screens changed — check 04_FEATURE_SPECS.md"

# Design tokens
CHANGED_TOKENS=$(git diff --name-only HEAD | grep "lib/core/theme/")
[ -n "$CHANGED_TOKENS" ] && echo "📝 Tokens changed — check 03_DESIGN_SYSTEM.md"

# Architecture
CHANGED_CORE=$(git diff --name-only HEAD | grep "lib/core/" | grep -v "theme/")
[ -n "$CHANGED_CORE" ] && echo "📝 Core changed — check 02_ARCHITECTURE_GUIDE.md"

# Always check handoff status
echo "📝 Always check: 08_HANDOFF_STATUS.md — progress bars correct?"
```

**Checklist tài liệu bắt buộc:**
- [ ] `docs/08_HANDOFF_STATUS.md` — progress bars, what works, bugs
- [ ] `docs/04_FEATURE_SPECS.md` — features list matches code
- [ ] `docs/03_DESIGN_SYSTEM.md` — token values match code
- [ ] `docs/06_DEBUG_LOG.md` — decisions/incidents ghi lại
- [ ] `docs/plans/P{N}_{TÊN}.md` — plan file task status cập nhật
- [ ] `docs/build_reports/` — build report tồn tại (nếu deliverable)
- [ ] Memory files — sync với file plan

**Rule:** Agent PHẢI liệt kê cụ thể docs nào đã cập nhật. "Không ảnh hưởng" PHẢI chứng minh bằng cách ghi rõ đã check docs nào.

### CHECK 19: 🏗️ Feature Completeness 6D

**Severity:** ⚠️ WARN (if new feature)

Nếu commit bao gồm feature MỚI, kiểm tra 6 chiều:

| # | Chiều | Có? | Ghi chú |
|---|-------|-----|---------|
| 1 | Animation | ✅/❌ | Transition, loading, feedback |
| 2 | Action | ✅/❌ | Tap, swipe, navigate handlers |
| 3 | Logic | ✅/❌ | Edge cases, error handling, offline |
| 4 | Customer Insight | ✅/❌ | Giải quyết pain point gì? |
| 5 | Value | ✅/❌ | Pro vs Free differentiation? |
| 6 | UX/UI Test | ✅/❌ | Compare vs design, dark/light |

**Rule:** Feature thiếu bất kỳ chiều nào → ⚠️ WARN. Agent PHẢI bổ sung hoặc ghi nhận thiếu trong commit message.

Nếu KHÔNG có feature mới → ghi "N/A — no new feature in this commit"

### CHECK 20: 📋 Build Report / Deliverable

**Severity:** ⚠️ WARN

Agent PHẢI tự report:
- Đã tạo build report chưa? (`docs/build_reports/YYYY-MM-DD_P{N}_{mô-tả}.md`)
- Build report có đủ 9 sections bắt buộc?
  1. Goals (từ Charter)
  2. Deliverables (trạng thái)
  3. Success Criteria (kết quả)
  4. Files Changed
  5. Technical Decisions
  6. 🧪 Test Checklist cho PM
  7. 🥋 Skills đã chạy
  8. Known Issues
  9. Next Steps
- Đã giao deliverable cho PM chưa? (bảng thay đổi, screenshots, cần test gì)

**Rule:** KHÔNG commit "work in progress" mà chưa có deliverable plan.

### CHECK 21: 📋 Plan File Updated

**Severity:** ⚠️ WARN

```bash
PLANS_DIR="Product/kfsp_flutter_migration/docs/plans"
LATEST_PLAN=$(ls -t "$PLANS_DIR/"*.md 2>/dev/null | head -1)

if [ -n "$LATEST_PLAN" ]; then
  echo "📋 Latest plan: $(basename $LATEST_PLAN)"
  # Check if plan has been modified recently (within last 2 hours)
  PLAN_MOD=$(stat -f "%m" "$LATEST_PLAN" 2>/dev/null || stat -c "%Y" "$LATEST_PLAN" 2>/dev/null)
  NOW=$(date +%s)
  DIFF=$(( (NOW - PLAN_MOD) / 3600 ))
  [ "$DIFF" -gt 2 ] && echo "⚠️ Plan not updated in ${DIFF}h — sync task status!"
else
  echo "⚠️ No plan file found in $PLANS_DIR — create one!"
fi
```

**Rule:** Source of truth cho tasks là file plan trong `docs/plans/`. PHẢI sync với memory. File plan wins over memory if conflict.

### CHECK 22: 📋 Changelog vs Preprod

**Severity:** ℹ️ INFO

```bash
BASE_BRANCH="preprod"
git rev-parse --verify "$BASE_BRANCH" 2>/dev/null || BASE_BRANCH="main"

echo "📋 Changelog (${BASE_BRANCH}..HEAD):"
git log ${BASE_BRANCH}..HEAD --oneline --no-merges 2>/dev/null | head -20

echo ""
echo "📁 Files changed:"
ADDED=$(git diff ${BASE_BRANCH}..HEAD --diff-filter=A --name-only 2>/dev/null | wc -l | tr -d ' ')
MODIFIED=$(git diff ${BASE_BRANCH}..HEAD --diff-filter=M --name-only 2>/dev/null | wc -l | tr -d ' ')
DELETED=$(git diff ${BASE_BRANCH}..HEAD --diff-filter=D --name-only 2>/dev/null | wc -l | tr -d ' ')
echo "  Added: $ADDED | Modified: $MODIFIED | Deleted: $DELETED"
```

**Khi merge preprod:** PHẢI chạy convention check trên TẤT CẢ files từ dev khác → báo cáo violations → đề xuất fix plan.

### CHECK 23: 📊 Legacy Violations Trend

**Severity:** ℹ️ INFO

```bash
EXCLUDE="kfsp_colors.dart\|kfsp_text_styles.dart\|kfsp_spacing.dart\|kfsp_animations.dart\|kfsp_shadows.dart\|kfsp_buttons.dart\|kfsp_components.dart\|_test.dart\|\.g\.dart\|theme/\|constants/"

HC_COLORS=$(grep -rn "Color(0x" lib/ --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')
HC_MATCOLORS=$(grep -rn "Colors\." lib/ --include="*.dart" | grep -v "$EXCLUDE\|Colors\.transparent" | wc -l | tr -d ' ')
HC_FONTSIZE=$(grep -rn "fontSize:" lib/ --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')
HC_EDGEINSETS=$(grep -rnE "EdgeInsets\." lib/ --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')

TK_COLORS=$(grep -rn "KfspColors\.\|context\.kfspColors\." lib/ --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')
TK_TEXT=$(grep -rn "KfspTextStyles\." lib/ --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')
TK_SPACING=$(grep -rn "KfspSpacing\." lib/ --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')

TOTAL_HC=$((HC_COLORS + HC_MATCOLORS + HC_FONTSIZE + HC_EDGEINSETS))
TOTAL_TK=$((TK_COLORS + TK_TEXT + TK_SPACING))
TOTAL=$((TOTAL_HC + TOTAL_TK))
[ "$TOTAL" -gt 0 ] && ADOPTION=$((TOTAL_TK * 100 / TOTAL)) || ADOPTION=100

echo "📊 Design System Adoption: ${ADOPTION}% ($TOTAL_TK/$TOTAL)"
echo "   Colors: $TK_COLORS token vs $((HC_COLORS + HC_MATCOLORS)) HC"
echo "   Text:   $TK_TEXT token vs $HC_FONTSIZE HC"
echo "   Space:  $TK_SPACING token vs $HC_EDGEINSETS HC"
```

Ghi nhận trend để so sánh với commits trước. Mục tiêu: adoption tăng dần mỗi commit.

---

## Output Format

```markdown
# ✅/❌ KFSP Pre-Commit Gate — 5 Tiers
**Date:** [timestamp]
**Commit:** [mô tả]
**Source:** [path]

## Results (23 checks)

### T1 🛡️ Safety
| # | Check | Expected | Actual | Status |
|---|-------|----------|--------|--------|
| 1 | Dev journal | ≥1 | N entries | ✅/⚠️ |
| 2 | Guard score | ≥70 | X/100 | ✅/⚠️ |
| 3 | Git remote | Correct | gitlab/phudh3 | ✅/🔴 |

### T2 🔧 Code Integrity
| # | Check | Expected | Actual | Status |
|---|-------|----------|--------|--------|
| 4 | Design system NEW | 0 | N | ✅/🔴 |
| 5 | Secrets | 0 | 0 | ✅/🔴 |
| 6 | File size | <500 | X files >500 | ✅/⚠️ |
| 7 | Imports | PASS | PASS | ✅/🔴 |
| 8 | Conventions | PASS | N suggestions | ✅/ℹ️ |
| 9 | Sweep ran | Done | [summary] | ✅/⚠️ |
| 10 | Sync-check | In sync | [status] | ✅/⚠️ |

### T3 🧪 Quality Assurance
| # | Check | Expected | Actual | Status |
|---|-------|----------|--------|--------|
| 11 | Unit test coverage | ≥80% | X% (N/M) | ✅/🔴/⚠️ |
| 12 | Flutter test pass | ALL | X pass, Y fail | ✅/🔴 |
| 13 | Test-code ratio | ≥0.3 | 0.XX | ✅/⚠️ |
| 14 | Build-verify | PASS | [status] | ✅/⚠️/N/A |

### T4 👤 User Experience
| # | Check | Expected | Actual | Status |
|---|-------|----------|--------|--------|
| 15 | Vietnamese diacritics | 0 | 0 | ✅/🔴 |
| 16 | Number formatting | PASS | N warnings | ✅/⚠️ |
| 17 | UX parity | Done | [status] | ✅/⚠️/N/A |

### T5 🎯 Business Impact
| # | Check | Expected | Actual | Status |
|---|-------|----------|--------|--------|
| 18 | Docs updated | All | [list] | ✅/⚠️ |
| 19 | Feature 6D | PASS | [N/6] | ✅/⚠️/N/A |
| 20 | Build report | Created | [status] | ✅/⚠️ |
| 21 | Plan file | Updated | [status] | ✅/⚠️ |
| 22 | Changelog | Generated | +A ~M -D | ✅ |
| 23 | Legacy trend | Documented | X% adoption | ℹ️ |

## Verdict
- 🔴 BLOCK items: [list] → **FAIL — fix before commit**
- ⚠️ WARN items: [list] → **commit OK, fix soon**
- Total: X/23 PASS, Y WARN, Z BLOCK
- **Decision: ✅ OK to commit / ❌ BLOCK**
```

---

## Severity Summary

| Severity | Meaning | Action |
|----------|---------|--------|
| 🔴 BLOCK | Critical violation | PHẢI fix TRƯỚC khi commit |
| ⚠️ WARN | Non-critical but important | Commit OK, lên kế hoạch fix |
| ℹ️ INFO | Informational | Ghi nhận, không cần action |
| ✅ PASS | Clean | Tốt |
| N/A | Not applicable this commit | Skip |

## BLOCK items (commit KHÔNG ĐƯỢC thực hiện nếu có bất kỳ item nào):
- CHECK 3: Git remote sai
- CHECK 4: NEW design system violations
- CHECK 5: Secrets detected
- CHECK 7: Broken imports
- CHECK 11: Logic file không có test (model, provider, util, service)
- CHECK 12: Flutter test FAIL
- CHECK 15: Vietnamese diacritics violations

## Agent tự report items (không thể automate):
- CHECK 2: Guard score (agent nhớ từ lần chạy gần nhất)
- CHECK 9: Sweep summary (agent nhớ từ lần chạy gần nhất)
- CHECK 14: Build-verify status
- CHECK 17: UX parity status
- CHECK 18: Docs list (agent liệt kê đã update docs nào)
- CHECK 19: Feature 6D checklist
- CHECK 20: Build report status
</instructions>
