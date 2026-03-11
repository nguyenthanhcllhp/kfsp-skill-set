---
name: kfsp:product-health
description: "Product Health Dashboard — PM's comprehensive view of all product dimensions: UX, tech, docs, business readiness, compliance, ops. Tier 5: Business Impact"
argument-hint: "[--dashboard] [--dimension <name>] [--gaps] [--template]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
---
<objective>
KFSP Product Health — Bảng điều khiển sức khỏe sản phẩm cho PM.

**Tầng:** 🎯 T5 Business Impact

**Dành cho PM không có background kỹ thuật:**
Thay vì hỏi "code OK chưa?" (đã có guard, sweep, pre-commit),
skill này hỏi: "SẢN PHẨM OK chưa? Sẵn sàng cho user/market/store chưa?"

**8 chiều sức khoẻ sản phẩm (Product Health Dimensions):**

```
┌─────────────────────────────────────────────────────┐
│                    PRODUCT HEALTH                    │
├──────────┬──────────┬──────────┬───────────────────┤
│ 1. UX    │ 2. TECH  │ 3. DOCS  │ 4. BUSINESS      │
│ Trải     │ Kỹ       │ Tài      │ Kinh doanh       │
│ nghiệm   │ thuật    │ liệu    │ sẵn sàng         │
├──────────┼──────────┼──────────┼───────────────────┤
│ 5. LEGAL │ 6. OPS   │ 7. DATA  │ 8. GROWTH        │
│ Pháp lý  │ Vận      │ Dữ liệu │ Tăng trưởng      │
│ tuân thủ │ hành     │ đo lường │ sẵn sàng         │
└──────────┴──────────┴──────────┴───────────────────┘
```

**Modes:**
- `--dashboard` → Overview 8 chiều, score mỗi chiều
- `--dimension <name>` → Deep dive 1 chiều cụ thể
- `--gaps` → Liệt kê tất cả gaps cần fill trước launch
- `--template` → Export template cho project mới
</objective>

<instructions>
## Parse Input
`$ARGUMENTS`:
- `--dashboard` → overview tất cả 8 chiều
- `--dimension <name>` → deep dive (ux, tech, docs, business, legal, ops, data, growth)
- `--gaps` → gap analysis tổng hợp
- `--template` → export blank template
- Rỗng → `--dashboard`

---

## 8 Product Health Dimensions

### 1. 🎨 UX (User Experience)
**Câu hỏi PM cần trả lời:**
- User mới có biết dùng app không? (onboarding effectiveness)
- Task phổ biến nhất hoàn thành trong bao nhiêu bước?
- Có screen nào user thường bỏ giữa chừng không?
- Error handling đủ rõ ràng?
- Accessibility cơ bản đạt không?

**Checklist:**
- [ ] Nielsen Heuristic score ≥ 20/30 cho mỗi core screen
- [ ] Core user flow ≤ 3 steps
- [ ] Loading/Error/Empty states cho mọi data screen
- [ ] Touch targets ≥ 44pt
- [ ] Text ≥ 14sp, contrast ≥ 4.5:1
- [ ] Onboarding completion > 80% (sau launch)

**Tool:** `/kfsp:ux-audit --full`

**Template fields:**
```
| Screen | Heuristic Score | Critical Issues | Last Audited |
|--------|----------------|-----------------|-------------|
```

---

### 2. ⚙️ TECH (Technical Health)
**Câu hỏi PM cần trả lời:**
- App có crash không? Build pass không?
- Performance có chấp nhận được? (launch < 3s, scroll smooth)
- Security có đủ? (no secrets in code, API keys safe)
- Dependencies có outdated/vulnerable không?

**Checklist:**
- [ ] Build pass (iOS + Android)
- [ ] Crash rate < 1% (sau launch)
- [ ] Cold start < 3 giây
- [ ] Scroll 60fps (no jank)
- [ ] No hardcoded secrets
- [ ] Dependencies updated (no critical vulnerabilities)
- [ ] Error tracking configured (Sentry/Crashlytics)

**Tools:** `/kfsp:guard`, `/kfsp:pre-commit`, `/kfsp:release-gate`

---

### 3. 📚 DOCS (Documentation)
**Câu hỏi PM cần trả lời:**
- User guide (HDSD) ready cho mọi feature?
- Developer docs đủ để onboard dev mới?
- API docs đầy đủ?
- CHANGELOG cập nhật?

**Checklist:**
- [ ] HDSD: mỗi feature có page + screenshot
- [ ] HDSD: FAQ cập nhật
- [ ] HDSD: Changelog page cập nhật
- [ ] Dev docs: architecture guide phản ánh code thực tế
- [ ] Dev docs: API spec đầy đủ
- [ ] Dev docs: build guide tested (ai đó build theo guide được)
- [ ] README: ai đọc cũng hiểu project là gì

