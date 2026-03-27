# 14. KFSP Interaction Surface Map

> Bản đồ tương tác 4 bề mặt — mô tả, tiêu chuẩn, theo dõi, kiểm tra cho mọi điểm tiếp xúc của sản phẩm.

**Tại sao cần?** App không sống đơn lẻ. Nó tương tác với user phía trước, server phía sau, dịch vụ bên cạnh, và các component bên trong. Mỗi điểm tương tác là một rủi ro tiềm ẩn — thay đổi ở một đầu có thể gây hỏng đầu kia.

**Liên kết Skill:** `/kfsp:surface-test` dùng tài liệu này làm reference.

---

## Mô hình 4 bề mặt

```
                         ┌──────────────────────┐
                         │   🔵 PHÍA TRƯỚC       │
                         │   User ↔ App          │
                         │   13 interaction types│
                         └──────────┬───────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         │                          │                          │
┌────────┴─────────┐    ┌───────────┴──────────┐   ┌──────────┴────────┐
│ 🟡 PHƯƠNG NGANG  │    │    🟢 BÊN TRONG      │   │ 🟡 PHƯƠNG NGANG   │
│ (Left)           │    │    (Internal)         │   │ (Right)           │
│ Core Services    │    │    Component ↔ Comp   │   │ Analytics/Monitor │
│ 5 services       │    │    8 subsystems       │   │ 4 services        │
└──────────────────┘    └───────────┬──────────┘   └───────────────────┘
                                    │
                         ┌──────────┴───────────┐
                         │   🔴 PHÍA SAU         │
                         │   App ↔ Backend       │
                         │   6 interaction types │
                         └──────────────────────┘
```

---

## 🔵 SURFACE 1: PHÍA TRƯỚC (User ↔ App)

### Tổng quan
| ID | Tương tác | Direction | Criticality |
|----|-----------|-----------|-------------|
| F01 | App Launch → Dashboard | User→App | 🔴 Critical |
| F02 | Navigation giữa screens | Bidirectional | 🔴 Critical |
| F03 | Pro/Free Feature Gate | App→User | 🔴 Critical |
| F04 | Search & Filter | User→App→User | 🟡 High |
| F05 | Stock Detail Drill-down | User→App | 🟡 High |
| F06 | Alert Create/Manage | User→App→Backend | 🟡 High |
| F07 | Trading Journal Entry | User→App→Backend | 🟡 High |
| F08 | Push Notification → Deep Link | Backend→App→Screen | 🔴 Critical |
| F09 | Onboarding Flow | App→User | 🟡 High |
| F10 | Error/Empty States | App→User | 🟡 High |
| F11 | Loading States | App→User | 🟢 Medium |
| F12 | Pull-to-Refresh | User→App→Backend | 🟢 Medium |
| F13 | Settings/Preferences | User→App→Storage | 🟢 Medium |

### Contract Details

#### F01: App Launch → Dashboard
```
CONTRACT:
  Input:  User taps app icon
  Output: Dashboard screen with data, within 3 seconds

STANDARD:
  Cold start: ≤ 3s (first time)
  Warm start: ≤ 1.5s (cached)
  Data freshness: ≤ 30s old
  Error case: Show cached data + retry banner

MONITOR:
  - App startup time (instrument in main.dart)
  - Dashboard data load time
  - Crash-on-launch rate

TEST:
  - Integration: launch app → verify Dashboard visible
  - Widget: Dashboard renders with mock data
  - Edge: launch offline → shows cached/empty state

IMPACT IF BROKEN:
  🔴 User cannot use app at all → 1-star reviews, uninstalls
```

#### F02: Navigation giữa screens
```
CONTRACT:
  Input:  User taps navigation element (tab, button, card)
  Output: Correct screen opens with correct params, within 300ms

STANDARD:
  Transition: ≤ 300ms
  Back navigation: always returns to correct previous screen
  Deep link: opens exact screen with params

MONITOR:
  - Navigation failures (GoRouter errors)
  - Screen load time per route

TEST:
  - Integration: navigate to every screen from every entry point
  - Widget: each screen renders independently
  - Edge: invalid route → error screen (not crash)

IMPACT IF BROKEN:
  🔴 User gets stuck, can't reach features → critical UX failure
```

