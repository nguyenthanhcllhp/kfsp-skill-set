---
name: kfsp:pre-commit
description: Pre-commit validation checklist — blast radius check, no hardcoded values, file size limits, import validation before committing changes
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - TodoWrite
---
<objective>
KFSP Pre-Commit Check — Validation gate trước khi commit code.

Chạy trước mỗi commit để đảm bảo:
1. Không có hardcoded colors/spacing/fonts leak vào code
2. Imports hợp lệ, không circular dependency
3. File size hợp lý (không commit file quá lớn)
4. Không commit secrets/credentials
5. Code conventions tuân thủ project patterns
6. Changelog so với preprod baseline (nếu có)

**Gate:** PASS/FAIL. Nếu FAIL → liệt kê violations cần fix trước khi commit.

## ⚠️ Checklist Protocol (BẮT BUỘC — 2026-03-20)

Agent PHẢI thực hiện 2 bước khi chạy pre-commit:

### Bước 1 — TRƯỚC KHI chạy: Gửi Thanh bảng checklist + expected results
```
📋 Pre-commit Checklist — [mô tả commit]
| # | Hạng mục | Expected | Files cần check |
|---|----------|----------|-----------------|
| 1 | 🎨 Hardcoded colors MỚI | 0 | file1.dart, file2.dart |
| 2 | 🎨 Hardcoded fontSize MỚI | 0 | ... |
| 3 | 🎨 Hardcoded spacing MỚI | 0 | ... |
| 4 | 🎨 Component convention | PASS | ... |
| 5 | 🔒 Secrets | 0 | ... |
| 6 | 📏 File size < 500 lines | PASS | ... |
| 7 | 📦 Import validation | PASS | ... |
| 8 | 🇻🇳 Vietnamese diacritics | PASS | ... |
| 9 | 🔢 Number formatting | PASS | ... |
| 10 | 📝 Changelog vs preprod | Generated | ... |
| 11 | 🧪 Unit test coverage | ≥80% files | lib/*.dart → test/*_test.dart |
| 12 | 🧪 Flutter test pass | ALL PASS | `flutter test` |
| 13 | 🧪 Test-code ratio | ≥0.3 (logic) | test/ vs lib/ LOC |
```

### Bước 2 — SAU KHI chạy: Lôi lại bảng + điền kết quả thực tế
```
✅/❌ Pre-commit Results — [mô tả commit]
| # | Hạng mục | Expected | Actual | Status |
|---|----------|----------|--------|--------|
| 1 | 🎨 Hardcoded colors MỚI | 0 | 0 | ✅ PASS |
| 2 | 🎨 Hardcoded fontSize MỚI | 0 | 2 | ❌ FAIL |
| ... |
| 11 | 🧪 Unit test coverage | ≥80% | 75% (15/20) | ⚠️ WARN |
| 12 | 🧪 Flutter test pass | ALL PASS | 42 pass, 0 fail | ✅ PASS |
| 13 | 🧪 Test-code ratio | ≥0.3 | 0.25 | ⚠️ WARN |
→ Tổng: X/13 PASS, Y FAIL → cần fix / OK to commit
```

Agent KHÔNG ĐƯỢC bỏ qua bước 1 hoặc bước 2. PM cần thấy CẢ HAI bảng.
</objective>

<instructions>
## Determine Source Path

Check which source exists:
1. `/tmp/kfsp_flutter/` (working copy — preferred)
2. `Product/kfsp_flutter_migration/source/` (package copy)

Use whichever exists. If neither → error.

## Check 1: 🎨 Design System Compliance (GATE — blocks on NEW violations)

### Token Reference Files
All design tokens MUST be defined in and imported from these files:
- `kfsp_colors.dart` — all Color values (brand, stock, surface, text)
- `kfsp_text_styles.dart` — all TextStyle/fontSize/fontFamily/fontWeight
- `kfsp_spacing.dart` — all EdgeInsets, padding, margin, gap values
- `kfsp_animations.dart` — all Duration, Curve values
- `kfsp_shadows.dart` — all BoxShadow, elevation values
- `kfsp_buttons.dart` — all ButtonStyle values