**Tool:** `/kfsp:doc-pilot --status`, `/kfsp:doc-pilot --checklist`

---

### 4. 💼 BUSINESS (Business Readiness)
**Câu hỏi PM cần trả lời:**
- Value proposition rõ ràng? User biết app này làm gì trong 5 giây?
- Monetization model hoạt động? (Pro/Free gate, trial, pricing)
- Store listing ready? (screenshots, description, keywords)
- Competitor comparison: app mình có gì khác biệt?

**Checklist:**
- [ ] App Store/Play Store listing complete
  - [ ] App name, subtitle (30 chars)
  - [ ] Description (4000 chars max, keyword-rich)
  - [ ] Screenshots (6 per device size, highlight key features)
  - [ ] App preview video (optional nhưng +35% conversion)
  - [ ] Category + age rating
- [ ] Pricing/subscription configured
  - [ ] Free tier scope clear
  - [ ] Pro tier value clear
  - [ ] Trial period configured (7-day standard)
  - [ ] Paywall UX smooth
- [ ] Marketing ready
  - [ ] Landing page live
  - [ ] Social media accounts ready
  - [ ] Launch announcement drafted
- [ ] Support ready
  - [ ] Support email/chat configured
  - [ ] Response SLA defined (< 24h)
  - [ ] FAQ covers top 10 questions

**Template fields:**
```
| Item | Status | Owner | Deadline | Notes |
|------|--------|-------|----------|-------|
| Store listing | 🔲/🟡/✅ | | | |
| Screenshots | 🔲/🟡/✅ | | | |
```

---

### 5. ⚖️ LEGAL (Legal & Compliance)
**Câu hỏi PM cần trả lời:**
- Privacy policy có và đúng?
- Terms of service có?
- Data collection có khai báo đúng trong store?
- GDPR/local regulations tuân thủ?

**Checklist:**
- [ ] Privacy Policy
  - [ ] URL public và accessible
  - [ ] Liệt kê data collected (analytics, crash, user info)
  - [ ] Giải thích cách dùng data
  - [ ] Có opt-out option
  - [ ] Ngày cập nhật gần nhất
- [ ] Terms of Service
  - [ ] URL public
  - [ ] Limitation of liability
  - [ ] Subscription terms (auto-renew, cancellation)
- [ ] Store Compliance
  - [ ] App privacy details filled (iOS App Privacy)
  - [ ] Data safety section filled (Android)
  - [ ] Content rating accurate
  - [ ] Age rating appropriate
- [ ] Financial App Specific (stock/fintech)
  - [ ] Disclaimer: "không phải tư vấn đầu tư"
  - [ ] Data source attribution
  - [ ] Real-time vs delayed data disclosure
  - [ ] No guaranteed returns language

---

### 6. 🔧 OPS (Operations Readiness)
**Câu hỏi PM cần trả lời:**
- Ai xử lý khi app crash lúc 2h sáng?
- Monitoring/alerting có?
- Backup/restore plan?
- Update/rollback process?

**Checklist:**
- [ ] Monitoring
  - [ ] Crash reporting (Sentry/Crashlytics)
  - [ ] Performance monitoring (load time, API latency)
  - [ ] Uptime monitoring (cho backend)
  - [ ] Alert thresholds configured
- [ ] Incident Response
  - [ ] On-call rotation defined (dù chỉ 1 người)
  - [ ] Escalation path clear
  - [ ] Incident template ready (`/kfsp:incident-review`)
  - [ ] Communication template (user-facing status page/email)
- [ ] Release Process
  - [ ] Release checklist (`/kfsp:release-gate`)
  - [ ] Rollback plan
  - [ ] Staged rollout (1% → 10% → 50% → 100%)
  - [ ] Beta testing group
- [ ] Backup
  - [ ] User data backup strategy
  - [ ] Configuration backup
  - [ ] Recovery time objective (RTO) defined

---

### 7. 📊 DATA (Data & Measurement)
**Câu hỏi PM cần trả lời:**
- Biết bao nhiêu user dùng app? DAU/MAU?
- Biết user dùng feature nào nhiều nhất?
- Biết user bỏ (churn) ở đâu?
- Biết feature nào tạo value (conversion)?

**Checklist:**
- [ ] Analytics SDK configured
  - [ ] Screen view tracking
  - [ ] Event tracking (key actions: search, add watchlist, view FA)
  - [ ] User properties (Pro/Free, registration date)
  - [ ] Funnel tracking (onboarding → activation → retention)
- [ ] Key Metrics Defined
  - [ ] Acquisition: downloads, install rate
  - [ ] Activation: onboarding completion, first key action
  - [ ] Retention: D1, D7, D30 retention rate
  - [ ] Revenue: trial → paid conversion, ARPU, MRR
  - [ ] Referral: invite rate, organic vs paid
- [ ] Dashboard
  - [ ] Real-time user count
  - [ ] Daily/weekly report auto-generated
  - [ ] Alerts for anomalies (sudden drop in DAU)

