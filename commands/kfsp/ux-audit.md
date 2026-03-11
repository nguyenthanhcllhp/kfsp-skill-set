---
name: kfsp:ux-audit
description: "UX Audit — comprehensive user experience evaluation using Nielsen heuristics, accessibility, performance perception, and domain-specific checks. Tier 4: User Experience"
argument-hint: "[screen-or-feature] [--heuristic] [--accessibility] [--flows] [--full] [--template]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP UX Audit — Đánh giá trải nghiệm người dùng toàn diện.

**Tầng:** 👤 T4 User Experience

**Thiết kế cho PM không có background UX/UI:**
Skill này KHÔNG yêu cầu kiến thức design. Nó cung cấp:
- Checklist có sẵn dựa trên 10 heuristic chuẩn quốc tế (Nielsen Norman)
- Template đánh giá cho TỪNG screen/feature
- Câu hỏi đơn giản mà PM có thể tự trả lời
- Score tự động — biết ngay đâu cần cải thiện

**Triết lý:** PM không cần biết design — PM cần biết ĐÁNH GIÁ design.
Giống như không cần là đầu bếp để biết món ăn ngon hay dở.

**Modes:**
- `--heuristic` → Đánh giá 10 Nielsen Heuristics cho screen/feature
- `--accessibility` → Kiểm tra accessibility (a11y) cơ bản
- `--flows` → Đánh giá user flows & task completion
- `--full` → Tất cả checks (heuristic + a11y + flows + domain)
- `--template` → Xuất template trống để PM tự đánh giá
- Mặc định → `--heuristic`
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `[screen-or-feature]` → tên screen/feature cần audit (vd: "dashboard", "onboarding", "FA detail")
- `--heuristic` → 10 Nielsen Heuristic evaluation
- `--accessibility` → Accessibility checks
- `--flows` → User flow analysis
- `--full` → Tất cả
- `--template` → Export blank template
- Rỗng → `--heuristic` cho toàn app

## Source Detection
```bash
for path in "/tmp/kfsp_flutter" "$HOME/Desktop/kfsp_flutter" "Product/source"; do
  [ -d "$path/lib" ] && SOURCE="$path" && break
done
```

---

## Mode: --heuristic [screen]

### 10 Nielsen Heuristics Evaluation

Với mỗi screen/feature, đánh giá 10 heuristic. Mỗi heuristic score 0-3:
- **3** = Tốt (đạt chuẩn, không cần sửa)
- **2** = Chấp nhận được (có vấn đề nhỏ)
- **1** = Cần cải thiện (có vấn đề rõ ràng)
- **0** = Vi phạm nghiêm trọng (cần fix ngay)

#### H1: Visibility of System Status
**Check:**
- [ ] Loading states: Có spinner/skeleton khi fetch data?
- [ ] Success feedback: User biết action thành công? (toast, animation, color change)
- [ ] Error feedback: User biết có lỗi? (error message, retry button)
- [ ] Progress indicators: Task nhiều bước có progress bar?
- [ ] Real-time status: Data có timestamp "cập nhật lúc..."?
- [ ] Empty states: Khi không có data, hiện gì? (không phải blank trắng)

```bash
# Auto-check: tìm loading/error states trong code
grep -rn "CircularProgressIndicator\|Shimmer\|skeleton\|loading" "$SOURCE/lib/features/$FEATURE/" 2>/dev/null | wc -l
grep -rn "SnackBar\|Toast\|AlertDialog\|showError" "$SOURCE/lib/features/$FEATURE/" 2>/dev/null | wc -l
grep -rn "EmptyState\|empty_state\|no_data" "$SOURCE/lib/features/$FEATURE/" 2>/dev/null | wc -l
```

#### H2: Match Between System and Real World
**Check:**
- [ ] Ngôn ngữ phù hợp user? (tiếng Việt cho KFSP, không dùng jargon kỹ thuật)
- [ ] Icons trực quan? (magnifying glass = search, bell = notification)
- [ ] Metaphor quen thuộc? (stock app: xanh=tăng, đỏ=giảm, vàng=tham chiếu)
- [ ] Số liệu format đúng? (1,285.43 thay vì 1285.43, % có dấu +/-)
- [ ] Thứ tự thông tin tự nhiên? (quan trọng nhất ở trên)