### Step 1: Get changed files
```bash
# Only check files in the current commit (staged changes)
CHANGED_DART=$(git diff --cached --name-only -- '*.dart' | grep -E "^lib/(screens|common|features|widgets)/" | grep -v "_test.dart")
if [ -z "$CHANGED_DART" ]; then
  echo "✅ No Dart files changed in lib/screens/ or lib/common/ — skip design system check"
  exit 0
fi
echo "📋 Changed files to check:"
echo "$CHANGED_DART"
```

### Step 2: Count violations BEFORE change (baseline)
```bash
for f in $CHANGED_DART; do
  # Get the BEFORE version from git
  BEFORE_COLORS=$(git show HEAD:"$f" 2>/dev/null | grep -c "Color(0x" || echo "0")
  BEFORE_MATCOLORS=$(git show HEAD:"$f" 2>/dev/null | grep -c "Colors\." | grep -v "Colors\.transparent" || echo "0")
  BEFORE_FONTSIZE=$(git show HEAD:"$f" 2>/dev/null | grep -c "fontSize:" || echo "0")
  BEFORE_EDGEINSETS=$(git show HEAD:"$f" 2>/dev/null | grep -cE "EdgeInsets\.(all|only|symmetric|fromLTRB)\s*\(" || echo "0")

  # Get the AFTER version (staged)
  AFTER_COLORS=$(git show :"$f" 2>/dev/null | grep -c "Color(0x" || echo "0")
  AFTER_MATCOLORS=$(git show :"$f" 2>/dev/null | grep "Colors\." | grep -vc "Colors\.transparent" || echo "0")
  AFTER_FONTSIZE=$(git show :"$f" 2>/dev/null | grep -c "fontSize:" || echo "0")
  AFTER_EDGEINSETS=$(git show :"$f" 2>/dev/null | grep -cE "EdgeInsets\.(all|only|symmetric|fromLTRB)\s*\(" || echo "0")

  # Delta = new violations added
  D_COLORS=$((AFTER_COLORS - BEFORE_COLORS))
  D_MATCOLORS=$((AFTER_MATCOLORS - BEFORE_MATCOLORS))
  D_FONTSIZE=$((AFTER_FONTSIZE - BEFORE_FONTSIZE))
  D_EDGEINSETS=$((AFTER_EDGEINSETS - BEFORE_EDGEINSETS))

  [ "$D_COLORS" -gt 0 ] && echo "🔴 BLOCK: $f — +$D_COLORS new Color(0x... (use KfspColors.*)"
  [ "$D_MATCOLORS" -gt 0 ] && echo "🔴 BLOCK: $f — +$D_MATCOLORS new Colors.* (use KfspColors.*)"
  [ "$D_FONTSIZE" -gt 0 ] && echo "🔴 BLOCK: $f — +$D_FONTSIZE new fontSize: raw (use KfspTextStyles.* or KfspColors.font*)"
  [ "$D_EDGEINSETS" -gt 0 ] && echo "🔴 BLOCK: $f — +$D_EDGEINSETS new EdgeInsets with raw numbers (use KfspSpacing.*)"
done
```

### Step 3: Show specific new violations (file:line)
```bash
for f in $CHANGED_DART; do
  # Show only ADDED lines (lines starting with +) that have violations
  git diff --cached -U0 "$f" | grep "^+" | grep -v "^+++" | grep -n "Color(0x" && echo "  ↳ in $f — use KfspColors.* token instead"
  git diff --cached -U0 "$f" | grep "^+" | grep -v "^+++" | grep "Colors\." | grep -v "Colors\.transparent" && echo "  ↳ in $f — use KfspColors.* token instead"
  git diff --cached -U0 "$f" | grep "^+" | grep -v "^+++" | grep "fontSize:" && echo "  ↳ in $f — use KfspTextStyles.* token instead"
  git diff --cached -U0 "$f" | grep "^+" | grep -v "^+++" | grep -E "EdgeInsets\.(all|only|symmetric|fromLTRB)\s*\([0-9]" && echo "  ↳ in $f — use KfspSpacing.* token instead"
done
```

### Blocking vs Warning Rules

