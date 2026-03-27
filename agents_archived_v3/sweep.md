---
name: kfsp:sweep
description: Run full impact analysis after code changes — detect blast radius, hidden dependencies, sync issues across design tokens, routes, providers, widgets, tests, and docs
argument-hint: "[changed-file-or-area]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Impact Sweep — Quét tầm soát toàn diện sau khi thay đổi code.

Khi dev thay đổi bất kỳ file/component nào, sweep sẽ:
1. Xác định blast radius (file nào bị ảnh hưởng thứ cấp)
2. Kiểm tra đồng bộ design tokens, routes, providers, widgets
3. Phát hiện hardcoded values, missing imports, broken references
4. Báo cáo rủi ro + đề xuất action items

**KHÔNG tự sửa code** — chỉ báo cáo. Dev quyết định sửa.
</objective>

<instructions>
## Input
User có thể cung cấp:
- Tên file vừa thay đổi (e.g., `kfsp_colors.dart`)
- Tên area (e.g., "design tokens", "routing", "onboarding")
- Không cung cấp gì → quét toàn bộ (full sweep)

Nếu argument là `$ARGUMENTS`, parse nó. Nếu rỗng, chạy full sweep.

## Step 1: Xác định scope thay đổi

Đọc dependency map reference:
```
Product/kfsp_flutter_migration/docs/13_DEPENDENCY_IMPACT_MAP.md
```

Nếu user chỉ định file cụ thể → tra bảng Impact Matrix trong doc 13 để tìm blast radius.
Nếu full sweep → kiểm tra tất cả categories.

## Step 2: Design Token Compliance & Adoption (🎨)

Quét source code Flutter tại source path detected in Step 1.

### 2A: Token Adoption Percentage

Calculate overall design system adoption rate:

```bash
SOURCE="[detected source path]"

# Exclude token definition files, tests, generated code
EXCLUDE="kfsp_colors.dart\|kfsp_text_styles.dart\|kfsp_spacing.dart\|kfsp_animations.dart\|kfsp_shadows.dart\|kfsp_buttons.dart\|_test.dart\|\.g\.dart\|\.freezed\.dart"

# Count violations (hardcoded)
HC_COLORS=$(grep -rn "Color(0x" "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE\|theme/\|constants/" | wc -l | tr -d ' ')
HC_MATCOLORS=$(grep -rn "Colors\." "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE\|theme/\|constants/\|Colors\.transparent" | wc -l | tr -d ' ')
HC_FONTSIZE=$(grep -rn "fontSize:" "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE\|theme/\|constants/" | wc -l | tr -d ' ')
HC_EDGEINSETS=$(grep -rnE "EdgeInsets\.(all|only|symmetric|fromLTRB)\s*\([0-9]" "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE\|theme/\|constants/" | wc -l | tr -d ' ')

# Count token usages (proper)
TK_COLORS=$(grep -rn "KfspColors\.\|context\.kfspColors\." "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')
TK_TEXTSTYLES=$(grep -rn "KfspTextStyles\.\|context\.kfspTextStyles\." "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')
TK_SPACING=$(grep -rn "KfspSpacing\." "$SOURCE/lib/" --include="*.dart" | grep -v "$EXCLUDE" | wc -l | tr -d ' ')

TOTAL_HC=$((HC_COLORS + HC_MATCOLORS + HC_FONTSIZE + HC_EDGEINSETS))
TOTAL_TK=$((TK_COLORS + TK_TEXTSTYLES + TK_SPACING))
TOTAL=$((TOTAL_HC + TOTAL_TK))

if [ "$TOTAL" -gt 0 ]; then
  ADOPTION=$((TOTAL_TK * 100 / TOTAL))
else
  ADOPTION=100
fi

echo "📊 Design System Adoption: ${ADOPTION}% ($TOTAL_TK token / $TOTAL total)"
echo "   Colors:     $TK_COLORS token vs $((HC_COLORS + HC_MATCOLORS)) hardcoded"
echo "   TextStyles: $TK_TEXTSTYLES token vs $HC_FONTSIZE hardcoded"
echo "   Spacing:    $TK_SPACING token vs $HC_EDGEINSETS hardcoded"
```

