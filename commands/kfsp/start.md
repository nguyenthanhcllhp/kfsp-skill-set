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

### 4. Hỏi phiên iOS hay Android (Rule 20)

Branch duy nhất: `thanh_sg`. Cả 2 phiên dùng chung branch, edit trong `current_source/`.

Agent PHẢI hỏi: **"Phiên này làm iOS hay Android?"** rồi:
- Khai báo files/folders sẽ đụng
- Kiểm tra không trùng phiên kia (nếu biết phiên kia đang làm gì)
- Nhắc quy tắc: KHÔNG sửa cùng 1 file ở 2 phiên, commit tuần tự, shared files dồn cuối

### 5. Hướng dẫn Dual-Session Terminal

Agent PHẢI output block hướng dẫn chi tiết (KHÔNG chỉ nhắc chung chung):

```
🖥️ MỞ TERMINAL — 4 bước:
1. Cmd+Space → gõ "Terminal" → Enter
2. Paste: cd "$HOME/Library/CloudStorage/OneDrive-Personal/Work/KFSP/Antigravity/Product"
3. Gõ: claude  (chờ dấu nhắc >)
4. Paste vào >: Tôi là phiên Terminal cho KFSP. Vai trò: rà soát và cập nhật docs, test cases, specs. KHÔNG đụng lib/ hoặc test/. Đọc Product/CLAUDE.md trước.

✅ Đúng khi: dấu nhắc >, Claude phản hồi tiếng Việt, nhắc KFSP
❌ Sai khi: "command not found" → chạy: npm install -g @anthropic-ai/claude-code
```

**KHÔNG ĐƯỢC** chỉ viết "Anh bật Terminal" mà không kèm 4 bước trên.

### 5. Đọc plan hiện tại
- Đọc file plan mới nhất trong `docs/plans/`
- Tóm tắt: phase nào, tasks nào đang in_progress

## Output
```
🥋 KFSP Session Start — [DATE]
├── Git: branch [X], [N] uncommitted files
├── Drift: [N] docs có thể lỗi thời
├── Sync: [N] entries chưa xử lý từ phiên trước
├── Phase: [current phase + status]
│
├── 🖥️ MỞ TERMINAL — 4 bước:
│   1. Cmd+Space → "Terminal" → Enter
│   2. cd "$HOME/Library/CloudStorage/OneDrive-Personal/Work/KFSP/Antigravity/Product"
│   3. claude  (chờ dấu nhắc >)
│   4. Paste: Tôi là phiên Terminal cho KFSP...
│   ✅ Đúng: dấu >, tiếng Việt, nhắc KFSP
│   ❌ Sai: "command not found" → npm install -g @anthropic-ai/claude-code
│
└── Khi Desktop nhắc "🔄 Sync → Terminal" → sang Terminal gõ: /kfsp:docs --update
```

## Enforcement
- Agent PHẢI output đầy đủ block 🖥️ MỞ TERMINAL, KHÔNG được tóm tắt
- Agent PHẢI nói rõ "thấy gì là đúng" và "thấy gì là sai"
- Agent PHẢI nói Terminal sau đó chạy lệnh gì, khi nào chạy