| Pattern | Where | Verdict |
|---------|-------|---------|
| NEW `Color(0xFF...` | `lib/screens/`, `lib/common/`, `lib/features/`, `lib/widgets/` | 🔴 **BLOCK** |
| NEW `Colors.*` (except `Colors.transparent`) | Same folders | 🔴 **BLOCK** |
| NEW `fontSize:` with raw number | Same folders | 🔴 **BLOCK** |
| NEW `EdgeInsets.*` with raw numbers | Same folders | 🔴 **BLOCK** |
| EXISTING violations (count unchanged or decreased) | Anywhere | ⚠️ **WARN** (legacy debt, fix later) |
| Any violation in `lib/theme/`, `lib/constants/` | Token definition files | ✅ **ALLOWED** (that's where tokens live) |
| Any violation in `_test.dart` | Test files | ✅ **ALLOWED** |
| Any violation in `generated/`, `l10n/` | Generated code | ✅ **ALLOWED** |

### Examples: WRONG vs RIGHT

```dart
// ❌ WRONG — hardcoded color
Container(color: Color(0xFF7B3AEC))
Container(color: Colors.purple)
Text('Hello', style: TextStyle(color: Colors.white))

// ✅ RIGHT — use design tokens
Container(color: context.kfspColors.brandPrimary)
Container(color: KfspColors.brandPrimary)
Text('Hello', style: KfspTextStyles.bodyMedium)

// ❌ WRONG — hardcoded fontSize
Text('Hello', style: TextStyle(fontSize: 14))

// ✅ RIGHT — use text style tokens
Text('Hello', style: KfspTextStyles.bodyMedium)
Text('Hello', style: context.kfspColors.font14)

// ❌ WRONG — hardcoded spacing
Padding(padding: EdgeInsets.all(16))
SizedBox(height: 8)

// ✅ RIGHT — use spacing tokens
Padding(padding: KfspSpacing.paddingM)
SizedBox(height: KfspSpacing.sm)

// ❌ WRONG — hardcoded shadow
BoxShadow(color: Colors.black26, blurRadius: 4, offset: Offset(0, 2))

// ✅ RIGHT — use shadow tokens
KfspShadows.cardShadow
```

### Legacy Violations Report (informational)
```bash
# Report total legacy violations (not blocking, just awareness)
TOTAL_HC_COLORS=$(grep -rn "Color(0x" lib/ --include="*.dart" | grep -v "kfsp_colors.dart\|_test.dart\|theme/\|constants/" | wc -l | tr -d ' ')
TOTAL_MAT_COLORS=$(grep -rn "Colors\." lib/ --include="*.dart" | grep -v "kfsp_colors.dart\|_test.dart\|theme/\|constants/\|Colors\.transparent" | wc -l | tr -d ' ')
TOTAL_FONTSIZE=$(grep -rn "fontSize:" lib/ --include="*.dart" | grep -v "kfsp_text_styles.dart\|_test.dart\|theme/\|constants/" | wc -l | tr -d ' ')
TOTAL_EDGEINSETS=$(grep -rnE "EdgeInsets\.(all|only|symmetric|fromLTRB)\s*\([0-9]" lib/ --include="*.dart" | grep -v "kfsp_spacing.dart\|_test.dart\|theme/\|constants/" | wc -l | tr -d ' ')
echo "📊 Legacy debt: $TOTAL_HC_COLORS Color(0x, $TOTAL_MAT_COLORS Colors.*, $TOTAL_FONTSIZE fontSize, $TOTAL_EDGEINSETS EdgeInsets"
```

**Exceptions:** Test files, generated files, theme/constants definition files.
**Severity for NEW violations:** 🔴 BLOCK — must fix before commit.
**Severity for EXISTING violations:** ⚠️ Warning (legacy debt, track separately).

## Check 2: 🔒 Secrets & Credentials

```bash
# API keys, tokens, passwords
grep -rni "api_key\|apikey\|api-key\|secret\|password\|token\|bearer" lib/ --include="*.dart" | grep -v "// " | grep -v "kDebugMode"
grep -rn "\.env" lib/ --include="*.dart"

# Hardcoded URLs that might be staging/dev
grep -rn "localhost\|127\.0\.0\.1\|192\.168\." lib/ --include="*.dart" | grep -v "env_config.dart"
```

**Severity:** 🔴 CRITICAL — block commit.

## Check 3: 📏 File Size Check

```bash
# Dart files > 500 lines (might need splitting)
find lib/ -name "*.dart" -exec wc -l {} + | sort -rn | head -20

# Any single file > 1000 lines
find lib/ -name "*.dart" -exec sh -c 'lines=$(wc -l < "$1"); [ "$lines" -gt 1000 ] && echo "⚠️ $1: $lines lines"' _ {} \;

# Large assets check
find assets/ -size +5M -exec ls -lh {} \; 2>/dev/null
```

**Severity:** > 500 lines = ℹ️ Info. > 1000 lines = ⚠️ Warning.

## Check 4: 📦 Import Validation

```bash
# Broken imports — import files that don't exist
grep -rn "^import.*package:kfsp" lib/ --include="*.dart" | while read line; do
  file=$(echo "$line" | sed "s/.*package:kfsp_flutter\//lib\//;s/'.*//" | sed "s/\";.*//")
  [ ! -f "$file" ] && echo "❌ Broken import: $line → $file not found"
done

# Relative imports (should use package: imports)
grep -rn "^import '\.\." lib/ --include="*.dart"
```

## Check 5: 🏗️ Convention Check

```bash
# Widget files should have Widget/Screen/Page suffix
find lib/features/ -name "*.dart" | grep -v "test" | while read f; do
  basename=$(basename "$f" .dart)
  # Skip non-UI files
  grep -q "extends.*Widget\|extends.*Screen\|extends.*Page" "$f" 2>/dev/null && \
  ! echo "$basename" | grep -qE "(widget|screen|page|view|card|tile|badge|button|dialog|sheet|bar|header|section|indicator|picker)" && \
  echo "ℹ️ $f — contains Widget but filename doesn't reflect it"
done

# Riverpod: providers should be in providers/ or controllers/
grep -rln "Provider\|Notifier" lib/features/ --include="*.dart" | grep -v "providers\|controllers\|_test"
```

## Check 6: 🧹 Dead Code Detection

```bash
# Unused imports (basic check)
for f in $(find lib/ -name "*.dart" ! -name "*_test.dart"); do
  grep "^import " "$f" | while read imp; do
    # Extract imported name
    name=$(echo "$imp" | grep "show " | sed 's/.*show //;s/;//;s/,.*//')
    if [ -n "$name" ]; then
      count=$(grep -c "$name" "$f")
      [ "$count" -le 1 ] && echo "ℹ️ $f: possibly unused import: $name"
    fi
  done
done 2>/dev/null | head -20
```

## Output Format

```markdown
# ✅/❌ KFSP Pre-Commit Check
**Date:** [timestamp]
**Source:** [path]

## Result: PASS ✅ / FAIL ❌

## Findings
| # | Check | Status | Violations |
|---|-------|--------|------------|
| 1 | 🎨 Design System Compliance | ✅/🔴 | +N new violations (BLOCK) / N legacy (WARN) |
| 2 | 🔒 Secrets | ✅/🔴 | N violations |
| 3 | 📏 File Size | ✅/⚠️ | N warnings |
| 4 | 📦 Imports | ✅/❌ | N broken |
| 5 | 🏗️ Conventions | ✅/ℹ️ | N suggestions |
| 6 | 🧹 Dead Code | ✅/ℹ️ | N candidates |
| 7 | 🧪 Unit Test Coverage | ✅/🔴/⚠️ | X/Y files have tests (Z%) |
| 8 | 🧪 Flutter Test Pass | ✅/🔴 | X passed, Y failed, Z skipped |
| 9 | 🧪 Test-Code Ratio | ✅/⚠️ | ratio: 0.XX (target ≥0.3) |

## Design System Compliance Detail
| File | Color(0x | Colors.* | fontSize | EdgeInsets | Verdict |
|------|----------|----------|----------|------------|---------|
| path/file.dart | +2 new | 0 | +1 new | 0 | 🔴 BLOCK |
| path/other.dart | 0 | 0 | 0 | 0 | ✅ PASS |

**Legacy debt (not blocking):** X Color(0x, Y Colors.*, Z fontSize, W EdgeInsets

## Violations Detail
[grouped by severity: 🔴 Critical → ⚠️ Warning → ℹ️ Info]

## Verdict
- 🔴 Critical found (secrets, NEW design system violations) → **FAIL — fix before commit**
- ⚠️ Warnings only (legacy debt, file size) → **PASS with warnings — commit OK, fix soon**
- ℹ️ Info only → **PASS — clean commit**
```

## CHECK 7: 🔢 Number Formatting (2026-03-15+)

```bash
# Tìm toStringAsFixed dùng cho số lớn (nên dùng NumberFormat thay vì toStringAsFixed)
grep -rn "toStringAsFixed" lib/ --include="*.dart" | grep -v "_test.dart" | grep -v "perChange\|percent"

# Tìm số hiển thị user-facing thiếu NumberFormat
grep -rn "\.toString()" lib/ --include="*.dart" | grep -v "_test.dart" | grep -v "// "
```

**Rule:**
- Dấu `.` = thập phân, dấu `,` = phần nghìn
- Dùng `NumberFormat` từ `intl` — KHÔNG `toStringAsFixed` cho số lớn
- Kiểm tra như chính tả — bắt buộc trước khi ship
- Ví dụ: `26,000` ✅ `26000` ❌ / `1,572.6 Tỷ` ✅ `1572.6 Tỷ` ❌

**Severity:** ⚠️ Warning — flag nhưng không block commit.

## CHECK 9: 📋 Test Case Registry (2026-03-15+)

```bash
REGISTRY="Product/kfsp_flutter_migration/docs/test_cases/master_test_registry.html"
if [ -f "$REGISTRY" ]; then
  # Check if changed screens have test cases
  CHANGED_SCREENS=$(git diff --cached --name-only | grep "lib/screens\|lib/features" | sed 's|.*/||;s|_screen\.dart||;s|_page\.dart||' | sort -u)
  for screen in $CHANGED_SCREENS; do
    TC=$(grep -ci "$screen" "$REGISTRY" 2>/dev/null || echo "0")
    [ "$TC" -eq 0 ] && echo "⚠️ Screen '$screen' modified but has 0 test cases in registry"
  done
else
  echo "⚠️ Master Test Registry not found — consider creating one"
fi
```

**Severity:** ⚠️ Warning — flag nhưng không block commit. Nhắc agent thêm test cases.

## CHECK 10: 📔 Dev Journal Compliance (2026-03-15+)

```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
MONTH=$(date +%Y-%m)

# Check if any major decision was made without journal entry
ENTRIES=$(find "$JOURNAL_DIR/$MONTH/" -name "*.md" 2>/dev/null | wc -l | tr -d ' ')
echo "📔 Journal entries (${MONTH}): $ENTRIES"

# Check: if committing significant changes, should have decision entry
CHANGED_FILES=$(git diff --cached --name-only | wc -l | tr -d ' ')
if [ "$CHANGED_FILES" -gt 5 ] && [ "$ENTRIES" -eq 0 ]; then
  echo "⚠️ Committing $CHANGED_FILES files but 0 journal entries — document decisions!"
fi
```

**Severity:** ⚠️ Warning — nhắc nhở, không block commit.

## CHECK 8: 🔒 Git Remote Verification (2026-03-13+)

Before any commit that might be pushed:
- Verify `git remote -v` → `origin` points to expected URL
- App code: `https://gitlab.com/phudh3/flutter_kfsp_app.git`
- Skills: `https://github.com/nguyenthanhcllhp/kfsp-skill-set.git`
- **NEVER** push without Thanh's explicit permission
- **NEVER** assume GitHub = GitLab
- If remote looks wrong → 🔴 FAIL — ask Thanh before proceeding

## CHECK 9: 🧪 Unit Test Coverage (2026-03-20+)

**Gate:**
- Pure logic files (models, utils, providers, controllers): 🔴 **BLOCK** nếu không có test
- Widget/screen files: ⚠️ **WARN** nếu không có widget test (flag, không block)
- Token/theme files: ⚠️ **WARN** nếu không có test (token values nên được test)

### Step 1: Map lib/ files → test/ files
```bash
TOTAL_SRC=0
TOTAL_WITH_TEST=0
BLOCK_LIST=""
WARN_LIST=""

# Check all changed .dart files in lib/
git diff --cached --name-only | grep "^lib/.*\.dart$" | grep -v "generated/" | while read -r src; do
  TOTAL_SRC=$((TOTAL_SRC + 1))

  # Map: lib/foo/bar.dart → test/foo/bar_test.dart
  test_file=$(echo "$src" | sed 's|^lib/|test/|;s|\.dart$|_test.dart|')

  # Also check alternative patterns: test/foo/bar_widget_test.dart
  test_alt=$(echo "$src" | sed 's|^lib/|test/|;s|\.dart$|_widget_test.dart|')

  if [ -f "$test_file" ] || [ -f "$test_alt" ]; then
    TOTAL_WITH_TEST=$((TOTAL_WITH_TEST + 1))
  else
    # Classify: logic file (BLOCK) vs widget file (WARN)
    if echo "$src" | grep -qE "(model|provider|controller|service|util|helper|repository|notifier|extension)"; then
      BLOCK_LIST="$BLOCK_LIST\n🔴 BLOCK — no test: $src → expected: $test_file"
    elif echo "$src" | grep -qE "(theme|token|color|style|spacing|animation|shadow|button)"; then
      WARN_LIST="$WARN_LIST\n⚠️ WARN — no test: $src (token/theme file nên có test)"
    else
      WARN_LIST="$WARN_LIST\n⚠️ WARN — no test: $src (widget/screen — nên có widget test)"
    fi
  fi
done

# Report coverage
if [ "$TOTAL_SRC" -gt 0 ]; then
  PCT=$((TOTAL_WITH_TEST * 100 / TOTAL_SRC))
  echo "🧪 Test coverage: $TOTAL_WITH_TEST/$TOTAL_SRC files have tests ($PCT%)"
else
  echo "🧪 No source files changed"
fi

# Show blockers and warnings
[ -n "$BLOCK_LIST" ] && echo -e "$BLOCK_LIST"
[ -n "$WARN_LIST" ] && echo -e "$WARN_LIST"
```

### Mapping Rules
| Source pattern | Test expected | Severity nếu thiếu |
|----------------|--------------|---------------------|
| `lib/**/model*.dart` | `test/**/model*_test.dart` | 🔴 BLOCK |
| `lib/**/provider*.dart` | `test/**/provider*_test.dart` | 🔴 BLOCK |
| `lib/**/controller*.dart` | `test/**/controller*_test.dart` | 🔴 BLOCK |
| `lib/**/service*.dart` | `test/**/service*_test.dart` | 🔴 BLOCK |
| `lib/**/util*.dart`, `helper*.dart` | `test/**/*_test.dart` | 🔴 BLOCK |
| `lib/**/repository*.dart` | `test/**/repository*_test.dart` | 🔴 BLOCK |
| `lib/**/notifier*.dart` | `test/**/notifier*_test.dart` | 🔴 BLOCK |
| `lib/**/*_screen.dart`, `*_page.dart` | `test/**/*_test.dart` | ⚠️ WARN |
| `lib/**/*_widget.dart`, `*_card.dart` | `test/**/*_test.dart` | ⚠️ WARN |
| `lib/**/theme*.dart`, `*_colors.dart` | `test/**/*_test.dart` | ⚠️ WARN |
| `lib/generated/`, `lib/l10n/` | — | ✅ SKIP |

**Rule:** Mọi feature/widget mới PHẢI có unit test. Agent KHÔNG ĐƯỢC commit code mà chưa có test tương ứng. Logic files (models, utils, providers) BLOCK commit nếu thiếu test.

## CHECK 12: 🧪 Flutter Test Pass (2026-03-20+)

**Gate:** 🔴 **BLOCK** — commit KHÔNG ĐƯỢC thực hiện nếu có test FAIL.

```bash
echo "🧪 Running flutter test..."

# Run flutter test (--no-pub for speed if packages already resolved)
TEST_OUTPUT=$(flutter test --no-pub 2>&1)
TEST_EXIT=$?

if [ $TEST_EXIT -eq 0 ]; then
  # Parse results
  PASSED=$(echo "$TEST_OUTPUT" | grep -oE "[0-9]+ tests? passed" | head -1)
  SKIPPED=$(echo "$TEST_OUTPUT" | grep -oE "[0-9]+ skipped" | head -1)
  echo "✅ Flutter test PASS — $PASSED, $SKIPPED"
else
  FAILED=$(echo "$TEST_OUTPUT" | grep -oE "[0-9]+ tests? failed" | head -1)
  PASSED=$(echo "$TEST_OUTPUT" | grep -oE "[0-9]+ tests? passed" | head -1)
  echo "🔴 BLOCK — Flutter test FAIL: $FAILED, $PASSED"
  echo ""
  echo "Failed tests:"
  echo "$TEST_OUTPUT" | grep -A2 "FAILED\|══.*Exception\|Expected:\|Actual:" | head -30
fi
```

**Rule:**
- `flutter test` PHẢI pass 100% trước khi commit
- Nếu có test fail → fix test HOẶC fix code → chạy lại → pass → mới commit
- Cho phép `--no-pub` để tăng tốc nếu packages đã resolved
- Report format: `X tests passed, Y failed, Z skipped`

**Severity:** 🔴 BLOCK — KHÔNG commit khi có test fail.

## CHECK 13: 🧪 Test-Code Ratio (2026-03-20+)

**Gate:** ⚠️ Warning — báo cáo tỷ lệ test/code, khuyến nghị cải thiện.

```bash
# Count lines in lib/ (excluding generated)
LIB_LINES=$(find lib/ -name "*.dart" ! -path "*/generated/*" ! -path "*/l10n/*" -exec cat {} + 2>/dev/null | wc -l | tr -d ' ')

# Count lines in test/
TEST_LINES=$(find test/ -name "*.dart" -exec cat {} + 2>/dev/null | wc -l | tr -d ' ')

# Calculate ratio
if [ "$LIB_LINES" -gt 0 ]; then
  # Use awk for float division
  RATIO=$(awk "BEGIN {printf \"%.2f\", $TEST_LINES / $LIB_LINES}")
  echo "🧪 Test-code ratio: $TEST_LINES test LOC / $LIB_LINES lib LOC = $RATIO"

  # Evaluate
  RATIO_INT=$(awk "BEGIN {printf \"%d\", $TEST_LINES * 100 / $LIB_LINES}")
  if [ "$RATIO_INT" -lt 10 ]; then
    echo "⚠️ Ratio < 0.10 — cần viết thêm test đáng kể"
  elif [ "$RATIO_INT" -lt 30 ]; then
    echo "⚠️ Ratio < 0.30 — chưa đạt target cho logic files"
  else
    echo "✅ Ratio ≥ 0.30 — đạt target"
  fi
else
  echo "🧪 No source files found in lib/"
fi

# Breakdown by folder (top-level)
echo ""
echo "📊 Test coverage by folder:"
for dir in $(find lib/ -mindepth 1 -maxdepth 1 -type d ! -name "generated" ! -name "l10n"); do
  DIR_NAME=$(basename "$dir")
  DIR_LINES=$(find "$dir" -name "*.dart" -exec cat {} + 2>/dev/null | wc -l | tr -d ' ')
  TEST_DIR="test/$DIR_NAME"
  if [ -d "$TEST_DIR" ]; then
    TEST_DIR_LINES=$(find "$TEST_DIR" -name "*.dart" -exec cat {} + 2>/dev/null | wc -l | tr -d ' ')
    DIR_RATIO=$(awk "BEGIN {printf \"%.2f\", $TEST_DIR_LINES / ($DIR_LINES > 0 ? $DIR_LINES : 1)}")
  else
    TEST_DIR_LINES=0
    DIR_RATIO="0.00"
  fi
  echo "  $DIR_NAME: $TEST_DIR_LINES/$DIR_LINES LOC (ratio: $DIR_RATIO)"
done
```

**Targets:**
| Loại file | Target ratio | Lý do |
|-----------|-------------|-------|
| Logic (models, providers, utils) | ≥ 0.5 | Business-critical, dễ test |
| Widgets/screens | ≥ 0.2 | Widget test phức tạp hơn |
| Overall project | ≥ 0.3 | Balanced coverage |

**Severity:** ⚠️ Warning — không block commit nhưng báo cáo rõ ràng cho PM thấy trạng thái test coverage.

## CHECK 10: 📦 Deliverable Check (2026-03-19+)

**Gate:** ⚠️ Warning — commit SHOULD be part of a deliverable.

Agent PHẢI đảm bảo trước khi commit:
- Đã có deliverable rõ ràng cho PM (bảng thay đổi, screenshots, cần test gì)
- KHÔNG commit "work in progress" mà chưa giao deliverable

## CHECK 11: 📋 Changelog vs Preprod Baseline (2026-03-20+)

So sánh branch hiện tại với preprod để tạo changelog.

### Khi nào chạy:
- Trước MỌI commit trên branch `thanh_sg` (hoặc feature branch)
- Sau khi merge preprod vào branch hiện tại
- Khi chuẩn bị giao deliverable cho PM

### Cách thực hiện:
```bash
# 1. Xác định base branch (preprod hoặc main)
BASE_BRANCH="preprod"
git rev-parse --verify "$BASE_BRANCH" 2>/dev/null || BASE_BRANCH="main"

# 2. Liệt kê tất cả commits từ base đến HEAD
echo "📋 Changelog (${BASE_BRANCH}..HEAD):"
git log ${BASE_BRANCH}..HEAD --oneline --no-merges

# 3. Liệt kê files changed
echo ""
echo "📁 Files changed (${BASE_BRANCH}..HEAD):"
git diff ${BASE_BRANCH}..HEAD --stat

# 4. Phân loại thay đổi
echo ""
echo "📊 Breakdown:"
ADDED=$(git diff ${BASE_BRANCH}..HEAD --diff-filter=A --name-only | wc -l | tr -d ' ')
MODIFIED=$(git diff ${BASE_BRANCH}..HEAD --diff-filter=M --name-only | wc -l | tr -d ' ')
DELETED=$(git diff ${BASE_BRANCH}..HEAD --diff-filter=D --name-only | wc -l | tr -d ' ')
echo "  Added: $ADDED files"
echo "  Modified: $MODIFIED files"
echo "  Deleted: $DELETED files"

# 5. Convention check trên TẤT CẢ files changed từ base
CHANGED_DART=$(git diff ${BASE_BRANCH}..HEAD --name-only -- '*.dart' | grep -E "^lib/(screens|common|features|widgets)/")
if [ -n "$CHANGED_DART" ]; then
  echo ""
  echo "🎨 Design System violations in changed files:"
  for f in $CHANGED_DART; do
    HC=$(grep -c "Color(0x" "$f" 2>/dev/null || echo "0")
    MC=$(grep "Colors\." "$f" 2>/dev/null | grep -vc "Colors\.transparent" || echo "0")
    FS=$(grep -c "fontSize:" "$f" 2>/dev/null || echo "0")
    [ "$HC" -gt 0 ] && echo "  ⚠️ $f — $HC Color(0x..."
    [ "$MC" -gt 0 ] && echo "  ⚠️ $f — $MC Colors.*"
    [ "$FS" -gt 0 ] && echo "  ⚠️ $f — $FS fontSize:"
  done
fi
```

### Output cho Thanh:
```markdown
## 📋 Changelog vs Preprod

### Commits (N total):
- abc1234 — feat: thêm foreign chart timeline
- def5678 — fix: stock code color to purple
- ...

### Files: +A added, ~M modified, -D deleted

### Design System Status:
| File | Violations | Type |
|------|-----------|------|
| screens/market/... | 3 | Color(0x — legacy |
| screens/auth/... | 5 | Colors.* — legacy |
→ Tổng: X violations (N legacy, M mới)
```

**Severity:** ℹ️ Info — không block, nhưng PM cần thấy changelog trước khi approve.
**Khi merge preprod:** PHẢI chạy convention check trên TẤT CẢ files từ dev khác → báo cáo violations → đề xuất fix plan.
</instructions>
