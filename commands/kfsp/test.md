---
name: kfsp:test
description: "Testing pipeline — verify build, generate PM test brief, log bugs from PM feedback. Replaces build-verify + test-brief + bug-log."
---

# /kfsp:test — Kiểm thử

## Mục đích
Pipeline test: verify build → tạo brief cho PM → log bug từ PM feedback.

**Gộp từ:** build-verify + test-brief + bug-log

## Modes

### --verify
Chạy sau build. Gate trước khi gửi PM.

**Checks:**
1. `flutter analyze` — 0 errors, warnings < 10
2. `flutter test` — all pass
3. Import validation — no circular, no unused
4. Design token compliance — no NEW violations
5. Dark mode basic check
6. Vietnamese text — UI strings có dấu
7. Runtime sanity — app khởi động được

**Output:** ✅ PASS / ❌ FAIL + chi tiết

### --brief
Chạy sau --verify PASS. Tạo checklist cho PM Thanh.

**Tự động tạo từ:**
- .changes.json entries trong phiên (biết chính xác thay đổi gì)
- Plan file (biết task nào đang làm)
- Test registry (biết TC nào liên quan)

**Format output:**
```
📋 KFSP Test Brief — Phase 4.11 — 2026-03-27
Build: KFSP_v3.1.0_2026-03-27.apk

CẦN TEST:
1. ☐ Notification Settings — bật/tắt 5 toggles
2. ☐ Bottom sheet tin mới — tap banner → mở sheet
3. ☐ "Không làm phiền" — toggle + verify im lặng
4. ☐ Regression: Dashboard ticker vẫn hoạt động
5. ☐ Regression: In-app banner animation

CÁCH TEST:
- Mở app → Settings → Thông báo
- Toggle từng loại, verify lưu + hoạt động
- ...

BUGS ĐÃ BIẾT:
- Chưa có push notification (Phase 4.11B — chờ backend)
```

### --bug
Ghi nhận bug từ PM feedback.

**Input:** Thanh mô tả bug
**Output:** Structured bug entry

```
BUG-043 | P2 | notification_settings_screen
Mô tả: Toggle "Không làm phiền" không lưu khi thoát app
Tái hiện: Bật toggle → thoát app → mở lại → toggle reset
Severity: major
Files: lib/screens/notification_settings_screen.dart
```

Ghi vào: `docs/logs/bugs/` + session HTML

### --bugs
Liệt kê tất cả bugs chưa fix trong phiên.
