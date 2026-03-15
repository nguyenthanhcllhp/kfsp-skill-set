---
name: kfsp:release-gate
description: Release readiness gate — comprehensive pre-release checklist covering all 4 surfaces, performance, security, accessibility, store compliance
argument-hint: "[--quick] [--store-check]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Release Gate — Cổng kiểm tra trước khi release.

**Triết lý:** Không ai muốn release app rồi mới phát hiện bug.
Release Gate là checkpoint cuối cùng TRƯỚC KHI submit lên TestFlight/Store.

**Modes:**
- `--quick` (~2 phút): Top 10 critical checks only
- Default (~5-8 phút): Full gate (tất cả checks)
- `--store-check`: Thêm store-specific requirements (iOS/Android)

**Khi nào dùng:**
- Trước khi build TestFlight
- Trước khi submit App Store / Google Play
- Trước release nội bộ (UAT)
- Sprint review cuối sprint
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `--quick` → top 10 checks
- `--store-check` → thêm store compliance
- Rỗng → full gate (no store check)

## Source Path Detection
```bash
for path in "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter" "Product/kfsp_flutter_migration/source"; do
  [ -d "$path/lib" ] && SOURCE="$path" && break
done
```

## GATE 1: 🏗️ Build Health (Pass/Fail)

### 1a: Build Compiles
```bash
# Check for obvious build breakers
# Missing imports
grep -rn "^import " "$SOURCE/lib/" --include="*.dart" | while read line; do
  file=$(echo "$line" | sed "s/:.*//")
  imported=$(echo "$line" | grep -oP "import '.*?'" | sed "s/import '//;s/'//")
  # Check if imported file exists (for package imports, skip)
done
```

### 1b: No Debug Code in Production
```bash
# These should NOT be in release
grep -rn "print(\|debugPrint(\|debugger\b\|assert(" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart" | grep -v "kDebugMode"
# kDebugMode is OK (it's stripped in release)
```

### 1c: Version Number Updated
```bash
grep "version:" "$SOURCE/pubspec.yaml" | head -1
# Check: version has been bumped from last release
```

## GATE 2: 🔐 Security (Pass/Fail)

### 2a: No Secrets in Code
```bash
grep -rni "password.*=.*['\"].*['\"]" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart" | grep -v "kDebugMode\|mock\|example"
grep -rni "api.key.*=.*['\"].*['\"]" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart"
grep -rni "secret.*=.*['\"].*['\"]" "$SOURCE/lib/" --include="*.dart" | grep -v "_test.dart"
```

### 2b: Network Security
```bash
# No HTTP (should be HTTPS)
grep -rn "http://" "$SOURCE/lib/" --include="*.dart" | grep -v "localhost\|127.0.0.1\|_test.dart"
```

### 2c: Secure Storage
```bash
# Tokens stored in secure storage, not SharedPreferences
grep -rn "SharedPreferences.*token\|prefs.*token" "$SOURCE/lib/" --include="*.dart" | grep -vi "secure"
# Should be empty — tokens must use secure storage
```

## GATE 3: 🎨 Design System Compliance

Run quick version of ux-parity check:
```bash
# Hardcoded colors count
HC_COLORS=$(grep -rn "Color(0x" "$SOURCE/lib/features/" --include="*.dart" | wc -l)
# Hardcoded spacing count
HC_SPACING=$(grep -rn "EdgeInsets\.\(all\|symmetric\)([0-9]" "$SOURCE/lib/features/" --include="*.dart" | wc -l)
echo "Hardcoded colors: $HC_COLORS, spacing: $HC_SPACING"
```

## GATE 4: 📱 UX Quality

### 4a: Error Handling Coverage
```bash
# Screens with error states
ERROR_SCREENS=$(grep -rl "ErrorWidget\|error.*state\|AsyncValue.*error\|\.error(" "$SOURCE/lib/features/" --include="*.dart" | wc -l)
TOTAL_SCREENS=$(find "$SOURCE/lib/features/" -name "*screen*.dart" -o -name "*page*.dart" | wc -l)
echo "Error handling: $ERROR_SCREENS / $TOTAL_SCREENS screens"
```

### 4b: Loading States
```bash
# Screens with loading indicators
LOADING_SCREENS=$(grep -rl "CircularProgressIndicator\|Loading\|loading.*state\|AsyncValue.*loading\|\.isLoading" "$SOURCE/lib/features/" --include="*.dart" | wc -l)
echo "Loading states: $LOADING_SCREENS / $TOTAL_SCREENS screens"
```

### 4c: Empty States
```bash
EMPTY_SCREENS=$(grep -rl "EmptyState\|empty.*state\|NoData\|no.*data" "$SOURCE/lib/features/" --include="*.dart" | wc -l)
echo "Empty states: $EMPTY_SCREENS / $TOTAL_SCREENS screens"
```

## GATE 5: 🧪 Test Coverage

