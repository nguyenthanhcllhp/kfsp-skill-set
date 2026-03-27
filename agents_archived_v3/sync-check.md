---
name: kfsp:sync-check
description: Verify all source copies are in sync, docs are up-to-date, and no drift between working copy, migration package, and build copy
argument-hint: "[--fix]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Sync Check — Kiểm tra đồng bộ giữa các bản source code và tài liệu.

Dự án có nhiều bản copy cần đồng bộ:
1. `/tmp/kfsp_flutter/` — Working copy (dev đang code)
2. `~/Desktop/kfsp_flutter/` — Build copy (rsync để build iOS)
3. `Product/kfsp_flutter_migration/source/` — Migration package (handoff)

Plus tài liệu cần reflect đúng trạng thái code.

**Mục tiêu:** Phát hiện drift trước khi nó gây vấn đề.
</objective>

<instructions>
## Input
- Không argument → full sync check
- `--fix` → sau khi báo cáo, đề xuất lệnh rsync cụ thể để fix (KHÔNG tự chạy)

## Step 1: Source Code Sync Check

So sánh 3 bản source (nếu tồn tại):

```bash
# Check if directories exist
ls -la /tmp/kfsp_flutter/lib/ 2>/dev/null
ls -la ~/Desktop/kfsp_flutter/lib/ 2>/dev/null
ls -la "Product/kfsp_flutter_migration/source/lib/" 2>/dev/null
```

Nếu >= 2 bản tồn tại, so sánh:
```bash
# File count comparison
find /tmp/kfsp_flutter/lib -name "*.dart" | wc -l
find ~/Desktop/kfsp_flutter/lib -name "*.dart" | wc -l

# Diff check (file list only, not content — avoid huge output)
diff <(cd /tmp/kfsp_flutter && find lib -name "*.dart" | sort) \
     <(cd ~/Desktop/kfsp_flutter && find lib -name "*.dart" | sort) 2>/dev/null

# Check modification times
find /tmp/kfsp_flutter/lib -name "*.dart" -newer ~/Desktop/kfsp_flutter/pubspec.yaml 2>/dev/null | head -20
```

## Step 2: pubspec.yaml Sync

```bash
diff /tmp/kfsp_flutter/pubspec.yaml ~/Desktop/kfsp_flutter/pubspec.yaml 2>/dev/null
diff /tmp/kfsp_flutter/pubspec.yaml "Product/kfsp_flutter_migration/source/pubspec.yaml" 2>/dev/null
```

Phát hiện: dependency version mismatch, missing packages.

## Step 3: Assets Sync

```bash
# Compare asset directories
diff <(cd /tmp/kfsp_flutter && find assets -type f | sort) \
     <(cd ~/Desktop/kfsp_flutter && find assets -type f | sort) 2>/dev/null
```

## Step 4: Doc Accuracy Check

Đọc các docs và cross-check với code:

### 4a: Tech Stack (docs/02)
- Check: packages in pubspec.yaml = packages listed in doc 02?
- Check: Flutter/Dart version trong doc = thực tế?

### 4b: Handoff Status (docs/08)
- Check: phase completion status trong doc = features có trong code?
- Check: "Known Issues" list = issues thực tế?

### 4c: Feature Specs (docs/04)
- Check: features listed = features có folder trong lib/features/?

### 4d: README.md
- Check: folder structure trong README = folder structure thực tế?
- Check: build commands trong README có hoạt động?

### 4e: Design Tokens (docs/03)
- Check: token count trong doc = token count trong kfsp_colors.dart?
- Check: hex values match?

## Step 5: QA Testcase Coverage

```
# So sánh features trong code vs features có testcases
ls Product/kfsp_flutter_migration/qa_testcases_RN_REFERENCE/
ls Product/kfsp_flutter_migration/source/lib/features/
```

Cross-check: feature nào trong code chưa có QA testcases?

## Output Format

```markdown
# 🔄 KFSP Sync Check Report
**Date:** [timestamp]

## Source Copies Status
| Copy | Path | Exists | Dart Files | Last Modified |
|------|------|--------|-----------|---------------|
| Working | /tmp/kfsp_flutter/ | ✅/❌ | N | [date] |
| Build | ~/Desktop/kfsp_flutter/ | ✅/❌ | N | [date] |
| Package | migration/source/ | ✅/❌ | N | [date] |

## Sync Status
| Pair | Status | Drift Files |
|------|--------|-------------|
| Working ↔ Build | ✅ Synced / ⚠️ X files differ | [list] |
| Working ↔ Package | ✅ Synced / ⚠️ X files differ | [list] |

## Doc Accuracy
| Doc | Status | Issues |
|-----|--------|--------|
| 02 Architecture | ✅/⚠️ | [details] |
| 04 Features | ✅/⚠️ | [details] |
| 08 Handoff | ✅/⚠️ | [details] |
| README | ✅/⚠️ | [details] |

## QA Coverage
| Feature | Code | QA Testcases | Gap |
|---------|------|-------------|-----|
| ... | ✅ | ✅/❌ | ... |

## Fix Commands (if --fix)
[rsync commands to run]
```
</instructions>