**Template fields:**
```
| Metric | Definition | Target | Current | Source |
|--------|-----------|--------|---------|--------|
| DAU | | | | Firebase/Mixpanel |
| Onboarding completion | | >80% | | |
```

---

### 8. 🚀 GROWTH (Growth Readiness)
**Câu hỏi PM cần trả lời:**
- App sẵn sàng scale? (10x users → vẫn hoạt động?)
- Có referral/viral loop?
- SEO/ASO tối ưu?
- Content strategy cho retention?

**Checklist:**
- [ ] Scalability
  - [ ] API rate limits understood
  - [ ] Caching strategy (local + server)
  - [ ] CDN cho static assets
- [ ] Acquisition Channels
  - [ ] ASO (App Store Optimization) keywords
  - [ ] SEO (landing page, blog)
  - [ ] Social media content plan
  - [ ] Paid acquisition (budget, CPA target)
- [ ] Retention
  - [ ] Push notification strategy
  - [ ] Email drip campaign
  - [ ] In-app engagement features (streaks, achievements)
  - [ ] Content refresh cycle (market data tự cập nhật)
- [ ] Virality
  - [ ] Share feature (share stock analysis, chart)
  - [ ] Referral program
  - [ ] Social proof (reviews, ratings, community)

---

## Mode: --dashboard

Output overview:

```markdown
# 🏥 Product Health Dashboard
**Product:** [name]
**Date:** [timestamp]
**Auditor:** PM

## Health Score
| # | Dimension | Score | Status | Last Checked |
|---|-----------|-------|--------|-------------|
| 1 | 🎨 UX | X/10 | ✅/🟡/❌ | [date] |
| 2 | ⚙️ Tech | X/10 | ✅/🟡/❌ | [date] |
| 3 | 📚 Docs | X/10 | ✅/🟡/❌ | [date] |
| 4 | 💼 Business | X/10 | ✅/🟡/❌ | [date] |
| 5 | ⚖️ Legal | X/10 | ✅/🟡/❌ | [date] |
| 6 | 🔧 Ops | X/10 | ✅/🟡/❌ | [date] |
| 7 | 📊 Data | X/10 | ✅/🟡/❌ | [date] |
| 8 | 🚀 Growth | X/10 | ✅/🟡/❌ | [date] |

**Overall: XX / 80**
| Range | Rating |
|-------|--------|
| 65-80 | ✅ Launch Ready |
| 50-64 | 🟡 Almost — fix critical gaps |
| 35-49 | ⚠️ Significant gaps |
| <35   | 🛑 Not ready |

## Top 3 Gaps
1. [highest impact gap]
2. [second]
3. [third]

## Recommended Next Actions
1. [action] → `/kfsp:skill-to-run`
2. [action]
3. [action]
```

### Score Calculation per Dimension:
- Count checklist items completed vs total
- Score = (completed / total) × 10
- ✅ ≥ 8/10, 🟡 = 5-7/10, ❌ < 5/10

---

## Mode: --gaps

Aggregate ALL incomplete checklist items across 8 dimensions:

```markdown
# 🕳️ Product Health — Gap Analysis

## Critical Gaps (blocks launch)
| # | Dimension | Gap | Impact | Effort |
|---|-----------|-----|--------|--------|
| 1 | Legal | Privacy Policy missing | 🛑 Store reject | 2h |
| 2 | UX | No error states | 🛑 User frustration | 4h |

## Important Gaps (should fix before launch)
[table]

## Nice-to-Have Gaps (fix after launch)
[table]

## Estimated Total Effort: X hours
```

---

## Mode: --template

Export blank template cho bất kỳ project nào:

Xuất file `PRODUCT_HEALTH_TEMPLATE.md` với tất cả 8 dimension checklists,
các trường để fill, hướng dẫn cách chấm điểm.

Template fields cho mỗi dimension:
```markdown
### [Dimension Name]
**Owner:** [ai chịu trách nhiệm]
**Last Audited:** [ngày]
**Score:** ___ / 10

#### Checklist
- [ ] Item 1
- [ ] Item 2
...

#### Notes
[ghi chú riêng cho project này]

#### Action Items
| Item | Owner | Deadline | Status |
|------|-------|----------|--------|
```

---

## Cross-Skill Integration

| After running... | Also run... |
|-----------------|------------|
| `/kfsp:product-health --dashboard` | → Fix top 3 gaps |
| `/kfsp:product-health --dimension ux` | → `/kfsp:ux-audit --full` |
| `/kfsp:product-health --dimension tech` | → `/kfsp:guard --deep` |
| `/kfsp:product-health --dimension docs` | → `/kfsp:doc-pilot --checklist` |
| `/kfsp:release-gate` | → `/kfsp:product-health --gaps` |
| Quarterly review | → `/kfsp:product-health --dashboard` |
</instructions>
