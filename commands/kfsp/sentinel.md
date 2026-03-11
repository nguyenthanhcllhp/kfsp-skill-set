---
name: kfsp:sentinel
description: Regression sentinel — detect if existing features still work after changes. Compare before/after state across screens, flows, and data
argument-hint: "[--baseline] [--compare] [--quick]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Regression Sentinel — Phát hiện feature cũ bị break sau thay đổi mới.

Khác với `/kfsp:sweep` (chạy 1 lần sau thay đổi), Sentinel:
- Tạo baseline snapshot TRƯỚC thay đổi
- So sánh AFTER snapshot với baseline
- Phát hiện regression trên 4 bề mặt

**Workflow:**
1. TRƯỚC thay đổi: `/kfsp:sentinel --baseline` → chụp snapshot
2. SAU thay đổi: `/kfsp:sentinel --compare` → so sánh với baseline
3. Quick check: `/kfsp:sentinel --quick` → chỉ check critical paths

**Khi nào dùng:**
- Trước/sau refactor lớn
- Trước/sau upgrade dependency
- Trước/sau merge branch
- Periodic (tuần) để detect drift dần dần
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `--baseline` → tạo baseline snapshot
- `--compare` → so sánh với baseline
- `--quick` → quick regression check (không cần baseline)
- Rỗng → `--quick`

## Source Path Detection
```bash
for path in "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter" "Product/kfsp_flutter_migration/source"; do
  [ -d "$path/lib" ] && SOURCE="$path" && break
done
SENTINEL_DIR="$SOURCE/.sentinel"
```

## Mode: --baseline (Snapshot Before)

Capture current state:

```bash
mkdir -p "$SENTINEL_DIR"
```

### Snapshot 1: File Inventory
```bash
find "$SOURCE/lib" -name "*.dart" | sort > "$SENTINEL_DIR/files.txt"
wc -l "$SENTINEL_DIR/files.txt"
```

### Snapshot 2: Public API Surface
```bash
# All public classes and their method signatures
grep -rn "class \|void \|Future<\|Widget \|String \|int \|double \|bool " "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart" | grep -v "^.*:.*//\|^.*:.*\*" | sort > "$SENTINEL_DIR/api_surface.txt"
```

### Snapshot 3: Route Map
```bash
grep -n "GoRoute\|path:\|name:" "$SOURCE/lib/config/routes/app_router.dart" > "$SENTINEL_DIR/routes.txt"
```

### Snapshot 4: Provider List
```bash
grep -rn "final.*Provider\|final.*Notifier\|class.*Notifier" "$SOURCE/lib/" --include="*.dart" | sort > "$SENTINEL_DIR/providers.txt"
```

### Snapshot 5: Design Token Values
```bash
cat "$SOURCE/lib/core/theme/kfsp_colors.dart" > "$SENTINEL_DIR/colors_snapshot.txt"
cat "$SOURCE/lib/core/theme/kfsp_spacing.dart" > "$SENTINEL_DIR/spacing_snapshot.txt"
```

### Snapshot 6: Import Graph
```bash
grep -rn "^import " "$SOURCE/lib/" --include="*.dart" | sort > "$SENTINEL_DIR/imports.txt"
```

### Snapshot 7: pubspec.yaml deps
```bash
cat "$SOURCE/pubspec.yaml" > "$SENTINEL_DIR/pubspec_snapshot.txt"
```

Output: "✅ Baseline captured at [timestamp]. X files, X routes, X providers."

## Mode: --compare (Diff After)

For each snapshot, generate current state and diff:

### Compare Files
```bash
find "$SOURCE/lib" -name "*.dart" | sort > "$SENTINEL_DIR/files_current.txt"
diff "$SENTINEL_DIR/files.txt" "$SENTINEL_DIR/files_current.txt"
# Added files (new features)
# Removed files (⚠️ deleted!)
# Renamed files (moved?)
```

### Compare API Surface
```bash
grep -rn "class \|void \|Future<\|Widget \|String \|int \|double \|bool " "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart" | sort > "$SENTINEL_DIR/api_surface_current.txt"
diff "$SENTINEL_DIR/api_surface.txt" "$SENTINEL_DIR/api_surface_current.txt"
# Changed signatures → breaking change!
# Removed methods → regression risk!
```

