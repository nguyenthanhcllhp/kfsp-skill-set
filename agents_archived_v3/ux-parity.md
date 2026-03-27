---
name: kfsp:ux-parity
description: Check design-code parity — compare actual UI vs Figma/prototype for spacing, colors, typography, layout drift
argument-hint: "[screen-name-or-feature] [--quick]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP UX Parity Check — So sánh UI thực tế với Figma/HTML prototype.

Design drift là khi code dần dần khác đi so với design gốc, do:
- Dev hardcode giá trị thay vì dùng token
- Spacing/padding bị sửa "cho đẹp" nhưng không update Figma
- Font size/weight khác Figma
- Color dùng sai shade
- Layout khác (flex direction, alignment)

**Modes:**
- `--quick` (~30s): Chỉ check token compliance (hardcoded vs token)
- Default (~2 phút): Check token + layout logic + typography
- Specific screen: Deep check 1 screen

**Khi nào dùng:**
- Sau khi dev build xong screen mới
- Sau khi sửa UI
- Trước release → chạy all screens
- Khi designer báo "nhìn khác design"
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- Tên screen/feature → check riêng screen đó
- `--quick` → chỉ token compliance
- Rỗng → check all screens

## Source Path Detection
```bash
for path in "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter" "Product/kfsp_flutter_migration/source"; do
  [ -d "$path/lib" ] && SOURCE="$path" && break
done
```

## Reference Files
- Design tokens: `$SOURCE/lib/core/theme/kfsp_colors.dart`
- Spacing: `$SOURCE/lib/core/theme/kfsp_spacing.dart`
- Typography: `$SOURCE/lib/core/theme/kfsp_text_styles.dart`
- Figma reference: `Product/Design/CLAUDE.md`
- Token doc: `Product/kfsp_flutter_migration/docs/03_DESIGN_SYSTEM.md`

## Check 1: Color Token Compliance 🎨

```bash
# Find hardcoded colors in feature files
grep -rn "Color(0x\|Color\.fromRGBO\|Colors\.\|#[0-9a-fA-F]\{6\}" "$SOURCE/lib/features/" --include="*.dart"
# Exclude: kfsp_colors.dart, test files, comments
# Each match = violation
```

Check: Are all colors accessed via `context.kfspColors.*`?

Violations:
- 🔴 Hardcoded hex color in widget → must use token
- 🟡 Using `Colors.white/black/grey` → should use token
- 🟢 Color in test/mock → acceptable

## Check 2: Spacing Token Compliance 📏

```bash
# Find hardcoded spacing
grep -rn "EdgeInsets\.\(all\|symmetric\|only\|fromLTRB\)(" "$SOURCE/lib/features/" --include="*.dart"
# Check: are values using context.kfspSpacing.* or hardcoded numbers?
grep -rn "SizedBox(\s*\(width\|height\):\s*[0-9]" "$SOURCE/lib/features/" --include="*.dart"
# Hardcoded SizedBox spacers
```

Standard spacing scale (from kfsp_spacing.dart):
- xs: 4, sm: 8, md: 12, lg: 16, xl: 20, xxl: 24, xxxl: 32

## Check 3: Typography Compliance 🔤

```bash
# Find hardcoded text styles
grep -rn "TextStyle(\|fontSize:\|fontWeight:\|fontFamily:" "$SOURCE/lib/features/" --include="*.dart"
# Should use context.kfspTextStyles.*
# Check font family is consistent
```

## Check 4: Layout Patterns 📐

For each screen widget, check:
```bash
# Screen uses Scaffold correctly
grep -rn "Scaffold(" "$SOURCE/lib/features/" --include="*.dart"
# AppBar follows pattern
grep -rn "AppBar(" "$SOURCE/lib/features/" --include="*.dart"
# SafeArea usage
grep -rn "SafeArea(" "$SOURCE/lib/features/" --include="*.dart"
```

## Check 5: Figma Cross-Reference 🎯

If specific screen requested, read the corresponding Figma data:
- Screen name → find in Product/Design/screenshots-by-feature/
- Compare: color palette, spacing, component structure
- Note any discrepancy

## Check 6: Dark Mode Readiness 🌙 (if applicable)

```bash
# Check if widgets use theme-aware colors
grep -rn "Theme\.of(context)\|context\.kfspColors" "$SOURCE/lib/features/" --include="*.dart"
# Check: hardcoded Colors.white in text → will fail in dark mode
```

## Output Format

```markdown
# 🎨 KFSP UX Parity Report
**Date:** [timestamp]
**Scope:** [all screens / specific screen]
**Mode:** Quick / Standard

## Parity Score: [X]%

| Check | Violations | Score |
|-------|-----------|-------|
| 🎨 Colors | X hardcoded | X% compliant |
| 📏 Spacing | X hardcoded | X% compliant |
| 🔤 Typography | X hardcoded | X% compliant |
| 📐 Layout | X issues | X% compliant |
| 🎯 Figma Match | X drifts | X% match |
| 🌙 Dark Mode | X issues | X% ready |

## Violations Detail

### 🔴 Must Fix (breaks design system)
| File | Line | Issue | Expected | Actual |
|------|------|-------|----------|--------|

### 🟡 Should Fix (drift from design)
| File | Line | Issue | Expected | Actual |
|------|------|-------|----------|--------|

### 🟢 Acceptable (minor, can defer)
[list]

## Quick Fixes
[copy-paste-ready code fixes for common violations]
```
</instructions>