#### F03: Pro/Free Feature Gate
```
CONTRACT:
  Input:  User accesses feature
  Output: Pro sees full content; Free sees blur/lock + upgrade CTA

STANDARD:
  Gate response: instant (local check, no API)
  Never: Free user sees Pro content ungated
  Never: Pro user sees Free restrictions
  Blur: consistent visual treatment across all gated features

MONITOR:
  - Gate bypass attempts (analytics event)
  - Conversion rate: Free tap upgrade CTA

TEST:
  - Unit: isPro/isFree returns correct value
  - Widget: each gated screen shows both variants
  - Integration: simulate tier change → all gates update

IMPACT IF BROKEN:
  🔴 Revenue loss (Free sees Pro) or 🔴 UX broken (Pro sees blur)
```

#### F04-F07: Interactive Features (Search, Stock Detail, Alert, Journal)
```
CONTRACT:
  Input:  User interacts with feature
  Output: Correct response, data persisted if needed

STANDARD:
  Response: ≤ 1s for local, ≤ 3s for network
  Data integrity: input saved = input displayed later
  Validation: clear error messages for invalid input

MONITOR:
  - Feature usage frequency (analytics)
  - Error rate per feature

TEST:
  - Happy path: create → read → update → delete
  - Edge: empty input, max length, special characters
  - Offline: queue action → sync when online
```

#### F08: Push Notification → Deep Link
```
CONTRACT:
  Input:  Backend sends push → User taps notification
  Output: App opens at exact screen referenced in notification

STANDARD:
  Foreground: in-app banner + tap → navigate
  Background: system notification + tap → app opens at correct screen
  Killed: system notification + tap → cold start + navigate

MONITOR:
  - Push delivery rate
  - Deep link resolve success rate
  - Notification open rate

TEST:
  - Foreground: receive notification → verify banner
  - Background: tap notification → verify correct screen
  - Edge: notification for deleted content → error screen (not crash)

IMPACT IF BROKEN:
  🔴 User gets lost after tapping notification → frustration + lower engagement
```

---

## 🔴 SURFACE 2: PHÍA SAU (App ↔ Backend)

### Tổng quan
| ID | Tương tác | Direction | Criticality |
|----|-----------|-----------|-------------|
| B01 | REST API — Stock Data | App→Server→App | 🔴 Critical |
| B02 | REST API — User Auth | Bidirectional | 🔴 Critical |
| B03 | REST API — CRUD Operations | Bidirectional | 🟡 High |
| B04 | WebSocket — Realtime Prices | Server→App (stream) | 🔴 Critical |
| B05 | WebSocket — Notifications | Server→App (push) | 🟡 High |
| B06 | Data Sync — Offline/Online | App↔Local↔Server | 🟢 Medium |

### Contract Details

#### B01: REST API — Stock Data
```
CONTRACT:
  Endpoints: GET /stock/{code}, GET /stock/list, GET /stock/{code}/history
  Request: Authorization Bearer token, Accept: application/json
  Response: JSON with fields: code, name, price, change, changePercent, volume, ...
  Status codes: 200 OK, 401 Unauthorized, 404 Not Found, 429 Rate Limited, 500 Server Error

STANDARD:
  Latency: p50 ≤ 500ms, p99 ≤ 2000ms
  Availability: 99.5% uptime (market hours), 99% off-hours
  Data freshness: ≤ 15s during market hours
  Rate limit: 100 req/min per user

MONITOR:
  - API response time (per endpoint)
  - Error rate (4xx, 5xx)
  - Data freshness gap

TEST:
  - Unit: model.fromJson handles all field types
  - Unit: model.fromJson handles null/missing fields → defaults
  - Integration: real API returns expected format
  - Edge: 429 rate limited → backoff + retry
  - Edge: 500 server error → user sees error state (not crash)

MOCK → REAL TRANSITION:
  Mock data structure MUST exactly match API response.
  Fields in mock_data.dart:
    ✅ Must match: field names, data types, nesting
    ⚠️ Watch: optional fields may be absent in real API
    🔴 Risk: mock has field X, real API doesn't → crash on access

IMPACT IF BROKEN:
  🔴 App shows stale/wrong stock prices → user makes bad trades
```

