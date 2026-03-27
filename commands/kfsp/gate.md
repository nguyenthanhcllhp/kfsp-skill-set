---
name: kfsp:gate
description: "Quality gates — pre-commit (23 checks) or pre-release (12 gates). MUST pass before proceeding. Replaces pre-commit + release-gate."
---

# /kfsp:gate — Cổng chặn

## Mục đích
Chặn trước commit hoặc trước release. PHẢI pass mới được tiếp.

**Gộp từ:** pre-commit + release-gate

## Modes

### --commit
23 checks trước mỗi commit. BẮT BUỘC (Rule 21).

**5 tầng kiểm tra:**

#### T1 Safety (BLOCK nếu fail)
1. Dev journal — có entry cho thay đổi này?
2. Health score ≥ 70 (từ /kfsp:check)
3. Git remote — KHÔNG tự ý push (Rule 14)
4. .changes.json — entry đã ghi cho thay đổi này?

#### T2 Code (BLOCK nếu fail)
5. Design tokens — 0 NEW hardcoded violations
6. Secrets scan — no .env, no API keys in code
7. Import validation — no circular, no unused
8. Naming conventions — Dart conventions
9. Sweep summary — blast radius acceptable
10. Sync status — source copies aware

#### T3 QA (WARN, không block)
11. Unit tests — all pass
12. Test ratio ≥ 5%
13. Build verify — flutter analyze clean

#### T4 UX (WARN)
14. Vietnamese diacritics — UI text có dấu (Rule 10)
15. Number format — dấu . thập phân, , phần nghìn (Rule 11)
16. Convention check — colors/spacing/typography theo design system

#### T5 Business (WARN)
17. Feature 6 chiều (Rule 9) — nếu hoàn thành feature
18. Build deliverable (Rule 18) — nếu sau build
19. Plan file cập nhật (Rule 19)
20. Changelog entry
21. Docs impact noted
22. Test cases covered
23. Legacy trend — hardcoded count trending down?

**Output:** ✅ PASS / ❌ BLOCK / ⚠️ WARN(N)

### --release
12 gates trước release. Score /100.

**12 gates:**
1. **Build health** — analyze clean, tests pass, no crashes
2. **Security** — no secrets, no debug flags, ProGuard/obfuscation
3. **Design tokens** — hardcoded count ≤ threshold
4. **UX quality** — error handling, loading states, empty states
5. **Accessibility** — font sizes, contrast, touch targets
6. **Test coverage** — test ratio, critical paths covered
7. **Dependencies** — no deprecated, no security vulnerabilities
8. **Docs sync** — all docs up-to-date, .changes.json 0 unprocessed
9. **Store compliance** — bundle ID, version, icons, screenshots
10. **Performance** — app size, startup time, memory
11. **Regression** — no existing features broken
12. **Feature completeness** — 6/6 chiều cho features mới

**Decision:**
- Score ≥ 80 → ✅ PASS
- Score 60-79 → ⚠️ CAUTION (list blockers)
- Score < 60 → ❌ FAIL
