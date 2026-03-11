---
name: kfsp:sweep
description: Run full impact analysis after code changes — detect blast radius, hidden dependencies, sync issues across design tokens, routes, providers, widgets, tests, and docs
argument-hint: "[changed-file-or-area]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Impact Sweep — Quét tầm soát toàn diện sau khi thay đổi code.

Khi dev thay đổi bất kỳ file/component nào, sweep sẽ:
1. Xác định blast radius (file nào bị ảnh hưởng thứ cấp)
2. Kiểm tra đồng bộ design tokens, routes, providers, widgets
3. Phát hiện hardcoded values, missing imports, broken references
4. Báo cáo rủi ro + đề xuất action items

**KHÔNG tự sửa code** — chỉ báo cáo. Dev quyết định sửa.
</objective>

<instructions>
## Input
User có thể cung cấp:
- Tên file vừa thay đổi (e.g., `kfsp_colors.dart`)
- Tên area (e.g., "design tokens", "routing", "onboarding")
- Không cung cấp gì → quét toàn bộ (full sweep)

Nếu argument là `$ARGUMENTS`, parse nó. Nếu rỗng, chạy full sweep.

## Step 1: Xác định scope thay đổi

Đọc dependency map reference:
```
Product/kfsp_flutter_migration/docs/13_DEPENDENCY_IMPACT_MAP.md
```

Nếu user chỉ định file cụ thể → tra bảng Impact Matrix trong doc 13 để tìm blast radius.
Nếu full sweep → kiểm tra tất cả categories.

## Step 2: Design Token Compliance (🎨)

Quét source code Flutter tại `/tmp/kfsp_flutter/lib/` hoặc migration source:

```
Grep: Color(0xFF  → trong tất cả .dart files NGOẠI TRỪ kfsp_colors.dart
Grep: EdgeInsets.all( với số cứng → ngoại trừ kfsp_spacing.dart
Grep: TextStyle( với fontSize cứng → ngoại trừ kfsp_text_styles.dart
Grep: fontFamily: ' → ngoại trừ kfsp_text_styles.dart
```

Mỗi violation = 1 finding với file:line + severity.

## Step 3: Route & Navigation Check (🧭)

```
Grep: GoRoute( → liệt kê tất cả routes
Grep: context.go( hoặc context.push( → liệt kê tất cả navigation calls
Cross-check: mỗi navigation call có route tương ứng không?
Check: app_router.dart → mọi route có screen import đúng không?
```

## Step 4: Provider Dependency Check (⚡)

```
Grep: Provider( hoặc Notifier( → liệt kê providers
Grep: ref.watch( hoặc ref.read( → liệt kê consumers
Cross-check: mỗi consumer ref đến provider tồn tại?
Check: circular dependency? (A watch B, B watch A)
```

## Step 5: Widget Tree Integrity (🧩)

```
Grep: import.*widgets/ → check import paths valid
Grep: class.*extends.*Widget → liệt kê tất cả widgets
Check: mỗi widget có được sử dụng ít nhất 1 nơi? (dead widget detection)
```

## Step 6: Mock Data Consistency (📊)

```
Read: mock_data.dart → check mock data structure
Grep: kDebugMode → liệt kê tất cả debug bypasses
Check: mỗi screen có mock data path khi kDebugMode = true?
```

## Step 7: Test Coverage Gap (🧪)

```
Glob: test/**/*_test.dart → existing tests
Cross-check với lib/ files: file nào chưa có test tương ứng?
Check: test file nào reference file đã bị xóa/rename?
```

## Step 8: Doc Sync Check (📝)

Kiểm tra:
- docs/08_HANDOFF_STATUS.md → progress bars có khớp thực tế?
- docs/04_FEATURE_SPECS.md → features listed = features trong code?
- README.md → known issues list có cập nhật?

## Output Format

Tạo báo cáo theo format:

```markdown
# 🔍 KFSP Impact Sweep Report
**Date:** [timestamp]
**Scope:** [full / specific files]
**Source:** [path to source code scanned]

## Summary
| Category | Findings | Critical | Warning | Info |
|----------|----------|----------|---------|------|
| 🎨 Design Tokens | X | X | X | X |
| 🧭 Routes | X | X | X | X |
| ⚡ Providers | X | X | X | X |
| 🧩 Widgets | X | X | X | X |
| 📊 Mock Data | X | X | X | X |
| 🧪 Tests | X | X | X | X |
| 📝 Docs | X | X | X | X |

## 🔴 Critical Findings
[list with file:line, description, suggested fix]

## 🟡 Warnings
[list]

## 🟢 Info
[list]

## Recommended Actions
1. [prioritized action items]
```

Nếu KHÔNG có findings → báo "✅ Clean sweep — no issues detected."
</instructions>
