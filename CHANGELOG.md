# Changelog — KFSP Skill Set

## v4.0 (2026-03-27) — Major Refactor

### Gộp 17 → 7 skills
Giảm 53% số lệnh, giữ đủ 5 tầng, loại bỏ trùng lặp.

| Skill mới | Gộp từ |
|-----------|--------|
| `/kfsp:start` | session-start |
| `/kfsp:check` | guard + sweep + sync-check + ux-parity (convention) |
| `/kfsp:test` | build-verify + test-brief + bug-log |
| `/kfsp:docs` | doc-pilot + post-phase |
| `/kfsp:gate` | pre-commit + release-gate |
| `/kfsp:log` | dev-journal + incident-review |
| `/kfsp:risk` | pre-mortem + surface-test + sentinel |

### Bỏ hẳn
- `product-health` — thông tin đã có trong `check --full`
- `help` — vẫn giữ file nhưng nội dung gọn, liệt kê 7 skills

### Tính năng mới

#### Sync System (Rule 21)
- Desktop ghi `.changes.json` sau mỗi checkpoint
- Terminal đọc → cập nhật docs/specs/testcases → đánh dấu processed
- Cuối phiên verify 0 unprocessed entries
- Đảm bảo docs luôn đồng bộ dù có nhiều thay đổi trong phiên

#### Dual-Session (Rule 20)
- 2 phiên Claude Code song song: Desktop (code) + Terminal (docs)
- Phân vùng file: Desktop `lib/`, Terminal `docs/`
- CẤM 2 phiên sửa cùng 1 file

#### Live Guide
- `KFSP_Live_Guide.html` — dashboard phiên làm việc
- Tự động mở khi chạy `/kfsp:start`
- Track bước hiện tại, checklist, gợi ý lệnh tiếp theo
- Timer phiên làm việc
- Lưu state vào localStorage

#### Response Format mở rộng (Rule 8)
- Thêm section `🔄 Sync → Terminal` trong mỗi response có code change
- Ghi impact map: docs nào bị ảnh hưởng, Terminal cần chạy gì

### Tài liệu HTML mới
- `KFSP_Skill_v4_Workflow.html` — flow, merge map, lifecycle, dual-session
- `KFSP_Sync_System_v4.html` — thiết kế sync system (6 tabs)
- `KFSP_Live_Guide.html` — live guide phiên làm việc

### Archive
- 19 skill files v3 → `agents_archived_v3/`

---

## v3.3 (2026-03-20)

- **ARCHITECTURE.md** — Holistic skill architecture (English, GitHub-facing)
- **OPERATIONS_GUIDE.md** — Internal operations guide (Vietnamese, 16 sections)
- **Interactive diagram** — HTML onboarding with flow drilldown + case study
- **Spec Compliance Check** — build-verify CHECK 17
- **Test from Spec** — test-brief Step 0
- **Verification Flow** — 2-stream Spec→Build→Verify

## v3.2 (2026-03-15)

- **ORCHESTRATION.md** — GSD + Ralph integration
- **Phase-as-Project Rule** — 6 chiều bắt buộc
- **Dev Journal Compliance** — 8-layer enforcement
- **Number Formatting Rule** — Financial app conventions

## v3.0 (2026-03-13)

- +4 skills: test-brief, bug-log, build-verify, session-start
- Feature Completeness Rule (6 dimensions)
- Vietnamese Diacritics Rule
- Data Accuracy Rule (zero tolerance)
- Simulator Testing Rule
- Git Remote Safety Rule

## v2.0 (2026-03-08)

- 13 core skills across 5 tiers
- Tier-based organization (T1-T5)
- Auto-trigger system
- GSD integration

## v1.0 (2026-03-01)

- Initial release: 8 skills
- Basic guard, sweep, pre-commit
