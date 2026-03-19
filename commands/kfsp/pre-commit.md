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

**Gate:** PASS/FAIL. Nếu FAIL → liệt kê violations cần fix trước khi commit.
</objective>

<instructions>
## Determine Source Path

Check which source exists:
1. `/tmp/kfsp_flutter/` (working copy — preferred)
2. `Product/kfsp_flutter_migration/source/` (package copy)

Use whichever exists. If neither → error.

## Check 1: 🚫 Hardcoded Values

```bash
# Colors — ONLY allowed in kfsp_colors.dart
grep -rn "Color(0x" lib/ --include="*.dart" | grep -v "kfsp_colors.dart" | grep -v "_test.dart"
grep -rn "Colors\." lib/ --include="*.dart" | grep -v "kfsp_colors.dart" | grep -v "_test.dart" | grep -v "Colors.transparent"

# Spacing — ONLY allowed in kfsp_spacing.dart
grep -rn "EdgeInsets\.all([0-9]" lib/ --include="*.dart" | grep -v "kfsp_spacing.dart"
grep -rn "SizedBox(height: [0-9]" lib/ --include="*.dart" | grep -v "kfsp_spacing.dart"
grep -rn "SizedBox(width: [0-9]" lib/ --include="*.dart" | grep -v "kfsp_spacing.dart"

# Font sizes — ONLY allowed in kfsp_text_styles.dart
grep -rn "fontSize:" lib/ --include="*.dart" | grep -v "kfsp_text_styles.dart" | grep -v "_test.dart"

# Font families
grep -rn "fontFamily:" lib/ --include="*.dart" | grep -v "kfsp_text_styles.dart"
```

**Exceptions:** Test files, generated files, third-party code.
**Severity:** Each violation = ⚠️ Warning (not blocking, but flagged).

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
| 1 | Hardcoded Values | ✅/⚠️ | N violations |
| 2 | Secrets | ✅/🔴 | N violations |
| 3 | File Size | ✅/⚠️ | N warnings |
| 4 | Imports | ✅/❌ | N broken |
| 5 | Conventions | ✅/ℹ️ | N suggestions |
| 6 | Dead Code | ✅/ℹ️ | N candidates |

## Violations Detail
[grouped by severity: 🔴 Critical → ⚠️ Warning → ℹ️ Info]

## Verdict
- 🔴 Critical found → **FAIL — fix before commit**
- ⚠️ Warnings only → **PASS with warnings — commit OK, fix soon**
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

## CHECK 9: 🧪 Unit Test Verification (2026-03-19+)

**Gate:** ⚠️ Warning — code changes SHOULD have corresponding tests.

```bash
# Check: modified source files have matching test files
git diff --cached --name-only | grep "lib/.*\.dart$" | while read -r src; do
  test_file=$(echo "$src" | sed 's|lib/|test/|;s|\.dart$|_test.dart|')
  if [ ! -f "$test_file" ]; then
    echo "⚠️ No test for: $src"
  fi
done
```

**Rule:** Mọi feature/widget mới PHẢI có unit test. Agent KHÔNG ĐƯỢC commit code mà chưa có test tương ứng.

## CHECK 10: 📦 Deliverable Check (2026-03-19+)

**Gate:** ⚠️ Warning — commit SHOULD be part of a deliverable.

Agent PHẢI đảm bảo trước khi commit:
- Đã có deliverable rõ ràng cho PM (bảng thay đổi, screenshots, cần test gì)
- KHÔNG commit "work in progress" mà chưa giao deliverable
</instructions>