### Compare Routes
```bash
grep -n "GoRoute\|path:\|name:" "$SOURCE/lib/config/routes/app_router.dart" > "$SENTINEL_DIR/routes_current.txt"
diff "$SENTINEL_DIR/routes.txt" "$SENTINEL_DIR/routes_current.txt"
# Removed routes → navigation break!
# Changed paths → deep link break!
```

### Compare Providers
```bash
grep -rn "final.*Provider\|final.*Notifier\|class.*Notifier" "$SOURCE/lib/" --include="*.dart" | sort > "$SENTINEL_DIR/providers_current.txt"
diff "$SENTINEL_DIR/providers.txt" "$SENTINEL_DIR/providers_current.txt"
```

### Compare Design Tokens
```bash
diff "$SENTINEL_DIR/colors_snapshot.txt" "$SOURCE/lib/core/theme/kfsp_colors.dart"
diff "$SENTINEL_DIR/spacing_snapshot.txt" "$SOURCE/lib/core/theme/kfsp_spacing.dart"
# Token value changed → 45+ files affected!
```

### Compare Imports
```bash
grep -rn "^import " "$SOURCE/lib/" --include="*.dart" | sort > "$SENTINEL_DIR/imports_current.txt"
diff "$SENTINEL_DIR/imports.txt" "$SENTINEL_DIR/imports_current.txt"
# Broken imports → build error!
```

### Compare Dependencies
```bash
diff "$SENTINEL_DIR/pubspec_snapshot.txt" "$SOURCE/pubspec.yaml"
# Version bumps → compatibility risk!
```

## Mode: --quick (No Baseline Needed)

Quick regression check based on code analysis:

### Critical Path 1: App Starts
```bash
# main.dart exists and imports correctly
head -30 "$SOURCE/lib/main.dart"
# Check: ProviderScope → MaterialApp → GoRouter chain intact
```

### Critical Path 2: Navigation Works
```bash
# All routes resolve to existing screens
grep "path:" "$SOURCE/lib/config/routes/app_router.dart"
# Check each builder/pageBuilder import exists
```

### Critical Path 3: Theme Loads
```bash
# Theme chain intact
ls -la "$SOURCE/lib/core/theme/"
# All 5 theme files exist and have content
```

### Critical Path 4: Features Accessible
```bash
# Each feature folder has at least 1 screen
for feature in $(ls "$SOURCE/lib/features/" 2>/dev/null); do
  count=$(find "$SOURCE/lib/features/$feature" -name "*screen*.dart" -o -name "*page*.dart" | wc -l)
  echo "$feature: $count screens"
done
```

## Output Format

### For --baseline:
```markdown
# 📸 KFSP Sentinel Baseline
**Captured:** [timestamp]
**Source:** [path]
**Stats:** X files | X routes | X providers | X tokens
**Next:** Make your changes, then run `/kfsp:sentinel --compare`
```

### For --compare:
```markdown
# 🔍 KFSP Sentinel Regression Report
**Baseline:** [timestamp]
**Current:** [timestamp]
**Changes detected:** X

## Change Summary
| Category | Added | Removed | Modified |
|----------|-------|---------|----------|
| Files | X | X | X |
| Routes | X | X | X |
| Providers | X | X | X |
| Tokens | X | X | X |
| Dependencies | X | X | X |
| Imports | X | X | X |

## 🔴 Regression Risks
[removed files, changed signatures, broken imports]

## 🟡 Review Needed
[changed tokens, new dependencies, modified providers]

## 🟢 Safe Changes
[new files, new routes, added tests]

## Recommended Actions
1. [prioritized]
```

### For --quick:
```markdown
# ⚡ KFSP Quick Regression Check
**Date:** [timestamp]
**Result:** ✅ All critical paths OK / ⚠️ X issues found

| Critical Path | Status |
|---------------|--------|
| App Starts | ✅/❌ |
| Navigation | ✅/❌ |
| Theme | ✅/❌ |
| Features | ✅/❌ |

[details if issues found]
```
</instructions>
