---
name: kfsp:log
description: "Dev journal — record decisions, incidents, learnings. 5-Whys for incidents. Search precedents. Replaces dev-journal + incident-review."
---

# /kfsp:log — Ghi chép

## Mục đích
Ghi quyết định, sự cố, bài học. Phân tích 5-Whys cho incident. Tìm kiếm precedent.

**Gộp từ:** dev-journal + incident-review

## Modes

### --log [type]
Ghi 1 entry vào dev journal. Types:
- `decision` — quyết định kỹ thuật/design
- `incident` → chuyển sang --incident mode
- `learning` — bài học rút ra
- `pm-feedback` — feedback từ Thanh
- `bug-fix` — ghi cách fix bug
- `phase` — ghi tổng kết phase

**Format entry:**
```markdown
## [DATE] [TYPE] — [Title]
**Context:** Tại sao quyết định này?
**Decision:** Chọn gì?
**Alternatives:** Đã cân nhắc gì khác?
**Impact:** Ảnh hưởng docs/code nào?
**Confidence:** High/Medium/Low
```

**Lưu tại:** `docs/06_DECISIONS_LOG.md` + session HTML

### --incident
Phân tích sự cố theo 5-Whys. Cho bugs nghiêm trọng.

**Template:**
```markdown
## INCIDENT — [Title]
**Severity:** Critical/Major/Minor
**Timeline:** Khi nào phát hiện, ai phát hiện

### 5-Whys
1. Why? [Symptom]
2. Why? [Cause 1]
3. Why? [Cause 2]
4. Why? [Root cause]
5. Why? [Systemic issue]

### Impact (4 bề mặt)
- User: [ảnh hưởng user thế nào]
- Backend: [ảnh hưởng API]
- Services: [ảnh hưởng bên thứ 3]
- Internal: [ảnh hưởng code/components]

### Fix
- Immediate: [fix ngay]
- Prevention: [tránh tái diễn]

### Lesson
[Bài học rút ra]
```

### --search [keyword]
Tìm kiếm trong dev journal theo keyword. Hữu ích khi gặp vấn đề tương tự trước đây.

### --digest
Tổng hợp entries gần đây (7 ngày). Cho PM review.
