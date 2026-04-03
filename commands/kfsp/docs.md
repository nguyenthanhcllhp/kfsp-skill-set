---
name: kfsp:docs
description: "Documentation management — status check, gap analysis, update docs from .changes.json, close phase. Replaces doc-pilot + post-phase."
---

# /kfsp:docs — Tài liệu

## Mục đích
Quản lý tài liệu: kiểm tra status, phát hiện gap, cập nhật theo .changes.json, đóng phase.

**Gộp từ:** doc-pilot + post-phase

## Modes

### --status
Kiểm tra trạng thái tất cả docs. Chạy ở Terminal.

**Kiểm tra:**
1. Đọc `docs/logs/.changes.json` → liệt kê entries chưa processed
2. So sánh timestamps: code files vs docs files
3. Kiểm tra docs registry:

| Doc | Last updated | Code changed since | Status |
|-----|-------------|-------------------|--------|
| 01_PROJECT.md | 03-26 | 03-27 | ⚠️ Cần cập nhật |
| 04_FEATURES_v2.md | 03-27 | 03-27 | ✅ OK |
| test_registry_v4.html | 03-26 | 03-27 | ⚠️ Thiếu TC |

**Output:** Bảng status + danh sách actions cần làm

### --update
Cập nhật docs + changelog theo .changes.json. **Đây là mode chính cho Terminal.**

**Quy trình:**
1. Đọc `.changes.json` → lấy entries có `processed: false`
2. Với mỗi entry, đọc impact map:
   - `spec` → cập nhật file spec tương ứng
   - `testcase` → thêm/sửa test cases trong registry
   - `architecture` → cập nhật 02_ARCH.md
   - `plan` → cập nhật plan file status
3. **Cập nhật changelog** (`changelog_thanh_sg.md`):
   - Tìm entry `[Unreleased]` có trạng thái 🔨
   - Thêm thay đổi vào đúng section (Added/Changed/Fixed) theo format trong file
   - Mô tả CỤ THỂ từng file: tên file, MỚI hay SỬA, số dòng, thay đổi gì
   - Nhóm theo feature, KHÔNG liệt kê lộn xộn
   - Nếu chưa có entry Unreleased → tạo mới với base = commit hiện tại
4. Sau khi xử lý → đánh dấu `processed: true`
5. Ghi kết quả vào session HTML

**Output:**
```
📝 Docs Updated:
├── 04_FEATURES_v2.md — +1 screen spec (NotificationSettings)
├── 02_ARCH.md — +1 route, +1 provider
├── test_registry_v4.html — +5 TC
├── P4.11 plan — task 4.11.11 → done
├── changelog_thanh_sg.md — +2 entries (Added: NotificationSettings, Changed: app_router)
└── .changes.json: 3/3 entries processed ✅
```

### --close-phase
Đóng phase: cập nhật tất cả docs, archive artifacts.

**Checklist đóng phase:**
1. Verify 0 unprocessed entries trong .changes.json
2. Cập nhật plan file → status DONE
3. Cập nhật 01_PROJECT.md → phase progress
4. Archive .changes.json → `docs/logs/changes/`
5. Tạo session summary trong session HTML
6. Kiểm tra 6/6 chiều feature (Rule 9)

## Chuẩn viết doc khi --update

### Spec (04_FEATURES_v2.md)
Khi code thay đổi feature → cập nhật spec theo cấu trúc 11 sections unified:
1. Mô tả + User Stories, 2. UI Spec, 3. Data & API, **4. Business Rules**, **5. Acceptance Criteria**, **6. Error & Edge Cases**, **7. Input & Boundary**, **8. Performance**, **9. Pro/Free Gating**, 10. Files, 11. Test Case IDs.

**Quy tắc:**
- Mỗi rule/criteria có ID: `BR-{FT}-{NN}`, `AC-{FT}-{NN}`, `ERR-{FT}-{NN}`...
- Business Rules PHẢI có giá trị cụ thể (ZERO TOLERANCE cho finance)
- Acceptance Criteria PHẢI rõ pass/fail (tester đọc hiểu)
- Sections 4-9 là phần QA/Tester cần — KHÔNG ĐƯỢC bỏ qua

### Test Case (master_test_registry)
Khi thêm/sửa test case → tuân thủ AI-RBT (xem `05_TESTING.md` section "Chuẩn viết Test Case"):
- **4 loại BẮT BUỘC** per feature: Positive, Negative, Boundary, Edge
- Mỗi test case có: `riskLevel` (High/Medium/Low), `testType`, `specRef` (link spec ID)
- Test data convention: `KFSP_{testName}_{timestamp}`
- Expected result CỤ THỂ — KHÔNG viết "hoạt động đúng"

### Changelog (changelog_thanh_sg.md)
- Mô tả CỤ THỂ từng file: tên, MỚI/SỬA, số dòng, thay đổi gì
- Nhóm theo feature
- Dùng merge-base khi so sánh branch

## Doc Registry

| File | Khi nào cập nhật |
|------|-----------------|
| 01_PROJECT.md | Phase status change |
| 02_ARCHITECTURE.md | Thêm/đổi route, provider, API, dependency |
| 03_DESIGN_SYSTEM.md | Thêm/đổi token, color, spacing |
| 04_FEATURES_v2.md | Thêm/đổi feature spec (unified 11 sections) |
| 05_TESTING.md | Test count change, coverage, standards |
| 06_DECISIONS_LOG.md | Decision/incident |
| plans/P{N}.md | Task status change |
| test_cases/master_test_registry_v4.html | Thêm/sửa test cases (4 loại, risk level, specRef) |
| changelog_thanh_sg.md | Mỗi code change — Added/Changed/Fixed, mô tả cụ thể từng file |
