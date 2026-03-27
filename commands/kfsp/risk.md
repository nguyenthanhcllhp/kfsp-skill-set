---
name: kfsp:risk
description: "Risk analysis — pre-mortem before building, contract verification on 4 surfaces, regression detection. Replaces pre-mortem + surface-test + sentinel."
---

# /kfsp:risk — Phân tích rủi ro

## Mục đích
Dự đoán rủi ro trước khi code, kiểm tra contracts 4 bề mặt, phát hiện regression.

**Gộp từ:** pre-mortem + surface-test + sentinel

## Modes

### --before
Pre-mortem: dự đoán rủi ro trước khi bắt đầu feature/phase.

**4 bề mặt phân tích:**

| Bề mặt | Câu hỏi | Ví dụ risk |
|--------|---------|-----------|
| User (UI/UX) | User có thể bối rối? | Bottom sheet che nội dung quan trọng |
| Backend (API) | API có handle được? | FCM token API chưa có |
| Services (3rd party) | Dependency nào có thể fail? | Firebase quota limit |
| Internal (Code) | Code nào dễ vỡ? | SharedPreferences vs Hive migration |

**Output:**
```
🔮 Pre-mortem — Phase 4.11 Notification

HIGH RISK:
├── Backend: FCM API chưa có → push notification blocked
├── Internal: Hive migration nếu đổi từ SharedPref
└── User: Quá nhiều toggles → user overwhelm

MEDIUM RISK:
├── Services: Firebase messaging quota
└── Internal: Bottom sheet conflict với in-app banner

MITIGATIONS:
├── Làm in-app trước, push notification sau khi backend sẵn
├── Max 5-6 toggles, nhóm theo category
└── Dismiss banner khi bottom sheet mở
```

### --contracts
Kiểm tra interaction contracts trên 4 bề mặt. Chạy trước release.

**Checks:**
1. **Front (User):**
   - Tất cả routes hoạt động
   - Pro/Free gates đúng
   - Loading/error/empty states
   - Deep link handling

2. **Back (API):**
   - API endpoints reachable
   - Request/response format đúng
   - Error handling (401, 403, 500)
   - Socket join/leave protocol

3. **Lateral (Services):**
   - Firebase config valid
   - ECharts WebView load
   - TradingView integration
   - Push notification permissions

4. **Internal (Code):**
   - Provider dependencies resolved
   - Router config complete
   - Theme/dark mode consistent
   - Lifecycle management

### --regression
So sánh trước/sau thay đổi lớn. Phát hiện regression.

**Baseline checks:**
1. File structure (count, new/deleted files)
2. Route count (trước vs sau)
3. Provider count
4. API surface (endpoints)
5. Test pass rate
6. Build size
7. Startup time (nếu đo được)

**Output:**
```
📊 Regression Check — Phase 4.11

                Before    After     Delta
Routes:         38        40        +2 ✅
Providers:      11        12        +1 ✅
Dart files:     208       211       +3 ✅
Test pass:      183/183   190/190   +7 ✅
Build size:     101MB     103MB     +2MB ⚠️
Hardcoded:      260       258       -2 ✅

NO REGRESSION DETECTED ✅
```
