---
name: kfsp:surface-test
description: Test interaction contracts across 4 surfaces — user-facing, backend APIs, third-party services, and internal components
argument-hint: "[surface: front|back|lateral|internal|all] [--quick]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Surface Test — Kiểm tra "hợp đồng tương tác" trên 4 bề mặt.

Mỗi bề mặt tương tác có CONTRACT (input/output kỳ vọng), STANDARD (ngưỡng chấp nhận),
và cần được TEST định kỳ.

4 Surfaces:
1. **Front** (User ↔ App): UX flows, Pro/Free gates, navigation, deep links
2. **Back** (App ↔ Backend): REST APIs, WebSocket, Auth, Data format
3. **Lateral** (App ↔ 3rd party): TradingView, Firebase, Analytics, Social login
4. **Internal** (Component ↔ Component): Theme, Router, Providers, Shared widgets

**Modes:**
- `--quick` (~1 phút): Chỉ check contracts có đúng format/tồn tại
- Default (~3 phút): Check contracts + validate data flow
- `all` (~5 phút): Test tất cả 4 surfaces

**Khi nào dùng:**
- Trước mỗi release → `all`
- Sau khi thay đổi API → `back`
- Sau khi thay đổi UI → `front`
- Sau khi update 3rd party SDK → `lateral`
- Sau khi refactor shared code → `internal`
</objective>

<instructions>
## Parse Input

`$ARGUMENTS`:
- `front` / `back` / `lateral` / `internal` / `all` → chọn surface
- `--quick` → quick mode
- Rỗng → `all` default

## Reference Docs
- `Product/kfsp_flutter_migration/docs/10_API_SOCKET_SPEC.md` → API contracts
- `Product/kfsp_flutter_migration/docs/13_DEPENDENCY_IMPACT_MAP.md` → dependency graph
- `Product/kfsp_flutter_migration/docs/02_ARCHITECTURE_GUIDE.md` → architecture

## SURFACE 1: FRONT (User ↔ App) 🔵

### 1a: Navigation Contract
```bash
# Extract all routes from app_router.dart
grep -n "GoRoute\|path:" lib/config/routes/app_router.dart
# Check: every route has a screen widget
# Check: no orphan screens (screen exists but no route)
```

### 1b: Pro/Free Gate Contract
```bash
# Find all Pro/Free conditional logic
grep -rn "isPro\|isFree\|userTier\|subscription\|ProFeature\|FreeLimit" lib/ --include="*.dart"
# Check: every gated feature has both Pro and Free variant
# Check: Free users see blur/lock, not crash
```

### 1c: User Flow Completeness
For each main flow:
- Onboarding → Dashboard: entry point clear?
- Search → Stock Detail: navigation params correct?
- Alert create → Alert list: state refresh after create?
- Filter → Results: filter params preserved on back?

### 1d: Error/Empty States
```bash
# Check each screen has error/empty state handling
grep -rn "ErrorWidget\|EmptyState\|NoData\|error.*state\|empty.*state" lib/features/ --include="*.dart"
# Cross-check with screen count: ratio of error-handled screens
```

## SURFACE 2: BACK (App ↔ Backend) 🔴

### 2a: API + Socket Endpoint Coverage (REAL DATA MODE)
```bash
# Extract API calls from code (REST + Socket.IO)
grep -rn "\.get(\|\.post(\|\.put(\|\.delete(\|emitWithAck\|SocketClient" lib/ --include="*.dart" | grep -v "_test.dart"
# Cross-check with docs/10_API_SOCKET_SPEC.md
# Flag: API/socket event in code but not in doc, or vice versa
# Verify socket join/leave lifecycle
grep -rn "emit.*join\|emit.*leave" lib/ --include="*.dart"
```

### 2b: Response Model Validation
```bash
# Extract model classes
grep -rn "class.*Model\|class.*Entity\|class.*Response\|fromJson\|toJson" lib/ --include="*.dart"
# Check: every API has a corresponding model
# Check: fromJson handles null/missing fields gracefully
```

### 2c: Real Data ↔ Offline Fallback Contract (REAL DATA MODE)
```bash
# Verify fallback mock structure matches real API/socket response
# Mock data is ONLY for offline fallback — NOT primary source
grep -rn "MockData\.\|mock_data" lib/ --include="*.dart" | grep -v "_test.dart"
# Flag: screen using mock as primary data source (should use real API)
# Verify: socket event field names match model fields (known gotcha: updateliveindex uses price/vol/totalval)
```

