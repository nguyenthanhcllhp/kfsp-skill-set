---
name: kfsp:guard
description: Continuous guardian — monitor codebase for regressions, drift, and hidden issues. Run periodically or after any agent-driven changes to prevent loss of control
argument-hint: "[--deep] [--report-only]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Guardian — Hệ thống giám sát liên tục, được thiết kế đặc biệt cho agent-driven development.

Khi agent (Claude hoặc future agents) thực hiện thay đổi code, Guardian sẽ:
1. Phát hiện regression (feature cũ bị break)
2. Phát hiện drift (code diverge khỏi design/spec)
3. Phát hiện anomaly (file bất thường, size spike, orphan files)
4. Cross-validate giữa nhiều chiều: code ↔ design tokens ↔ docs ↔ tests ↔ routes

**Triết lý:** "Trust but verify" — Agent làm tốt, Guardian kiểm tra lại.

**Flags:**
- `--deep` → Full deep scan (chậm hơn, kỹ hơn)
- `--report-only` → Chỉ báo cáo, không đề xuất fix
</objective>

<instructions>
## Determine Mode

Parse `$ARGUMENTS`:
- Không flag → Standard scan
- `--deep` → Deep scan (thêm content analysis, cross-file logic check)
- `--report-only` → Skip fix suggestions

## Source Path Detection

```bash
# Priority order
for path in "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter" "Product/kfsp_flutter_migration/source"; do
  [ -d "$path/lib" ] && echo "Using: $path" && break
done
```

## Guardian Check 1: 🏗️ Structural Integrity

### 1a: File Structure Matches Architecture
```
Expected structure:
lib/
├── main.dart
├── core/theme/  (5 files: colors, theme, theme_colors, spacing, text_styles)
├── core/network/ (client, interceptors)
├── config/routes/ (app_router, bottom_nav_shell)
├── config/env/ (env_config)
├── common/ (extensions, utils, constants, data)
├── features/
│   ├── onboarding/
│   ├── dashboard/
│   ├── priceboard/
│   ├── tech_analysis/
│   ├── fundamental/
│   ├── watchlist/
│   └── alert/
```

Verify actual structure matches. Flag:
- Missing expected directories → 🔴
- Unexpected top-level directories → ⚠️
- Empty feature directories → ⚠️

### 1b: Orphan Files
```bash
# Files not imported by anything
for f in $(find lib/ -name "*.dart" ! -name "main.dart" ! -name "*_test.dart" ! -name "*.g.dart" ! -name "*.freezed.dart"); do
  basename=$(basename "$f" .dart)
  count=$(grep -rl "$basename" lib/ --include="*.dart" | grep -v "$f" | wc -l)
  [ "$count" -eq 0 ] && echo "🟡 Orphan: $f (not imported anywhere)"
done
```

### 1c: File Size Anomalies
```bash
# Compare file sizes against baseline
find lib/ -name "*.dart" -exec wc -l {} + | sort -rn | head -10
# Flag files > 500 lines
# Flag files that grew >50% since last check (if baseline exists)
```

## Guardian Check 2: 🎨 Design System Fidelity

### 2a: Token Coverage
```bash
# Count how many .dart files use context.kfspColors vs hardcoded
total=$(find lib/features/ -name "*.dart" | wc -l)
using_tokens=$(grep -rl "context\.kfspColors\|context\.kfspSpacing\|context\.kfspTextStyles" lib/features/ --include="*.dart" | wc -l)
echo "Token adoption: $using_tokens / $total files"
```

### 2b: Color Palette Integrity
Read kfsp_colors.dart → extract all color hex values.
Cross-check: are these same values in docs/03_DESIGN_SYSTEM.md?
Any new colors added without doc update?

### 2c: Theme Extension Completeness
Check kfsp_theme_colors.dart covers all colors from kfsp_colors.dart.
Check kfsp_theme.dart registers ThemeExtension.

## Guardian Check 3: 🔗 Cross-Cutting Concerns

### 3a: kDebugMode Bypass Completeness
```bash
# Every controller/provider that fetches data should have kDebugMode bypass
grep -rn "kDebugMode" lib/ --include="*.dart" | grep -v "_test.dart"
# Check: any new API call without kDebugMode bypass?
grep -rn "\.get(\|\.post(\|\.put(\|\.delete(" lib/ --include="*.dart" | grep -v "kDebugMode" | grep -v "_test.dart"
```

### 3b: SharedPreferences Keys
```bash
# All pref keys should be in one place (constants or enum)
grep -rn "getString\|setString\|getBool\|setBool\|getInt\|setInt" lib/ --include="*.dart"
# Flag: key strings used inline instead of constants
```

### 3c: Error Handling Consistency
```bash
# Every async function should have try/catch or error handler
grep -rn "async " lib/features/ --include="*.dart" | head -30
# Flag: async without try/catch pattern
```

## Guardian Check 4: 📐 Spec ↔ Code Alignment

### 4a: Features in Code vs Features in Plan
Read docs/11_CLEAN_SLATE_PLAN.md → extract expected features per phase.
Check lib/features/ → actual features.
Gap analysis.

### 4b: Route Map Accuracy
Read app_router.dart → extract all routes.
Read docs/02_ARCHITECTURE_GUIDE.md → route list in doc.
Compare.

### 4c: Mock Data Completeness
Read mock_data.dart → extract mock data structures.
Check: every screen that needs data has mock data path?

## Guardian Check 5: 🛡️ Agent Safety Checks

### 5a: No Unauthorized Deletions
```bash
# Check git status for deleted files (if in git repo)
cd [source_path] && git status --short 2>/dev/null | grep "^D " | head -20
```

### 5b: No Secret Leaks
```bash
grep -rni "password\|secret\|api.key\|token\|bearer\|private.key" lib/ --include="*.dart" | grep -v "kDebugMode\|// \|TODO\|FIXME"
```

### 5c: No External Dependencies Added Without Review
```bash
# Compare pubspec.yaml dependencies vs known list
# Flag any new packages not in docs/02
```

### 5d: Build Still Works
Check if recent changes might break build:
- Any import pointing to deleted file?
- Any widget referencing removed provider?
- Any route pointing to non-existent screen?

## Output Format

```markdown
# 🛡️ KFSP Guardian Report
**Date:** [timestamp]
**Mode:** Standard / Deep
**Source:** [path]

## Health Score: [X]/100

| Category | Score | Findings |
|----------|-------|----------|
| 🏗️ Structure | /25 | [summary] |
| 🎨 Design System | /25 | [summary] |
| 🔗 Cross-Cutting | /25 | [summary] |
| 📐 Spec Alignment | /15 | [summary] |
| 🛡️ Agent Safety | /10 | [summary] |

## 🔴 Critical (Must Fix)
[list]

## 🟡 Warnings (Should Fix)
[list]

## 🟢 Good Practices Detected
[list positive findings]

## 📊 Trends (vs last run)
[if previous report exists, compare]

## Recommended Actions
1. [prioritized]
```

## Nếu `--deep` mode:

Thêm các check:
- Content-level analysis: đọc từng widget file, check logic consistency
- Cross-file data flow tracing: follow data from mock_data → provider → widget → UI
- Performance check: find widgets that rebuild unnecessarily (missing const, heavy build methods)
- Accessibility check: find widgets without semantics labels
</instructions>
