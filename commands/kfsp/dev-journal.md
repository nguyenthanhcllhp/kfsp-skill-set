---
name: kfsp:dev-journal
description: "Developer journal — record decisions, incidents, learnings as 'precedents' for future diagnosis. Like case law for your product. Tier 1-5: All tiers (context preservation)"
argument-hint: "[--log <type>] [--search <keyword>] [--digest] [--init]"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Dev Journal — Nhật ký phát triển / Hệ thống "Án lệ".

**Tầng:** Xuyên suốt T1→T5 (context preservation cho cả human + agent)

**Triết lý:**
Giống như "nhật ký giao dịch" trong đầu tư — ghi lại MỌI quyết định quan trọng,
không phải để đọc lại hàng ngày, mà để KHI CẦN DIAGNOSE có căn cứ.

Mỗi entry là một "án lệ" (precedent) — khi gặp tình huống tương tự trong tương lai,
agent hoặc human có thể search journal để tìm:
- "Lần trước gặp lỗi này, đã xử lý thế nào?"
- "Tại sao lại chọn approach A thay vì B?"
- "Bài học gì rút ra từ incident X?"

**Đặc biệt quan trọng cho Agent team/swarm:**
- Agent mới join không có context → search journal
- Agent gặp vấn đề → journal có precedent
- Giảm hallucination vì có evidence-based decisions
- Giữ chặt bối cảnh xuyên suốt lifecycle

**Modes:**
- `--log <type>` → Ghi entry mới (decision/incident/learning/discovery/experiment)
- `--search <keyword>` → Tìm precedent liên quan
- `--digest` → Tổng hợp digest tuần/tháng
- `--init` → Khởi tạo journal cho project mới
</objective>

<instructions>
## Journal Location

```
Product/kfsp_flutter_migration/journal/
├── INDEX.md                    ← Master index (searchable)
├── 2026-03/                    ← Theo tháng
│   ├── 2026-03-08_yolo-phase-failure.md
│   ├── 2026-03-10_clean-slate-restart.md
│   ├── 2026-03-11_onboarding-design-decisions.md
│   └── ...
├── 2026-04/
│   └── ...
├── tags/                       ← Tag index
│   ├── architecture.md
│   ├── performance.md
│   ├── ui-ux.md
│   └── ...
└── DIGEST.md                   ← Weekly/monthly digest
```

## Mode: --init

Create journal structure for new project:

```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
mkdir -p "$JOURNAL_DIR/$(date +%Y-%m)" "$JOURNAL_DIR/tags"
```

Create INDEX.md:
```markdown
# 📔 KFSP Dev Journal — Index

> Nhật ký phát triển KFSP. Mỗi entry là một "án lệ" — căn cứ cho decisions tương lai.

## Cách sử dụng
- **Ghi entry:** `/kfsp:dev-journal --log decision` (hoặc incident/learning/discovery/experiment)
- **Tìm kiếm:** `/kfsp:dev-journal --search <keyword>`
- **Tổng hợp:** `/kfsp:dev-journal --digest`

## Entry Types

| Type | Icon | Khi nào ghi |
|------|------|------------|
| decision | 🔷 | Khi chọn approach/architecture/tool |
| incident | 🔴 | Khi có bug/crash/failure |
| learning | 🟢 | Khi rút ra bài học |
| discovery | 🟡 | Khi phát hiện điều bất ngờ |
| experiment | 🔬 | Khi thử approach mới |

## All Entries
| Date | Type | Title | Tags | Link |
|------|------|-------|------|------|
```

## Mode: --log <type>

### Entry Types:

**🔷 Decision** — "Tại sao chọn A thay vì B?"
```markdown
# 🔷 Decision: [Title]
**Date:** [YYYY-MM-DD]
**Context:** [Tình huống dẫn đến quyết định]
**Options Considered:**
1. [Option A] — Pros: ... Cons: ...
2. [Option B] — Pros: ... Cons: ...
3. [Option C] — Pros: ... Cons: ...
**Decision:** [Chọn option nào]
**Rationale:** [Tại sao — lý do chính]
**Trade-offs Accepted:** [Nhược điểm chấp nhận]
**Affected Areas:** [Surfaces/files bị ảnh hưởng]
**Tags:** [architecture, routing, state-management, ...]
**Review Date:** [Khi nào xem lại — 1 month? 1 quarter?]
```