### 2d: Auth Flow Contract
```bash
# Check token handling
grep -rn "token\|Token\|Bearer\|auth\|Auth\|refresh" lib/ --include="*.dart" | grep -v "_test.dart"
# Check: token refresh flow exists
# Check: 401 response handling exists
# Check: logout flow clears all state
```

### 2e: WebSocket Contract
```bash
# Check socket event handlers
grep -rn "socket\|Socket\|\.on(\|\.emit(\|\.connect\|\.disconnect" lib/ --include="*.dart"
# Check: reconnect logic
# Check: event format matches doc
```

## SURFACE 3: LATERAL (App ↔ 3rd Party Services) 🟡

### 3a: TradingView Integration
```bash
# Check TradingView widget configuration
grep -rn "TradingView\|tradingview\|trading.view\|InAppWebView.*chart" lib/ --include="*.dart"
# Check: widget params match docs/05_TRADINGVIEW_INTEGRATION.md
# Check: error handling if chart fails to load
```

### 3b: Firebase/FCM
```bash
# Check push notification setup
grep -rn "firebase\|Firebase\|FCM\|messaging\|notification" lib/ --include="*.dart"
grep -rn "firebase" pubspec.yaml
# Check: FCM token registration
# Check: notification handler (foreground + background)
```

### 3c: Analytics Events
```bash
# Check analytics tracking
grep -rn "analytics\|Analytics\|trackEvent\|logEvent\|track(" lib/ --include="*.dart"
# Check: key user actions are tracked
# Check: event naming convention consistent
```

### 3d: External SDK Versions
```bash
# Check pubspec.yaml for 3rd party versions
grep -A1 "tradingview\|firebase\|google_sign_in\|apple\|sentry\|crashlytics\|analytics" pubspec.yaml
# Flag: outdated versions with known issues
```

## SURFACE 4: INTERNAL (Component ↔ Component) 🟢

### 4a: Theme Cascade
```bash
# Verify theme chain is intact
# kfsp_colors.dart → kfsp_theme_colors.dart → kfsp_theme.dart → context_extensions.dart
for file in "kfsp_colors" "kfsp_theme_colors" "kfsp_theme" "context_extensions"; do
  find lib/ -name "${file}.dart" -exec echo "✅ Found: {}" \;
done
# Check: all colors in kfsp_colors are exposed in theme_colors
# Check: theme_colors is registered in kfsp_theme
# Check: context_extensions provides shortcuts for all
```

### 4b: Provider Graph
```bash
# List all providers
grep -rn "final.*Provider\|final.*Notifier" lib/ --include="*.dart" | grep -v "_test.dart"
# Check: no circular dependencies
# Check: every provider is consumed somewhere
# Check: no orphan providers
```

### 4c: Import Health
```bash
# Check for relative vs package imports
grep -rn "import '\.\." lib/ --include="*.dart" | head -20
# Should use package: imports for cross-feature references
# Relative imports OK within same feature
```

### 4d: Shared Widget Contracts
```bash
# List shared widgets
find lib/common/widgets/ -name "*.dart" | sort
# For each: check required params haven't changed
# Check: no widget depends on feature-specific code
```

## Output Format

```markdown
# 🧪 KFSP Surface Test Report
**Date:** [timestamp]
**Surfaces Tested:** [front/back/lateral/internal/all]
**Mode:** Quick / Standard

## Surface Health Dashboard
| Surface | Tests | Pass | Fail | Warn | Health |
|---------|-------|------|------|------|--------|
| 🔵 Front (User↔App) | X | X | X | X | X% |
| 🔴 Back (App↔Backend) | X | X | X | X | X% |
| 🟡 Lateral (App↔Services) | X | X | X | X | X% |
| 🟢 Internal (Comp↔Comp) | X | X | X | X | X% |
| **TOTAL** | X | X | X | X | **X%** |

## Contract Violations
[list of failed contract checks with severity]

## Missing Contracts
[interactions that don't have clear contracts defined]

## Recommendations
1. [prioritized]

## Next Steps
- [ ] Fix critical violations
- [ ] Document missing contracts
- [ ] Add automated tests for key contracts
```
</instructions>
</content>
</invoke>