### 2B: Per-File Violations in Changed Files

If user specified file or area, check only those. Otherwise check files changed in recent commits:

```bash
# Detect changed files (user-specified or git diff)
if [ -n "$INPUT_FILE" ]; then
  CHANGED="$INPUT_FILE"
else
  CHANGED=$(git diff --name-only HEAD~1..HEAD -- lib/ 2>/dev/null | grep "\.dart$" || echo "")
fi

for f in $CHANGED; do
  [ ! -f "$SOURCE/$f" ] && continue
  FULL="$SOURCE/$f"
  echo "--- $f ---"

  # Color(0xFF...
  grep -n "Color(0x" "$FULL" 2>/dev/null | grep -v "// token-ok" | while read -r line; do
    HEX=$(echo "$line" | grep -oP "Color\(0x[0-9A-Fa-f]+\)" | head -1)
    echo "  ⚠️ $line"
    echo "    → Replace with KfspColors.* or context.kfspColors.* (see token map below)"
  done

  # Colors.*
  grep -n "Colors\." "$FULL" 2>/dev/null | grep -v "Colors\.transparent\|// token-ok" | while read -r line; do
    echo "  ⚠️ $line"
    echo "    → Replace with KfspColors.* token"
  done

  # fontSize raw
  grep -n "fontSize:" "$FULL" 2>/dev/null | while read -r line; do
    SIZE=$(echo "$line" | grep -oP "fontSize:\s*\K[0-9]+")
    echo "  ⚠️ $line"
    echo "    → Replace with KfspTextStyles.* or context.kfspColors.font${SIZE}"
  done

  # EdgeInsets raw
  grep -nE "EdgeInsets\.(all|only|symmetric|fromLTRB)\s*\([0-9]" "$FULL" 2>/dev/null | while read -r line; do
    echo "  ⚠️ $line"
    echo "    → Replace with KfspSpacing.* token"
  done
done
```

### 2C: Common Token Replacement Map

**Colors:**
| Hardcoded | Token |
|-----------|-------|
| `Color(0xFF7B3AEC)` | `KfspColors.brandPrimary` |
| `Color(0xFF0DBE55)` | `KfspColors.stockUp` |
| `Color(0xFFFF3B30)` | `KfspColors.stockDown` |
| `Color(0xFFFF763D)` | `KfspColors.stockRef` |
| `Color(0xFF000000)` | `KfspColors.textPrimary` |
| `Color(0xFF7E839E)` | `KfspColors.textSecondary` |
| `Color(0xFFF4F4F4)` | `KfspColors.surface` |
| `Colors.white` | `KfspColors.backgroundPrimary` |
| `Colors.black` | `KfspColors.textPrimary` |
| `Colors.grey` | `KfspColors.textSecondary` |
| `Colors.red` | `KfspColors.stockDown` |
| `Colors.green` | `KfspColors.stockUp` |
| `Colors.transparent` | OK — no token needed |

**FontSize:**
| Hardcoded | Token |
|-----------|-------|
| `fontSize: 10` | `KfspTextStyles.caption` / `context.kfspColors.font10` |
| `fontSize: 12` | `KfspTextStyles.bodySmall` / `context.kfspColors.font12` |
| `fontSize: 14` | `KfspTextStyles.bodyMedium` / `context.kfspColors.font14` |
| `fontSize: 16` | `KfspTextStyles.bodyLarge` / `context.kfspColors.font16` |
| `fontSize: 18` | `KfspTextStyles.titleSmall` / `context.kfspColors.font18` |
| `fontSize: 20` | `KfspTextStyles.titleMedium` / `context.kfspColors.font20` |
| `fontSize: 24` | `KfspTextStyles.titleLarge` / `context.kfspColors.font24` |

