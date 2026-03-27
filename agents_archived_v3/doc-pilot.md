---
name: kfsp:doc-pilot
description: "Documentation autopilot — tracks what docs need updating, reminds during dev, ensures HDSD/guides ready before store submission. Tier 4-5: User Experience + Business Impact"
argument-hint: "[--status] [--what-changed <feature>] [--hdsd-gap] [--checklist]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Doc Pilot — Hoa tiêu tài liệu.

**Tầng:** 🎯 T5 Business Impact + 👤 T4 User Experience

**Triết lý:** "Làm đến đâu, tài liệu đến đó."
Không đợi xong mới viết doc. Doc phải sẵn sàng TRƯỚC KHI submit store.

Doc Pilot biết:
- **GHI GÌ** → loại thông tin nào cần capture
- **VÀO ĐÂU** → file nào trong hệ thống tài liệu
- **KHI NÀO** → trigger tự động sau mỗi thay đổi
- **CHO AI** → human (HDSD), agent (migration docs), QA (test docs)

**3 hệ thống tài liệu KFSP:**

```
1. HDSD (User Guide)         → Product/HDSD_KFSP/ (GitBook, 168 trang)
   Đối tượng: End user       → Hướng dẫn sử dụng từng tính năng
   Cần: ready trước store

2. Migration Docs (Dev)      → Product/kfsp_flutter_migration/docs/ (15 files)
   Đối tượng: Dev, QA, PM    → Architecture, specs, testing, deployment
   Cần: cập nhật song song dev

3. Dev Journal (Precedent)   → Xem /kfsp:dev-journal
   Đối tượng: Future agent/dev → Kinh nghiệm, decisions, bài học
   Cần: ghi ngay khi xảy ra
```

**Modes:**
- `--status` → Bảng trạng thái: doc nào outdated, doc nào missing
- `--what-changed <feature>` → Feature X thay đổi → liệt kê docs cần update
- `--hdsd-gap` → So sánh features trong code vs HDSD pages → gap
- `--checklist` → Pre-release doc checklist
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `--status` → overview trạng thái tài liệu
- `--what-changed <feature>` → feature-specific doc update list
- `--hdsd-gap` → HDSD gap analysis
- `--checklist` → pre-release checklist
- Rỗng → `--status`

## DOC REGISTRY — Ghi gì vào đâu

### Khi BUILD/SỬA feature:

| Thay đổi | Ghi vào | Mô tả ghi |
|----------|---------|-----------|
| UI mới/sửa | `docs/04_FEATURE_SPECS.md` | Screen name, behavior, data source (real API + offline fallback) |
| UI mới/sửa | `HDSD_KFSP/` trang tương ứng | Screenshot + hướng dẫn user step-by-step |
| API mới/sửa | `docs/10_API_SOCKET_SPEC.md` | Endpoint, request/response format |
| Design token mới/sửa | `docs/03_DESIGN_SYSTEM.md` | Token name, value, where used |
| Route mới/sửa | `docs/02_ARCHITECTURE_GUIDE.md` | Route path, screen, params |
| Dependency mới | `docs/02_ARCHITECTURE_GUIDE.md` | Package name, version, purpose |
| Bug fix quan trọng | `docs/06_DEBUG_LOG.md` | Root cause, fix, prevention |
| Test case mới | `docs/07_TESTING_QA.md` | Test name, what it covers |

### Khi HOÀN THÀNH phase:

| Action | Ghi vào | Mô tả ghi |
|--------|---------|-----------|
| Phase complete | `docs/08_HANDOFF_STATUS.md` | Progress update, what works, known bugs |
| Phase complete | `docs/04_FEATURE_SPECS.md` | Move features to "Implemented" section |
| Phase complete | Dev Journal (via `/kfsp:dev-journal`) | Decisions, learnings, surprises |

### Khi CHUẨN BỊ release:

| Action | Ghi vào | Mô tả ghi |
|--------|---------|-----------|
| App Store submission | `docs/09_BUILD_DEPLOY.md` | Build version, release notes |
| App Store submission | `HDSD_KFSP/cap-nhat-thay-doi-phien-ban.md` | Changelog cho user |
| All new features | `HDSD_KFSP/` các trang tính năng | Screenshots + step-by-step guides |
| Privacy/permissions | `HDSD_KFSP/chinh-sach-bao-mat.md` | Nếu permission mới |

## Mode: --status

### 1. Migration Docs Status
```bash
# Check each doc's last modified date
for doc in Product/kfsp_flutter_migration/docs/*.md; do
  echo "$(stat -f '%Sm' -t '%Y-%m-%d' "$doc") | $(basename "$doc")"
done | sort -r
```

Cross-check:
- Source code last modified vs doc last modified
- If code newer than doc → 🟡 Potentially outdated

### 2. HDSD Status
```bash
# Count HDSD pages
find Product/HDSD_KFSP/ -name "*.md" ! -path "*/.git/*" | wc -l
# Check last updated
stat -f '%Sm' Product/HDSD_KFSP/SUMMARY.md
```

### 3. Feature Coverage Check
Read feature list from `docs/04_FEATURE_SPECS.md`.
For each implemented feature:
- Has HDSD page? → ✅/❌
- HDSD page up-to-date? → ✅/🟡/❌
- Has migration doc coverage? → ✅/❌

### Output:
```markdown
# 📚 KFSP Doc Pilot — Status Report
**Date:** [timestamp]

## Doc Systems Health
| System | Files | Last Updated | Health |
|--------|-------|-------------|--------|
| Migration Docs | 15 | [date] | ✅/🟡/❌ |
| HDSD (User Guide) | 168 | [date] | ✅/🟡/❌ |
| Dev Journal | X entries | [date] | ✅/🟡/❌ |

## Docs Potentially Outdated
| Doc | Last Updated | Code Changed Since | Action |
|-----|-------------|-------------------|--------|

## HDSD Feature Coverage
| Feature | In Code | HDSD Page | Status |
|---------|---------|-----------|--------|

## Immediate Actions
1. [highest priority doc to update]
```

## Mode: --what-changed <feature>

Given a feature name (e.g., "dashboard", "onboarding", "priceboard"):

### 1. Identify code files for feature
```bash
find /tmp/kfsp_flutter/lib/features/<feature>/ -name "*.dart" 2>/dev/null
```

### 2. Map to docs that need updating
Based on DOC REGISTRY above, list:

```markdown
# 📝 Doc Update Required: [feature]

## Files Changed
[list of code files]

## Docs to Update

### MUST update (code logic changed):
| Doc | Section | What to Write |
|-----|---------|---------------|
| docs/04_FEATURE_SPECS.md | [feature] section | Updated behavior, new screens |
| docs/08_HANDOFF_STATUS.md | Progress section | Mark feature status |

### SHOULD update (UX changed):
| Doc | Section | What to Write |
|-----|---------|---------------|
| HDSD_KFSP/[related page] | Step-by-step | New screenshots + updated guide |

### MAY need update (if applicable):
| Doc | Section | Condition |
|-----|---------|-----------|
| docs/10_API_SOCKET_SPEC.md | API section | If API endpoints changed |
| docs/03_DESIGN_SYSTEM.md | Tokens | If new colors/spacing |
| docs/13_DEPENDENCY_IMPACT_MAP.md | Impact matrix | If new dependencies |
```

## Mode: --hdsd-gap

Compare features in code vs HDSD pages.

### 1. Extract features from code
```bash
ls /tmp/kfsp_flutter/lib/features/ 2>/dev/null || ls Product/kfsp_flutter_migration/source/lib/features/
```

### 2. Extract HDSD pages
```bash
cat Product/HDSD_KFSP/SUMMARY.md
```