#### H3: User Control and Freedom
**Check:**
- [ ] Back navigation: Mọi screen đều quay lại được?
- [ ] Undo: Action quan trọng có thể undo? (xoá watchlist → undo)
- [ ] Cancel: Form/modal đều có nút đóng/cancel?
- [ ] Exit: User có thể thoát bất kỳ flow nào giữa chừng?
- [ ] Reset: Filter/search có nút xoá tất cả?

```bash
# Auto-check: back navigation
grep -rn "AppBar.*leading\|pop\|Navigator.pop\|context.pop\|GoRouter.*go" "$SOURCE/lib/features/$FEATURE/" 2>/dev/null | head -5
```

#### H4: Consistency and Standards
**Check:**
- [ ] Cùng action → cùng UI? (tất cả "Thêm" đều dùng FAB hoặc đều dùng button)
- [ ] Cùng data type → cùng format? (giá stock luôn 2 decimal)
- [ ] Cùng icon cho cùng concept? (watchlist icon nhất quán)
- [ ] Platform conventions? (iOS: back swipe, pull-to-refresh)
- [ ] Design tokens: Không hardcode colors/spacing?

```bash
# Auto-check: hardcoded values
grep -rn "Color(0x\|Colors\.\|EdgeInsets\.\|fontSize:" "$SOURCE/lib/features/$FEATURE/" 2>/dev/null | grep -v "kfsp\|theme\|KfspColors\|KfspSpacing" | head -10
```

#### H5: Error Prevention
**Check:**
- [ ] Destructive actions có confirm dialog? ("Xoá cổ phiếu khỏi watchlist?")
- [ ] Input validation trước submit? (OTP phải 6 số, email phải có @)
- [ ] Disable button khi không valid?
- [ ] Auto-save draft? (bài viết community, ghi chú)
- [ ] Prevent double-tap? (nút mua/bán không tap 2 lần)

#### H6: Recognition Rather Than Recall
**Check:**
- [ ] Recent searches hiện suggestion?
- [ ] Dropdown thay vì text input khi có options cố định?
- [ ] Breadcrumb/tab highlight cho biết đang ở đâu?
- [ ] Placeholder text gợi ý format? ("VD: FPT, VNM, HPG...")
- [ ] Default values hợp lý? (timeframe mặc định = 1D cho chart)

#### H7: Flexibility and Efficiency of Use
**Check:**
- [ ] Power user shortcuts? (swipe actions, long-press menus)
- [ ] Customizable? (sắp xếp watchlist, chọn tools nhanh)
- [ ] Search everywhere? (search bar tìm được mã, feature, setting)
- [ ] Quick actions? (từ notification → thẳng tới stock detail)
- [ ] Personalization? (remember last tab, last viewed stock)

#### H8: Aesthetic and Minimalist Design
**Check:**
- [ ] Mỗi element có mục đích? (không có element chỉ "cho đẹp")
- [ ] Information hierarchy rõ ràng? (title > subtitle > body)
- [ ] White space đủ? (không quá chật, không quá trống)
- [ ] Visual noise thấp? (không quá nhiều colors/icons/badges cùng lúc)
- [ ] Focus user attention? (CTA nổi bật, secondary actions nhỏ hơn)

#### H9: Help Users Recognize, Diagnose, and Recover from Errors
**Check:**
- [ ] Error message bằng ngôn ngữ user? ("Không tìm thấy mã" thay vì "404 Not Found")
- [ ] Error message gợi ý cách fix? ("Kiểm tra kết nối internet và thử lại")
- [ ] Retry button? (không bắt user tự navigate lại)
- [ ] Offline mode graceful? (hiện data cũ + badge "offline")
- [ ] Form error chỉ rõ field nào sai?

#### H10: Help and Documentation
**Check:**
- [ ] Onboarding cho user mới?
- [ ] Tooltip cho tính năng phức tạp? (P/E là gì? ROE là gì?)
- [ ] FAQ/Help dễ tìm?
- [ ] In-app guide cho features mới?
- [ ] Contact support dễ dàng?

