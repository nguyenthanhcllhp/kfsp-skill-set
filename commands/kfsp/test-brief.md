---
name: kfsp:test-brief
description: Generate PM test brief after each build — structured checklist for Thanh with what to test, expected behavior, how to test, pass/fail criteria
argument-hint: "[build-report-path] [--from-changes]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
  - Write
---
<objective>
KFSP Test Brief — Tạo bản brief test CHO PM (Thanh) sau mỗi build session.

**Vấn đề giải quyết:** Thanh là người test chính. Agent build xong → Thanh không biết test gì, kỳ vọng gì.
Test Brief tự động tạo checklist dựa trên deliverables của build session.

**Triết lý:** PM test = Acceptance Test. Không cần biết code, chỉ cần biết:
1. Test GÌ (item cụ thể)
2. KỲ VỌNG gì (expected behavior)
3. Test THẾ NÀO (steps cụ thể trên device)
4. PASS/FAIL criteria

**Modes:**
- Default: Đọc build report gần nhất → tạo test brief
- `--from-changes`: Scan git diff → tạo test brief từ code changes
- `[path]`: Đọc build report cụ thể

**Khi nào dùng:**
- SAU MỖI BUILD SESSION — bắt buộc trước khi giao Thanh
- Là phần bắt buộc trong build report (section `🧪 Test Checklist cho Thanh`)
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- Có path → đọc build report đó
- `--from-changes` → scan recent git changes
- Rỗng → tìm build report gần nhất trong `docs/build_reports/`

## Step 1: Collect Deliverables

### From Build Report:
Read build report → extract:
1. **Deliverables table** (deliverables đã hoàn thành)
2. **Files changed** (biết scope thay đổi)
3. **Technical decisions** (biết behavior mới)
4. **Known bugs / tech debt** (biết limitation)

### From Git Changes (--from-changes):
```bash
# Recent changes
cd "$SOURCE"
git diff --name-only HEAD~5..HEAD -- lib/ 2>/dev/null || find lib/ -name "*.dart" -newer pubspec.yaml
```

## Step 2: Classify Test Items

Phân loại mỗi deliverable thành test categories:

### Category A: Visual / UI
- Màu sắc, layout, typography
- Dark mode + Light mode
- Responsive (different screen sizes)
- Animation / transition

### Category B: Navigation / Flow
- Tap → đúng screen
- Back button hoạt động
- Tab switching
- Deep link (nếu có)

### Category C: Data / Content
- Dữ liệu hiển thị đúng (real data hoặc mock)
- Format đúng (số VN, ngày, %)
- Empty state khi không có data
- Error state khi API fail

### Category D: Interaction
- Tap, swipe, scroll
- Search/filter
- Edit mode
- Drag & drop (nếu có)

### Category E: Performance / Edge Cases
- Load time
- Scroll smooth (no jank)
- Background/resume
- Offline behavior

## Step 3: Generate Test Brief — FILE HTML (BẮT BUỘC)

### Output Location
Write to: `Product/kfsp_flutter_migration/docs/build_reports/YYYY-MM-DD_test-brief.html`

### ⚠️ QUY TẮC OUTPUT — KHÔNG ĐƯỢC DÙNG MARKDOWN

> **BẮT BUỘC:** Test brief PHẢI là file `.html` tương tác.
> PM mở trực tiếp trên trình duyệt hoặc Claude preview để điền.
> KHÔNG tạo file `.md` cho test brief.
> Chi tiết quy tắc: `docs/15_TESTING_RULES.md`

### Yêu cầu file HTML

1. **Giao diện SÁNG** (light mode) — nền `#F6F6F8`, thẻ trắng
2. **100% tiếng Việt** — tiêu đề, nút, placeholder, hướng dẫn
3. **Tương tác:** bấm Đạt/Lỗi/Bỏ qua, gõ ghi chú, tick checkbox
4. **Hình + Video:** mỗi mục test + form bug có nút đính kèm hình/video
5. **Lưu tự động:** localStorage — đóng mở trình duyệt vẫn còn
6. **Sao chép kết quả:** nút export → copy Markdown dán vào chat
7. **Form báo lỗi:** tích hợp sẵn, tự tạo mã BUG-YYYYMMDD-NN

### Cấu trúc HTML bắt buộc

```
1. Tiêu đề + mô tả build + thời gian ước tính
2. Hướng dẫn ngắn
3. Thanh thống kê (Đạt / Lỗi / Bỏ qua / Hoàn thành)
4. Danh sách Test — mỗi mục:
   - Số + Nhóm + Tên + Ưu tiên (P0/P1/P2)
   - Kỳ vọng + Cách test
   - Nút: Đạt / Lỗi / Bỏ qua / Xóa
   - Ô ghi chú + nút đính kèm hình/video
5. Bảng chế độ tối — checkbox Sáng/Tối
6. Kiểm tra hồi quy — checkbox
7. Form báo lỗi — có trường đính kèm hình/video
8. Hạn chế đã biết
9. Thanh sticky: "Sao chép kết quả" + "Làm lại"
```

### Template tham khảo
Dùng file gần nhất trong `docs/build_reports/*_test-brief.html` làm template.
Sửa `tests[]`, `regressions[]`, `darkScreens[]` trong `<script>` cho đúng build.

## Step 4: Priority Rules

### P0 (PHẢI test — block release)
- Navigation crash
- Data hiển thị sai
- Dark mode white-on-white
- App freeze / blank screen

### P1 (NÊN test — important)
- Animation smooth
- Layout match design
- Error states show correctly
- Vietnamese text đúng dấu

### P2 (KIỂM TRA nếu có thời gian)
- Edge cases (rotate, background, low memory)
- Performance (load time, scroll fps)
- Accessibility (VoiceOver, contrast)

## Step 5: Estimate Test Time

Based on item count:
- 1-5 items: ~5 phút
- 6-15 items: ~15 phút
- 16-30 items: ~30 phút
- 30+: ~45-60 phút (recommend split sessions)

## Step 6: Map to Project Charter (2026-03-15+)

If a Project Charter exists for the current phase:
```bash
PHASE=$(grep -r "Phase hiện tại" memory/flutter-migration.md 2>/dev/null | grep -oP "P\d+" | head -1)
CHARTER="Product/kfsp_flutter_migration/docs/build_reports/${PHASE}_project_charter.md"
[ -f "$CHARTER" ] && echo "✅ Mapping test items to charter deliverables"
```

**Map each test item to charter deliverables:**
- Test item covers Deliverable X → tag as "Charter: X"
- Deliverable without test coverage → ⚠️ FLAG: "Deliverable [X] has no test items"
- Add charter coverage summary to test brief header

## Integration with Build Report

**BẮT BUỘC:** Mỗi build report PHẢI có section `🧪 Test Checklist cho Thanh`.
Nếu build report chưa có → skill sẽ append vào.
Nếu đã có → verify completeness, update nếu cần.
</instructions>