#### B02: REST API — User Auth
```
CONTRACT:
  Endpoints: POST /auth/login, POST /auth/register, POST /auth/refresh, POST /auth/logout
  Token: JWT (access token + refresh token)
  Flow: login → access token (15min) → refresh → new access token

STANDARD:
  Login response: ≤ 2s
  Token refresh: transparent to user (no visible loading)
  Failed refresh: redirect to login (not crash)
  Secure storage: tokens in FlutterSecureStorage (not SharedPreferences)

MONITOR:
  - Login success/failure rate
  - Token refresh rate
  - Unauthorized (401) frequency

TEST:
  - Unit: token decode, expiry check
  - Integration: login → get token → access protected endpoint
  - Edge: expired token → auto refresh → retry original request
  - Edge: refresh fails → logout → login screen
  - Security: token not in logs, not in URL params

IMPACT IF BROKEN:
  🔴 User locked out of app / 🔴 Security breach if token leaked
```

#### B04: WebSocket — Realtime Prices
```
CONTRACT:
  Connection: wss://[server]/socket.io/?transport=websocket
  Events IN: stock_price_update, market_status, system_alert
  Events OUT: subscribe_stock, unsubscribe_stock
  Payload: JSON { code: string, price: number, change: number, timestamp: number }

STANDARD:
  Connection: establish ≤ 3s
  Reconnect: auto-reconnect with exponential backoff (1s, 2s, 4s, 8s, max 30s)
  Data latency: ≤ 1s from server emit to UI update
  Max subscriptions: 50 stocks simultaneously

MONITOR:
  - Socket connection uptime
  - Reconnect frequency
  - Message delivery latency

TEST:
  - Unit: parse socket event payload
  - Integration: connect → subscribe → receive update → UI updates
  - Edge: disconnect → reconnect → resume subscription
  - Edge: server sends malformed data → log error (not crash)

IMPACT IF BROKEN:
  🔴 Prices freeze on screen → user thinks market stopped → bad decisions
```

---

## 🟡 SURFACE 3: PHƯƠNG NGANG (App ↔ Dịch vụ bên ngoài)

### Tổng quan
| ID | Service | Direction | Criticality | Side |
|----|---------|-----------|-------------|------|
| L01 | TradingView Chart | App→TV Widget | 🔴 Critical | Left |
| L02 | Firebase Cloud Messaging | Firebase→App | 🟡 High | Left |
| L03 | Social Login (Google/Apple) | User→Provider→App | 🟡 High | Left |
| L04 | Payment Gateway | User→Payment→Server | 🟡 High | Left |
| L05 | In-App Purchase (iOS/Android) | User→Store→App | 🟡 High | Left |
| R01 | Analytics (Mixpanel/Amplitude) | App→Service | 🟢 Medium | Right |
| R02 | Crash Reporting (Sentry) | App→Service | 🟡 High | Right |
| R03 | Remote Config / Feature Flags | Service→App | 🟢 Medium | Right |
| R04 | CDN / Asset Delivery | CDN→App | 🟢 Medium | Right |

### Contract Details