### Output:
```markdown
# 🎨 UX Heuristic Audit: [Screen/Feature]
**Date:** [timestamp]
**Auditor:** PM (via /kfsp:ux-audit)

## Scores
| # | Heuristic | Score | Issues |
|---|-----------|-------|--------|
| H1 | Visibility of system status | X/3 | [brief] |
| H2 | Match real world | X/3 | |
| H3 | User control & freedom | X/3 | |
| H4 | Consistency & standards | X/3 | |
| H5 | Error prevention | X/3 | |
| H6 | Recognition > recall | X/3 | |
| H7 | Flexibility & efficiency | X/3 | |
| H8 | Aesthetic & minimal | X/3 | |
| H9 | Error recovery | X/3 | |
| H10 | Help & documentation | X/3 | |

**Total: XX / 30**
| Range | Rating |
|-------|--------|
| 25-30 | ✅ Excellent UX |
| 20-24 | 🟡 Good, minor issues |
| 15-19 | ⚠️ Needs improvement |
| <15   | 🛑 Significant UX problems |

## Critical Issues (Score 0-1)
[Detail each, with suggested fix]

## Improvement Opportunities (Score 2)
[Detail each]

## Domain-Specific Notes (Stock/Fintech)
- Real-time data freshness indicator?
- Number formatting consistency?
- Pro/Free feature gate UX smooth?
- Color = sentiment mapping correct? (xanh/đỏ/vàng)
```

---

## Mode: --accessibility

### Basic Accessibility Checks (no expertise needed)

| Check | What to Look For | How to Auto-Check |
|-------|-----------------|-------------------|
| **Color contrast** | Text readable trên background? Light text trên dark OK? | Grep for color combinations |
| **Touch targets** | Buttons/links ≥ 44x44pt? Không quá gần nhau? | Check widget sizes |
| **Font sizes** | Body text ≥ 14sp? Không có text quá nhỏ? | Grep fontSize values |
| **Color-blind safe** | Thông tin KHÔNG chỉ dùng màu? (icon + color cho stock up/down) | Check if color is sole indicator |
| **Screen reader** | Widgets có semantic labels? Images có alt text? | Grep for Semantics widget |
| **Dynamic type** | Text scale khi user tăng font size? | Check TextStyle scaleFactor |
| **Motion** | Animation có thể tắt? Không có flash/strobe? | Check animation usage |

```bash
# Auto-checks:
# 1. Font size check
grep -rn "fontSize:" "$SOURCE/lib/" 2>/dev/null | awk -F: '{print $NF}' | grep -oP '\d+' | sort -n | head -5
echo "--- Smallest font sizes above ---"

# 2. Semantics usage
grep -rn "Semantics\|semanticsLabel\|tooltip:" "$SOURCE/lib/features/" 2>/dev/null | wc -l
echo "--- Semantics widget count ---"

# 3. Touch target sizes (looking for small widgets)
grep -rn "SizedBox.*height.*[0-9]" "$SOURCE/lib/features/" 2>/dev/null | grep -oP 'height:\s*\K\d+' | sort -n | head -5
```

### Output:
```markdown
# ♿ Accessibility Audit: [Screen/Feature]

## WCAG Level A (Minimum — MUST pass)
| Check | Status | Detail |
|-------|--------|--------|
| Color contrast ≥ 4.5:1 | ✅/❌ | |
| Touch targets ≥ 44pt | ✅/❌ | |
| Text ≥ 14sp | ✅/❌ | |
| Non-color indicators | ✅/❌ | |

## WCAG Level AA (Recommended)
| Check | Status | Detail |
|-------|--------|--------|
| Screen reader labels | ✅/❌ | |
| Dynamic type support | ✅/❌ | |
| Focus order logical | ✅/❌ | |
| Motion reducible | ✅/❌ | |

## Score: X / 8 checks passed
```

---

## Mode: --flows

### User Flow Analysis

Đánh giá các luồng user chính — task nào user muốn làm, bao nhiêu bước, ở đâu có thể bỏ?

#### Template cho mỗi flow:
```markdown
### Flow: [Tên flow — VD: "Xem chi tiết cổ phiếu FPT"]

**User Goal:** [User muốn đạt được gì]
**Entry Points:** [User bắt đầu từ đâu — dashboard, search, watchlist, notification]
**Happy Path:**
1. [Bước 1] → [Screen/Action]
2. [Bước 2] → [Screen/Action]
3. [Bước 3] → Goal achieved ✅

**Steps Count:** X steps (target: ≤ 3 for frequent tasks)
**Possible Drop-offs:**
- Bước X: [Lý do user có thể bỏ — VD: "loading quá lâu", "không tìm thấy nút"]
**Edge Cases:**
- [Nếu network offline?]
- [Nếu data rỗng?]
- [Nếu user Free gặp Pro feature?]
**Error Recovery:**
- [Nếu lỗi ở bước X, user có thể quay lại bước X-1?]
```

