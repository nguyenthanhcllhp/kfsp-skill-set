---
name: kfsp:check
description: "Codebase health check — health score, blast radius, token audit, source sync, convention. Replaces guard + sweep + sync-check + ux-parity."
---

# /kfsp:check — Rà soát sức khoẻ

## Mục đích
Đánh giá toàn diện codebase: health score, blast radius sau thay đổi, design token audit, source sync, convention check.

**Gộp từ:** guard + sweep + sync-check + ux-parity (convention part)

## Modes

### --quick (mặc định)
Chạy sau mỗi lần code. Nhanh, tập trung vào thay đổi gần nhất.

**Checks:**
1. **Health score** — file count, test ratio, compile status
2. **Token audit** — NEW hardcoded colors/spacing/fontSize so với lần check trước
3. **Convention** — tiếng Việt có dấu, number format, design system compliance
4. **Blast radius** — files nào bị ảnh hưởng bởi thay đổi gần nhất

**Output:** Score /100 + danh sách issues

### --full
Chạy cuối phiên hoặc cuối phase. Đầy đủ 10 chiều.

**10 chiều đánh giá:**
1. Cấu trúc file (file count, test ratio, orphans)
2. Design system (hardcoded colors, spacing, fontSize, borderRadius)
3. Cross-cutting (providers, routes, models sync)
4. Spec alignment (code vs docs/plans)
5. Safety (git remote, secrets, .env)
6. Feature completeness (6 chiều per feature)
7. Vietnamese diacritics (UI text có dấu)
8. Unit test coverage
9. Source sync (3 copies: working, build, package)
10. **.changes.json** — verify 0 unprocessed entries (Rule 21)

**Output:** Score /100 + dashboard 10 chiều + recommendations

### --tokens
Chi tiết hardcoded values:

```
Hardcoded Colors:   260 occurrences in 45 files
Hardcoded fontSize: 102 occurrences in 28 files
Hardcoded radius:    47 occurrences in 19 files
NEW since last check: 3 colors in notification_settings_screen.dart
```

### --sync
Source copy sync check:

| Location | Files | Last modified | Status |
|----------|-------|--------------|--------|
| current_source/ | 208 | 2026-03-27 | ✅ Primary |
| ~/Desktop/kfsp_flutter_dev/ | 205 | 2026-03-26 | ⚠️ 3 files behind |

## Ghi .changes.json
Nếu phát hiện issues → ghi entry type "check" vào .changes.json với impact map.