**Spacing:**
| Hardcoded | Token |
|-----------|-------|
| `EdgeInsets.all(4)` | `KfspSpacing.paddingXS` |
| `EdgeInsets.all(8)` | `KfspSpacing.paddingS` |
| `EdgeInsets.all(12)` | `KfspSpacing.paddingSM` |
| `EdgeInsets.all(16)` | `KfspSpacing.paddingM` |
| `EdgeInsets.all(24)` | `KfspSpacing.paddingXL` |
| `SizedBox(height: 4)` | `KfspSpacing.verticalXS` |
| `SizedBox(height: 8)` | `KfspSpacing.verticalS` |
| `SizedBox(height: 16)` | `KfspSpacing.verticalM` |
| `SizedBox(width: 8)` | `KfspSpacing.horizontalS` |
| `SizedBox(width: 16)` | `KfspSpacing.horizontalM` |

> **Note:** If no matching token exists, ADD the token to the definition file first, then reference it. Never hardcode.

Mỗi violation = 1 finding với file:line + severity + suggested token replacement.

## Step 3: Route & Navigation Check (🧭)

```
Grep: GoRoute( → liệt kê tất cả routes
Grep: context.go( hoặc context.push( → liệt kê tất cả navigation calls
Cross-check: mỗi navigation call có route tương ứng không?
Check: app_router.dart → mọi route có screen import đúng không?
```

## Step 4: Provider Dependency Check (⚡)

```
Grep: Provider( hoặc Notifier( → liệt kê providers
Grep: ref.watch( hoặc ref.read( → liệt kê consumers
Cross-check: mỗi consumer ref đến provider tồn tại?
Check: circular dependency? (A watch B, B watch A)
```

## Step 5: Widget Tree Integrity (🧩)

```
Grep: import.*widgets/ → check import paths valid
Grep: class.*extends.*Widget → liệt kê tất cả widgets
Check: mỗi widget có được sử dụng ít nhất 1 nơi? (dead widget detection)
```

## Step 6: Real Data Integration + Offline Fallback (📊) — REAL DATA MODE

```
Read: stock_provider.dart → check real API/socket integration
Grep: SocketClient\|emitWithAck\|api_client → liệt kê tất cả real data sources
Check: mỗi screen có offline fallback khi API/socket unavailable?
Read: mock_data.dart → verify fallback structure matches real API response fields
Check: socket join/leave lifecycle đúng? (emit join trước on, leave khi dispose)
```

## Step 7: Test Coverage Gap (🧪)

```
Glob: test/**/*_test.dart → existing tests
Cross-check với lib/ files: file nào chưa có test tương ứng?
Check: test file nào reference file đã bị xóa/rename?
```

## Step 8: Doc Sync Check (📝)

Kiểm tra:
- docs/08_HANDOFF_STATUS.md → progress bars có khớp thực tế?
- docs/04_FEATURE_SPECS.md → features listed = features trong code?
- README.md → known issues list có cập nhật?

## Output Format

Tạo báo cáo theo format:

```markdown
# 🔍 KFSP Impact Sweep Report
**Date:** [timestamp]
**Scope:** [full / specific files]
**Source:** [path to source code scanned]

## Summary
| Category | Findings | Critical | Warning | Info |
|----------|----------|----------|---------|------|
| 🎨 Design Tokens | X | X | X | X |
| 📊 Token Adoption | X% (Y token / Z total) | — | — | — |
| 🧭 Routes | X | X | X | X |
| ⚡ Providers | X | X | X | X |
| 🧩 Widgets | X | X | X | X |
| 📊 Real Data | X | X | X | X |
| 🧪 Tests | X | X | X | X |
| 📝 Docs | X | X | X | X |

## 🎨 Design Token Adoption Detail
| Type | Token Usage | Hardcoded | Adoption |
|------|------------|-----------|----------|
| Colors | X | Y | Z% |
| TextStyles | X | Y | Z% |
| Spacing | X | Y | Z% |
| **Total** | **X** | **Y** | **Z%** |

### Files with Hardcoded Values (changed files only)
| File | Color(0x | Colors.* | fontSize | EdgeInsets | Suggestions |
|------|----------|----------|----------|------------|-------------|
| path/file.dart | 3 | 1 | 2 | 0 | See replacement map |

## 🔴 Critical Findings
[list with file:line, description, suggested fix]

## 🟡 Warnings
[list]

## 🟢 Info
[list]

## Recommended Actions
1. [prioritized action items]
```

Nếu KHÔNG có findings → báo "✅ Clean sweep — no issues detected."
</instructions>