#### Core Flows cần đánh giá (cho app stock):
1. **First-time user:** Install → Onboarding → Dashboard → Explore
2. **Check stock price:** Open app → Find stock → See price + chart
3. **Add to watchlist:** Find stock → Add → Verify in watchlist
4. **View FA detail:** Stock → FA screen → 13 tabs → Read analysis
5. **View chart:** Stock → Chart tab → Change timeframe → Draw
6. **Set alert:** Stock → Alert → Set conditions → Confirm
7. **Read news:** Dashboard → News → Read → Back
8. **Pro upgrade:** Free user → Hit gate → See upgrade → Purchase

### Output:
```markdown
# 🔄 User Flow Audit

## Flow Summary
| Flow | Steps | Target | Status | Drop-off Risk |
|------|-------|--------|--------|---------------|
| First-time user | X | ≤5 | ✅/⚠️ | |
| Check stock price | X | ≤3 | ✅/⚠️ | |
| [etc.] | | | | |

## Critical Flows (≤3 steps target)
[Detail each flow that exceeds target]

## Drop-off Risks
[Where users most likely abandon]

## Missing Flows
[Flows that should exist but don't have clear path]
```

---

## Mode: --template

Export blank templates cho PM tự đánh giá (không cần chạy code):

```markdown
# 📋 UX Evaluation Template — [Product Name]

## A. Screen Inventory
Liệt kê TẤT CẢ screens trong app:

| # | Screen | Entry Point | Key Actions | States |
|---|--------|-------------|-------------|--------|
| 1 | | | | Normal / Empty / Error / Loading |
| 2 | | | | |

## B. Heuristic Scorecard (per screen)
Copy bảng 10 heuristics ở trên, fill score 0-3 cho mỗi screen.

## C. User Flow Map
Vẽ/mô tả các flow chính:
- Flow 1: [Goal] → Step 1 → Step 2 → ... → Done
- Flow 2: ...

## D. Accessibility Checklist
- [ ] Tất cả text đọc được (không quá nhỏ, đủ contrast)
- [ ] Tất cả button/link đủ lớn để tap (≥44pt)
- [ ] Thông tin không chỉ truyền bằng màu sắc
- [ ] Loading/error states có ở mọi nơi fetch data
- [ ] App vẫn usable khi tăng font size

## E. Domain-Specific Checklist

### Stock/Fintech:
- [ ] Giá realtime có timestamp?
- [ ] Số format đúng (dấu phẩy, decimal)?
- [ ] Xanh=tăng, Đỏ=giảm nhất quán?
- [ ] Pro/Free gate không gây frustration?
- [ ] Sensitive data (portfolio) có bảo vệ?

### E-commerce:
- [ ] Checkout ≤ 3 bước?
- [ ] Trust signals (reviews, badges)?
- [ ] Cart persistent across sessions?

### SaaS:
- [ ] Onboarding < 2 phút?
- [ ] Empty states có hướng dẫn?
- [ ] Trial → Paid transition smooth?

### General (mọi app):
- [ ] Splash → usable < 3 giây?
- [ ] Offline có fallback?
- [ ] Push notification → đúng screen?
- [ ] Deep link hoạt động?
- [ ] Dark mode (nếu có) nhất quán?

## F. Competitive Benchmark (optional)
| Tiêu chí | App mình | Competitor 1 | Competitor 2 |
|-----------|----------|-------------|-------------|
| Onboarding steps | | | |
| Time to core action | | | |
| Feature discovery | | | |
```

---

## PM's UX Role Summary

Khi output, luôn nhắc PM:

```
💡 Nhắc nhở cho PM:
Bạn KHÔNG CẦN biết design để đánh giá UX. Vai trò của bạn:
1. DEFINE: Quyết định tiêu chuẩn (dùng checklist này)
2. SPECIFY: Viết rõ mỗi screen cần có gì (template E)
3. VALIDATE: Chấm điểm output (scorecard B)
4. MEASURE: Theo dõi user có dùng được không (sau khi launch)

Designer/Developer CẦN BẠN nói rõ: "Screen này thiếu loading state"
hơn là "Screen này xấu". Cụ thể = actionable.
```

## Auto-Integration

- Chạy `--heuristic` sau mỗi `/kfsp:ux-parity` → bổ sung góc nhìn user
- Chạy `--flows` trước `/kfsp:release-gate` → đảm bảo flow hoàn chỉnh
- Chạy `--template` khi bắt đầu project mới → baseline UX expectations
- Log kết quả vào Dev Journal: `/kfsp:dev-journal --log discovery`
</instructions>