### 3. Map features → HDSD pages

KFSP Feature → HDSD Mapping:
| Feature (code) | HDSD Section | HDSD Path |
|----------------|-------------|-----------|
| onboarding | Đăng ký/Đăng nhập | `ban-thong-nhat/10-tai-khoan/` |
| dashboard | Trang Dashboard | `phien-ban-app/` hoặc `phien-ban-web/trang-dashboard.md` |
| priceboard | Bảng giá | `phien-ban-web/bang-gia-*.md` |
| tech_analysis | Biểu đồ kỹ thuật | `phien-ban-app/bieu-do.md` |
| fundamental | Phân tích FA | `phien-ban-app/phan-tich-fa/` |
| watchlist | Watchlist | `ban-thong-nhat/8-watchlist/` |
| alert | Cảnh báo | (check SUMMARY.md) |
| trading_journal | Nhật ký GD | `phien-ban-web/nhat-ky-giao-dich/` |
| market | Thị trường | `phien-ban-app/thi-truong/` |
| filter | Bộ lọc | `phien-ban-app/bo-loc.md` |

### 4. Gap Report
```markdown
# 📊 HDSD Gap Analysis
**Date:** [timestamp]

## Coverage: X / Y features have HDSD pages

### ✅ Covered (HDSD exists + up to date)
[list]

### 🟡 Exists but may be outdated (Flutter version differs from web/RN)
[list — these are pages written for web/RN that need update for Flutter]

### ❌ Missing (no HDSD page)
[list]

### 📸 Screenshots Needed
[features where HDSD exists but screenshots show old UI]
```

## Mode: --checklist

Pre-release documentation checklist:

```markdown
# 📋 KFSP Pre-Release Doc Checklist

## A. User-Facing (PHẢI xong trước submit store)
- [ ] HDSD: Tất cả features mới có hướng dẫn sử dụng
- [ ] HDSD: Screenshots được update với UI mới nhất
- [ ] HDSD: Changelog page cập nhật (`cap-nhat-thay-doi-phien-ban.md`)
- [ ] HDSD: FAQ cập nhật nếu có tính năng mới
- [ ] HDSD: 7-day trial roadmap phù hợp features hiện tại
- [ ] App Store: Description text cập nhật
- [ ] App Store: Screenshots (5+ kích thước) cập nhật
- [ ] App Store: "What's New" release notes
- [ ] Privacy Policy: cập nhật nếu permissions mới

## B. Dev-Facing (PHẢI xong trước handoff)
- [ ] docs/04: Feature specs mô tả tất cả implemented features
- [ ] docs/08: Handoff status updated (what works, known bugs)
- [ ] docs/10: API spec đầy đủ cho endpoints đang dùng
- [ ] docs/02: Architecture guide phản ánh code thực tế
- [ ] docs/03: Design system tokens đồng bộ code ↔ doc
- [ ] docs/09: Build/deploy guide tested (ai đó build theo guide được)

## C. Quality (NÊN xong)
- [ ] docs/07: Test cases cover all features
- [ ] docs/12: Testing lifecycle milestones checked off
- [ ] docs/13: Dependency map updated for new features
- [ ] docs/14: Interaction surface contracts verified

## D. Knowledge (NÊN có)
- [ ] Dev Journal: tất cả decisions quan trọng được ghi
- [ ] docs/06: Debug log cập nhật incidents gần đây
- [ ] docs/11: Rebuild plan status updated

## Score: [X] / [total] items complete
## READY: ✅ / ⚠️ / 🛑
```

## Auto-Trigger Logic

Doc Pilot NÊN được gọi tự động:
- Sau `/kfsp:post-phase` → show `--what-changed` cho features trong phase
- Trước `/kfsp:release-gate` → run `--checklist`
- Tuần 1 lần → `--status` trong `/kfsp:guard --deep`
- Khi user hỏi "tài liệu nào cần update?" → `--status`
</instructions>
