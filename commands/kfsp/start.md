---
name: kfsp:start
description: "New session startup — git check, drift detect, create .changes.json, remind dual-session. Run at the start of every dev session."
---

# /kfsp:start — Mở phiên làm việc

## Mục đích
Khởi tạo phiên làm việc: kiểm tra git, phát hiện drift code-docs, tạo file sync, nhắc bật dual-session.

## Quy trình

### 1. Git Status
```bash
cd Product/kfsp_flutter_migration/current_source/flutter_kfsp_app
git status --short
git log --oneline -5
```
- Báo cáo uncommitted changes
- Báo cáo branch hiện tại

### 2. Drift Detection
- Đọc `docs/logs/.changes.json` — nếu còn entries `processed: false` từ phiên trước → cảnh báo
- So sánh timestamps: code files vs docs files → phát hiện docs lỗi thời

### 3. Tạo .changes.json mới
Nếu file rỗng hoặc phiên mới:
```json
{
  "session": "YYYY-MM-DD",
  "phase": "",
  "entries": []
}
```
Nếu có entries cũ chưa processed → giữ lại, cảnh báo Thanh.

### 4. Nhắc Dual-Session (Rule 20)
> "Anh bật thêm 1 phiên Claude Code trên Terminal để chạy agent rà soát tài liệu/spec/test case song song."

### 5. Đọc plan hiện tại
- Đọc file plan mới nhất trong `docs/plans/`
- Tóm tắt: phase nào, tasks nào đang in_progress

### 6. Mở Live Guide (TỰ ĐỘNG)
TỰ ĐỘNG mở Live Guide trong browser:
```bash
open "Product/kfsp-skill-set/KFSP_Live_Guide.html"
```
Nếu `open` không khả dụng, nhắc Thanh mở thủ công.

## Output
```
🥋 KFSP Session Start — [DATE]
├── Git: branch [X], [N] uncommitted files
├── Drift: [N] docs có thể lỗi thời
├── Sync: [N] entries chưa xử lý từ phiên trước
├── Phase: [current phase + status]
├── 📊 Live Guide: đã mở trong browser
└── ⚠️ Nhắc: Bật Terminal để rà soát docs song song
```