```bash
# Count test files
TEST_COUNT=$(find "$SOURCE/test/" -name "*_test.dart" 2>/dev/null | wc -l)
LIB_COUNT=$(find "$SOURCE/lib/" -name "*.dart" | wc -l)
echo "Tests: $TEST_COUNT, Source: $LIB_COUNT, Ratio: $(echo "scale=1; $TEST_COUNT * 100 / $LIB_COUNT" | bc)%"

# Master Test Registry coverage
REGISTRY="Product/kfsp_flutter_migration/docs/test_cases/master_test_registry.html"
if [ -f "$REGISTRY" ]; then
  TC_COUNT=$(grep -c '"id":' "$REGISTRY" 2>/dev/null || echo "0")
  echo "📋 Master Test Registry: $TC_COUNT test cases"
  # Check all screens have test cases
  SCREENS=$(find "$SOURCE/lib/screens" -maxdepth 1 -type d 2>/dev/null | wc -l)
  echo "Screens: $SCREENS, Test cases: $TC_COUNT"
fi
```

**Gate criteria:**
- Test registry exists with ≥50 test cases → ✅
- No registry → ⚠️ (-5 points)
- Critical screens without test cases → ⚠️ (-3 points per screen)

## GATE 6: 📦 Dependencies

```bash
# Check for outdated/deprecated packages
grep -A1 "dependencies:" "$SOURCE/pubspec.yaml"
# Count direct dependencies
DEPS=$(grep "^\s\s[a-z]" "$SOURCE/pubspec.yaml" | grep -v "#" | wc -l)
echo "Direct dependencies: $DEPS"
# Any ^ constraints that might break?
grep "\^" "$SOURCE/pubspec.yaml"
```

## GATE 7: 🔗 Surface Contract Validation

Quick check each surface:
- **Front**: All routes resolve, Pro/Free gates work
- **Back**: API models have fromJson, error handling exists
- **Lateral**: 3rd party configs valid (TradingView, Firebase)
- **Internal**: Theme chain intact, no circular deps

## GATE 8: 📝 Documentation Sync (if not --quick)

```bash
# Key docs are up to date
ls -la "Product/kfsp_flutter_migration/docs/08_HANDOFF_STATUS.md" 2>/dev/null
# Check: last modified date vs source code last modified
```

## GATE 9: 🍎🤖 Store Requirements (if --store-check)

### iOS
- [ ] App icon: 1024×1024 PNG (no alpha)
- [ ] Launch screen configured
- [ ] Info.plist: NSAppTransportSecurity, privacy descriptions
- [ ] Bundle ID correct
- [ ] Minimum iOS version set
- [ ] No UIWebView usage (deprecated)

### Android
- [ ] minSdkVersion ≥ 21
- [ ] targetSdkVersion current (API 34+)
- [ ] AndroidManifest permissions minimal
- [ ] ProGuard/R8 rules configured
- [ ] Signing key configured

Check from source:
```bash
# iOS
cat "$SOURCE/ios/Runner/Info.plist" | head -50
# Android
cat "$SOURCE/android/app/build.gradle" | grep -i "sdk\|version\|sign"
```

## GATE 10: 🚀 Rollback Readiness

- [ ] Previous version available for rollback
- [ ] Feature flags for new features (can disable remotely)
- [ ] Crash reporting configured (Sentry/Crashlytics)
- [ ] Analytics tracking on key flows

## Output Format

```markdown
# 🚦 KFSP Release Gate Report
**Date:** [timestamp]
**Version:** [from pubspec.yaml]
**Mode:** Quick / Full / Store Check

## Gate Results: [PASS ✅ / FAIL ❌ / WARN ⚠️]

| Gate | Status | Score | Details |
|------|--------|-------|---------|
| 🏗️ Build Health | ✅/❌ | X/10 | [summary] |
| 🔐 Security | ✅/❌ | X/10 | [summary] |
| 🎨 Design System | ✅/⚠️ | X/10 | [summary] |
| 📱 UX Quality | ✅/⚠️ | X/10 | [summary] |
| 🧪 Tests | ✅/⚠️ | X/10 | [summary] |
| 📦 Dependencies | ✅/⚠️ | X/10 | [summary] |
| 🔗 Surface Contracts | ✅/⚠️ | X/10 | [summary] |
| 📝 Docs | ✅/⚠️ | X/10 | [summary] |
| 🍎🤖 Store (if checked) | ✅/❌ | X/10 | [summary] |
| 🚀 Rollback | ✅/⚠️ | X/10 | [summary] |

## Overall: [X]/100

### Release Decision
- **Score ≥ 80, no ❌:** ✅ READY TO RELEASE
- **Score 60-79, no ❌:** ⚠️ RELEASE WITH CAUTION (fix warnings first)
- **Score < 60 OR any ❌:** 🛑 NOT READY — fix blockers first

## Blockers (must fix)
[list of ❌ items]

## Warnings (should fix)
[list of ⚠️ items]

## Release Notes Draft
[auto-generated from changes detected]
```
</instructions>