#### L01: TradingView Chart
```
CONTRACT:
  Integration: WebView loading TradingView Charting Library
  Input: symbol, interval, indicators, theme (dark/light)
  Output: Interactive chart with crosshair, zoom, scroll
  License: required TradingView license key

STANDARD:
  Chart load: ≤ 5s (includes WebView init)
  Interaction: smooth 60fps pan/zoom
  Data: matches stock data from API (same source)
  Theme: matches app theme (dark/light toggle)

MONITOR:
  - Chart load time
  - Chart load failure rate
  - WebView crash rate

TEST:
  - Widget: chart WebView renders without crash
  - Integration: chart shows correct stock data
  - Edge: no internet → chart shows error message (not blank)
  - Edge: TradingView CDN down → fallback message

BREAKING CHANGE RISK:
  TradingView updates library version → API changes → chart breaks
  Mitigation: pin library version, test before upgrading

IMPACT IF BROKEN:
  🔴 Technical analysis useless → Pro users leave

REFERENCE: docs/05_TRADINGVIEW_INTEGRATION.md
```

#### L02: Firebase Cloud Messaging
```
CONTRACT:
  Platform: Firebase SDK for Flutter
  iOS: APNs → FCM → App
  Android: FCM → App
  Token: device token registered on login, updated on token refresh
  Payload: { title, body, data: { type, target_screen, params } }

STANDARD:
  Delivery: ≤ 30s from server send to device receive
  Token registration: on every app launch (if changed)
  Background handling: notification shown without waking app

MONITOR:
  - Push delivery success rate
  - Token registration failures
  - Notification open rate

BREAKING CHANGE RISK:
  Firebase deprecates API version (FCM v1 → v2)
  Apple changes APNs requirements yearly (WWDC)

IMPACT IF BROKEN:
  🟡 User misses important alerts → reduced engagement
```

#### L03-L05: Auth & Payment Services
```
CONTRACT:
  Social Login: OAuth 2.0 flow → callback → exchange code → user profile
  Payment: Initiate → 3rd party handles → webhook confirms → app updates tier
  IAP: StoreKit (iOS) / Google Play Billing → purchase verified → tier updated

STANDARD:
  Auth: ≤ 5s total flow (including provider)
  Payment: ≤ 30s total flow
  IAP restore: handles previously purchased subscriptions

BREAKING CHANGE RISK:
  Apple requires Sign in with Apple if Google Sign-In offered
  Google Play Billing Library major version changes annually

IMPACT IF BROKEN:
  🔴 Users can't sign in → complete block
  🔴 Payment fails → revenue loss
```

---

## 🟢 SURFACE 4: BÊN TRONG (Component ↔ Component)

