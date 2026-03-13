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

## Step 3: Generate Test Brief

### Output Location
Write to: `Product/kfsp_flutter_migration/docs/build_reports/` (append to existing build report)

### Output Format

```markdown
## 🧪 Test Checklist cho Thanh

**Build:** [build date + scope]
**Device:** iPhone Simulator (iOS XX) — test cả dark + light mode
**Thời gian ước tính:** X phút

### Hướng dẫn test
1. Mở app trên simulator
2. Test từng item theo bảng bên dưới
3. Đánh dấu ✅ (pass) hoặc ❌ (fail) + ghi chú
4. Bug → dùng `/kfsp:bug-log` hoặc ghi vào bảng

### Test Items

| # | Category | Test item | Kỳ vọng | Cách test | Priority |
|---|----------|-----------|---------|-----------|----------|
| 1 | 🎨 UI | [item] | [expected] | [steps] | P0/P1/P2 |
| 2 | 🧭 Nav | [item] | [expected] | [steps] | P0/P1/P2 |
| ... | | | | | |

### Dark Mode Checklist
| # | Screen | Light mode OK | Dark mode OK |
|---|--------|---------------|--------------|
| 1 | [screen] | ⬜ | ⬜ |

### Regression Check (features cũ vẫn hoạt động)
| # | Feature | Test nhanh | Expected |
|---|---------|------------|----------|
| 1 | Onboarding 5 slides | Swipe qua 5 slides | Smooth, dot indicator đúng |
| 2 | Bottom nav 6 tabs | Tap từng tab | Tab switch, content hiện |
| ... | | | |

### Known Limitations (KHÔNG cần test)
- [list items that are known issues / out of scope]

### Bug Report Template
Nếu tìm thấy bug, ghi:
- **Ở đâu:** [screen name]
- **Làm gì:** [steps]
- **Kỳ vọng:** [expected]
- **Thực tế:** [actual]
- **Screenshot:** [chụp]
- **Severity:** Critical / High / Medium / Low
```

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

## Integration with Build Report

**BẮT BUỘC:** Mỗi build report PHẢI có section `🧪 Test Checklist cho Thanh`.
Nếu build report chưa có → skill sẽ append vào.
Nếu đã có → verify completeness, update nếu cần.
</instructions>