**🔴 Incident** — "Chuyện gì xảy ra, fix thế nào?"
```markdown
# 🔴 Incident: [Title]
**Date:** [YYYY-MM-DD]
**Severity:** Critical / Major / Minor
**Symptom:** [User/dev thấy gì]
**Root Cause:** [Nguyên nhân gốc — 1 câu]
**Fix Applied:** [Sửa gì, file nào]
**Time to Fix:** [Bao lâu]
**Could Have Prevented By:** [Test/check nào nên có]
**Affected Surfaces:** [Front/Back/Lateral/Internal]
**Tags:** [crash, ui-bug, api-error, ...]
**Related Entries:** [Link đến entries liên quan]
```

**🟢 Learning** — "Bài học rút ra"
```markdown
# 🟢 Learning: [Title]
**Date:** [YYYY-MM-DD]
**Source:** [Từ incident nào / task nào / nghiên cứu nào]
**Lesson:** [1-2 câu core lesson]
**Evidence:** [Cụ thể gì chứng minh lesson này đúng]
**Applied To:** [Đã áp dụng ở đâu]
**Should Apply To:** [Nên áp dụng thêm ở đâu]
**Tags:** [testing, code-quality, process, ...]
```

**🟡 Discovery** — "Phát hiện bất ngờ"
```markdown
# 🟡 Discovery: [Title]
**Date:** [YYYY-MM-DD]
**What Found:** [Phát hiện gì]
**Where:** [Ở đâu — file, feature, service]
**Implication:** [Ý nghĩa — có thể ảnh hưởng gì]
**Action Taken:** [Đã làm gì — hoặc chưa làm gì]
**Tags:** [performance, security, compatibility, ...]
```

**🔬 Experiment** — "Thử approach mới"
```markdown
# 🔬 Experiment: [Title]
**Date:** [YYYY-MM-DD]
**Hypothesis:** [Giả thuyết — "nếu làm X thì Y sẽ xảy ra"]
**Method:** [Làm thế nào để test]
**Result:** [Kết quả — confirm/reject hypothesis]
**Conclusion:** [Rút ra gì]
**Next Step:** [Áp dụng rộng hơn? Bỏ? Iterate?]
**Tags:** [performance, ux, architecture, ...]
```

### Workflow khi ghi entry:

1. Ask user cho context (hoặc tự extract từ conversation)
2. Tạo file: `journal/YYYY-MM/YYYY-MM-DD_slug-title.md`
3. Update `journal/INDEX.md` — thêm row vào table
4. Update relevant tag file trong `journal/tags/`
5. Check: có liên kết đến entries cũ không? → thêm "Related Entries"

## Mode: --search <keyword>

Tìm precedent trong journal:

```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
# Search across all entries
grep -rl "$KEYWORD" "$JOURNAL_DIR/" --include="*.md" | grep -v INDEX | grep -v DIGEST
# Also search by tag
grep -l "$KEYWORD" "$JOURNAL_DIR/tags/"*.md 2>/dev/null
```

Output:
```markdown
# 🔎 Journal Search: "[keyword]"

## Matching Entries (X found)
| Date | Type | Title | Relevance |
|------|------|-------|-----------|
| [date] | [icon] | [title] | [why relevant] |

## Key Precedents
[For each highly relevant entry, summarize the lesson/decision]

## Recommendation
[Based on precedents, what should we do now?]
```

## Mode: --digest

Generate weekly/monthly summary:

```bash
# Find entries from current week/month
find "$JOURNAL_DIR/$(date +%Y-%m)/" -name "*.md" -newer "$SINCE_DATE" 2>/dev/null
```

```markdown
# 📊 Dev Journal Digest — [Period]

## Stats
| Metric | Count |
|--------|-------|
| Total entries | X |
| Decisions | X |
| Incidents | X |
| Learnings | X |
| Discoveries | X |
| Experiments | X |

## Top Themes
1. [most common tag] — X entries
2. [second] — X entries

## Key Decisions This Period
[summarize]

## Unresolved Items
[entries with pending actions]

## Patterns Detected
[recurring issues, common root causes]
```

## Auto-Trigger Hints

Doc Pilot và các skills khác NÊN gợi ý ghi journal:

| After... | Suggest |
|----------|---------|
| `/kfsp:incident-review` | `--log incident` |
| `/kfsp:pre-mortem` (Go/No-Go decision) | `--log decision` |
| `/kfsp:post-phase` | `--log learning` |
| `/kfsp:guard --deep` finds issue | `--log discovery` |
| Dev thử approach mới | `--log experiment` |

## Seeding: Pre-populate from Existing History

On `--init`, also seed from existing sources:
- `docs/06_DEBUG_LOG.md` → convert to incident entries
- YOLO phase failure → incident + learning entries
- Clean Slate restart decision → decision entry
- Onboarding design decisions → decision entries

This gives new agents/devs immediate context even on day 1.
</instructions>