### Tổng quan
| ID | Subsystem | Depends On | Consumed By | Criticality |
|----|-----------|-----------|-------------|-------------|
| I01 | Theme System | kfsp_colors → theme_colors → theme → extensions | 45+ widget files | 🔴 Critical |
| I02 | Router | app_router, bottom_nav_shell | All screens, deep links | 🔴 Critical |
| I03 | State (Riverpod) | Providers, Notifiers | All feature screens | 🔴 Critical |
| I04 | Network Layer | DioClient, interceptors | All API calls | 🔴 Critical |
| I05 | Shared Widgets | CustomComponent/* | All features | 🟡 High |
| I06 | Mock Data | mock_data.dart | All kDebugMode screens | 🟡 High |
| I07 | Storage | SecureStorage, SharedPrefs | Auth, Settings | 🟡 High |
| I08 | Utils & Extensions | context_extensions, formatters | Everywhere | 🟡 High |

### Contract Details

#### I01: Theme System (CASCADE CHAIN)
```
FILE CHAIN:
  kfsp_colors.dart          → raw Color values (source of truth)
  └→ kfsp_theme_colors.dart → ThemeExtension wrapper
     └→ kfsp_theme.dart     → MaterialTheme with extension registered
        └→ context_extensions.dart → shortcuts (context.kfspColors)
           └→ 45+ widget files → context.kfspColors.primary etc.

CONTRACT:
  Every color used in UI MUST come from context.kfspColors.*
  Every spacing MUST come from context.kfspSpacing.*
  Never: hardcoded Color(0xFF...) in feature widgets

STANDARD:
  kfsp_colors.dart values match docs/03_DESIGN_SYSTEM.md
  All colors in kfsp_colors are exposed in kfsp_theme_colors
  context_extensions provides shortcut for every token

CHANGE IMPACT:
  Change color value in kfsp_colors.dart:
    → Automatically updates 45+ files ✅ (if all use tokens)
    → Only breaks files with hardcoded values

  Remove color from kfsp_colors.dart:
    → kfsp_theme_colors compile error
    → context_extensions compile error
    → All consumer files compile error
    → 🔴 FULL CASCADE BREAK

  Add new color:
    → Must add to: kfsp_colors → kfsp_theme_colors → kfsp_theme → context_extensions
    → Forget any step → color unavailable to consumers

TEST:
  - Unit: every color in kfsp_colors exists in kfsp_theme_colors
  - Unit: context_extensions exposes all theme_colors
  - Grep: no hardcoded Color() in lib/features/
```

#### I02: Router
```
FILE CHAIN:
  app_router.dart           → route definitions + guards
  └→ bottom_nav_shell.dart  → 5 bottom tab branches
     └→ Screen imports      → actual screen widgets
     └→ Deep link handlers  → external URL → internal route

CONTRACT:
  Every screen MUST have a route in app_router.dart
  Every route MUST point to existing screen file
  Route names MUST be unique
  Route params MUST match screen constructor

CHANGE IMPACT:
  Rename route path:
    → Deep links using old path break
    → Other screens with context.go('/old-path') break
    → Push notification payloads with old path break

  Remove screen:
    → Route pointing to it → compile error ✅
    → Other screens navigating to it → runtime error ❌

TEST:
  - Unit: all routes resolve to existing screens
  - Integration: navigate to every route → no crash
  - Edge: invalid route → error screen
```

#### I03: State Management (Riverpod)
```
PROVIDER GRAPH:
  envConfigProvider (root)
  └→ dioClientProvider → API calls
  └→ secureStorageProvider → token storage

  preferencesProvider (root)
  └→ themeModeProvider → app theme

  appRouterProvider (root) → navigation

  Feature providers:
  └→ dashboardProvider → Dashboard screen
  └→ priceboardProvider → PriceBoard screen
  └→ watchlistProvider → Watchlist screen
  └→ stockDetailProvider → Stock Detail screen

CONTRACT:
  Providers expose AsyncValue (loading/data/error states)
  Consumers MUST handle all 3 states
  Provider disposal: auto-dispose when screen pops
  Never: provider stores user-specific data after logout

CHANGE IMPACT:
  Change provider return type:
    → All consumers using ref.watch() → type error
    → 🔴 Multiple screens break

  Change provider dependencies:
    → Circular dependency → runtime crash

TEST:
  - Unit: provider returns expected mock data
  - Unit: provider handles error state
  - Widget: screen renders for loading/data/error
```

#### I06: Real Data Integration + Offline Fallback (REAL DATA MODE — 2026-03-13+)
```
CONTRACT:
  REAL DATA is primary source: Socket.IO events + REST API
  Mock data is OFFLINE FALLBACK ONLY — shown when API/socket unavailable
  Pattern:
    final data = await api.getStockList();  // Real API first
    if (data.isEmpty) return MockData.stockList;  // Fallback only

  Fallback mock fields MUST match real API response fields

SOCKET PROTOCOL (BẮT BUỘC):
  - emit('join', [stockCodes]) BEFORE on('updateliveindex', handler)
  - emit('leave', [stockCodes]) on dispose
  - emitWithAck args: wrap list in [] → emitWithAck('event', [listArg], cb)
  - updateliveindex fields: price/vol/totalval (NOT lastprice/lastvol/totalvalue)

INTEGRATION CHECKLIST:
  For each feature with real data:
  - [ ] API endpoint/socket event documented in docs/10
  - [ ] Model class handles all API fields
  - [ ] Model.fromJson handles null/missing
  - [ ] Error handling: API fail → show error state or offline fallback (not crash)
  - [ ] Loading state shown during API call
  - [ ] Offline fallback mock data available
  - [ ] Socket join/leave lifecycle correct

CHANGE IMPACT:
  Real API response structure changes:
    → Runtime crash on field access
    → 🔴 Field name mismatch between socket events (verified gotcha!)
    → Socket event field names differ from initial fetch response fields
```

---

## Dependency Matrix: Surface × Surface

Khi thay đổi ở một surface, surface nào khác bị ảnh hưởng?

| Changed ↓ Affected → | 🔵 Front | 🔴 Back | 🟡 Lateral | 🟢 Internal |
|----------------------|----------|---------|-----------|-------------|
| 🔵 **Front** (UI change) | — | 🟢 Low | 🟢 Low | 🟡 Med (shared widgets) |
| 🔴 **Back** (API change) | 🔴 High (data display) | — | 🟢 Low | 🔴 High (models, providers) |
| 🟡 **Lateral** (3rd party update) | 🟡 Med (chart, push) | 🟢 Low | — | 🟡 Med (SDK wrappers) |
| 🟢 **Internal** (refactor) | 🔴 High (UI renders) | 🟡 Med (API calls) | 🟡 Med (service calls) | — |

**Key insight:** Backend API changes and Internal refactors have the highest cross-surface impact.

---

## Monitoring Dashboard Specification

### Real-time Metrics (khi có production app)

| Metric | Source | Threshold | Alert |
|--------|--------|-----------|-------|
| App crash rate | Sentry/Crashlytics | < 1% | 🔴 if > 1% |
| API error rate | Server logs | < 0.5% | 🔴 if > 1% |
| API p99 latency | Server monitoring | < 2s | 🟡 if > 3s |
| Socket uptime | Socket.IO stats | > 99% | 🔴 if < 95% |
| Push delivery rate | FCM dashboard | > 95% | 🟡 if < 90% |
| Chart load success | Analytics | > 98% | 🟡 if < 95% |
| Login success rate | Analytics | > 99% | 🔴 if < 95% |
| Screen load time | Analytics | < 2s p90 | 🟡 if > 3s |

### Development Metrics (kiểm tra bằng skills)

| Metric | Skill | Frequency |
|--------|-------|-----------|
| Design token compliance | `/kfsp:ux-parity` | Mỗi phase |
| Code regression | `/kfsp:sentinel` | Trước/sau thay đổi lớn |
| Surface contract health | `/kfsp:surface-test` | Mỗi release |
| Overall codebase health | `/kfsp:guard --deep` | Hàng tuần |
| Pre-release readiness | `/kfsp:release-gate` | Trước release |

---

## Checklist: Khi thay đổi ở mỗi Surface

### ☑️ Thay đổi PHÍA TRƯỚC (UI/UX)
- [ ] Navigation vẫn hoạt động? (`/kfsp:sweep` hoặc check routes)
- [ ] Pro/Free gate đúng?
- [ ] Error/empty/loading states có?
- [ ] Design token compliance? (`/kfsp:ux-parity`)
- [ ] Accessibility (text size, contrast)?

### ☑️ Thay đổi PHÍA SAU (API/Data)
- [ ] Model.fromJson handle null fields?
- [ ] Mock data updated to match? (I06 contract)
- [ ] Error handling cho API failure?
- [ ] Token/auth flow intact?
- [ ] docs/10 updated?

### ☑️ Thay đổi PHƯƠNG NGANG (3rd party)
- [ ] SDK version compatible?
- [ ] Fallback nếu service down?
- [ ] Config/keys correct?
- [ ] docs/02 updated (tech stack)?

### ☑️ Thay đổi BÊN TRONG (Refactor)
- [ ] Theme cascade intact? (I01)
- [ ] All routes still resolve? (I02)
- [ ] Providers graph OK? No circular deps? (I03)
- [ ] Import paths updated?
- [ ] Tests updated for changed interfaces?
- [ ] `/kfsp:sentinel --compare` clean?

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-03-11 | 1.0 | Initial version — 4 surfaces, 36 interactions, contracts, standards, monitoring specs |
