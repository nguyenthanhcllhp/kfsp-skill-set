# KFSP + GSD + Ralph — Orchestration Guide v1.0

> **Tài liệu BẮT BUỘC.** Agent PHẢI đọc file này khi bắt đầu session mới trên BẤT KỲ project nào có cài KFSP Skill Set. File này mô tả cách 3 hệ thống phối hợp để đảm bảo project hoàn thành đúng, đủ, an toàn.

---

## 1. Tổng Quan — 3 Hệ Thống

| Hệ thống | Vai trò | Cài ở đâu | Check có sẵn |
|-----------|---------|------------|--------------|
| **KFSP Skill Set** | 🛡️ Bảo vệ + Kiểm soát chất lượng | `~/.claude/commands/kfsp/` + `~/.claude/agents/` | `ls ~/.claude/commands/kfsp/ \| wc -l` → ≥17 |
| **GSD (Get Shit Done)** | 📋 Lên kế hoạch + Thực thi + Kiểm tra | Global Claude skill | `/gsd:help` → có output |
| **Ralph Loop** | 🔄 Vòng lặp tự động cho task phức tạp | Global CLI tool | `which ralph` hoặc `ralph --help` → có output |

### Tại sao cần CẢ 3?

```
GSD = BỘ NÃO     → Biết phải làm GÌ, theo thứ tự NÀO, verify XONG chưa
Ralph = ĐÔI CHÂN → Chạy tự động, lặp đi lặp lại cho đến khi xong
KFSP = ÁO GIÁP   → Bảo vệ khỏi sai sót ở MỌI bước (5 tầng, 17 skills)
```

| Không có | Hậu quả |
|----------|---------|
| Không GSD | Code mù, không có plan, không verify kết quả |
| Không Ralph | Task phức tạp phải chạy thủ công, không AFK được |
| Không KFSP | Không guard, không kiểm tra regression, mất kiểm soát agent |
| Có cả 3 | ✅ Plan → Execute → Verify → Protect → Learn → Repeat |

---

## 2. Detect Availability — Agent PHẢI Kiểm Tra

### 2.1 Khi nào kiểm tra?
- **Session mới** (trong `/kfsp:session-start`)
- **Khi spawn agent mới** (agent con phải biết tools nào có sẵn)
- **Khi nhận task mới** (xác định workflow phù hợp)

### 2.2 Cách kiểm tra

```bash
# CHECK 1: KFSP Skill Set
KFSP_COMMANDS=$(ls ~/.claude/commands/kfsp/*.md 2>/dev/null | wc -l | tr -d ' ')
KFSP_AGENTS=$(ls ~/.claude/agents/kfsp-*.md 2>/dev/null | wc -l | tr -d ' ')
if [ "$KFSP_COMMANDS" -ge 17 ]; then
  echo "✅ KFSP Skill Set: $KFSP_COMMANDS commands, $KFSP_AGENTS agents"
else
  echo "❌ KFSP Skill Set: NOT INSTALLED ($KFSP_COMMANDS/17)"
  echo "   → Cài: git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git"
fi

# CHECK 2: GSD (Get Shit Done)
GSD_SKILL=$(ls ~/.claude/commands/gsd/*.md 2>/dev/null | wc -l | tr -d ' ')
GSD_PLANNING=$([ -f ".planning/PROJECT.md" ] && echo "true" || echo "false")
if [ "$GSD_SKILL" -gt 0 ] || [ "$GSD_PLANNING" = "true" ]; then
  echo "✅ GSD: installed ($GSD_SKILL commands)"
else
  echo "⚠️ GSD: NOT DETECTED → xem https://github.com/gsd-build/get-shit-done"
fi

# CHECK 3: Ralph Loop
if command -v ralph &>/dev/null; then
  echo "✅ Ralph: installed ($(which ralph))"
elif [ -f ".ralph/PROMPT.md" ]; then
  echo "✅ Ralph: project enabled (.ralph/ found)"
else
  echo "⚠️ Ralph: NOT DETECTED → xem https://github.com/frankbria/ralph-claude-code"
fi
```

### 2.3 Quyết định workflow dựa trên availability

| KFSP | GSD | Ralph | Workflow |
|------|-----|-------|----------|
| ✅ | ✅ | ✅ | **Full Orchestration** — dùng tất cả (khuyến nghị) |
| ✅ | ✅ | ❌ | **GSD + KFSP** — plan+execute có bảo vệ, không AFK |
| ✅ | ❌ | ✅ | **Ralph + KFSP** — lặp tự động có bảo vệ, không planning |
| ✅ | ❌ | ❌ | **KFSP Only** — bảo vệ chất lượng, agent tự quản lý workflow |
| ❌ | ✅ | ✅ | ⚠️ GSD+Ralph KHÔNG CÓ BẢO VỆ — cài KFSP trước |
| ❌ | ❌ | ❌ | ❌ Không có gì — agent chạy mù, rủi ro cao |

**Quy tắc:** KFSP là BẮT BUỘC. GSD và Ralph là KHUYẾN NGHỊ MẠNH.

---

## 3. Mapping — KFSP Skills × GSD Stages × Ralph Phases

### 3.1 Vòng đời Project hoàn chỉnh

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          PROJECT LIFECYCLE                                   │
│                                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ KHỞI TẠO │→ │ LÊN KẾ   │→ │ THỰC THI │→ │ KIỂM TRA │→ │ ĐÓNG     │     │
│  │          │  │ HOẠCH    │  │          │  │          │  │ PHASE    │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│       │              │              │              │              │           │
│  GSD: │         GSD: │         GSD: │         GSD: │         GSD: │           │
│  new- │         plan-│         exec-│         veri-│         comp-│           │
│  proj │         phase│         phase│         fy   │         lete │           │
│       │              │              │              │              │           │
│  KFSP:│         KFSP:│         KFSP:│         KFSP:│         KFSP:│           │
│  sess │         pre- │         sweep│         build│         post-│           │
│  start│         mort │         guard│         veri │         phase│           │
│       │         dev-j│         pre- │         test-│         sync │           │
│       │              │         comm │         brief│         guard│           │
│       │              │         ux-  │         bug- │         doc- │           │
│       │              │         pari │         log  │         pilot│           │
│       │              │              │              │              │           │
│  Ralph:             Ralph:    Ralph:                                          │
│  (N/A)              setup     --monitor                                      │
│                     enable    (loop until done)                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Chi tiết từng giai đoạn

#### 🔹 GIAI ĐOẠN 1: KHỞI TẠO (Init)

| Bước | Tool | Lệnh | Mục đích |
|------|------|-------|----------|
| 1.1 | KFSP | `/kfsp:session-start` | Check git, detect drift, update memory |
| 1.2 | GSD | `/gsd:new-project` | Nghiên cứu domain, tạo roadmap |
| 1.3 | KFSP | `/kfsp:dev-journal --init` | Khởi tạo nhật ký |
| 1.4 | KFSP | `/kfsp:doc-pilot --status` | Baseline tài liệu |

#### 🔹 GIAI ĐOẠN 2: LÊN KẾ HOẠCH (Planning)

| Bước | Tool | Lệnh | Mục đích |
|------|------|-------|----------|
| 2.1 | GSD | `/gsd:discuss-phase N` | Thảo luận cách triển khai |
| 2.2 | GSD | `/gsd:plan-phase N` | Tạo plan chi tiết (max 3 tasks/plan) |
| 2.3 | KFSP | `/kfsp:pre-mortem "Phase N: mô tả"` | Dự báo rủi ro 4 bề mặt |
| 2.4 | KFSP | `/kfsp:dev-journal --log decision` | Ghi decision |
| 2.5 | Ralph | `ralph-enable` / sửa `.ralph/PROMPT.md` | Chuẩn bị Ralph nếu dùng |

**KFSP pre-mortem → GSD plan enrichment:**
- Risks từ pre-mortem → thêm vào plan's considerations
- Test cases từ pre-mortem → thêm vào Master Test Registry
- Mitigation strategies → thêm vào plan's implementation notes

#### 🔹 GIAI ĐOẠN 3: THỰC THI (Execution)

**Mode A: GSD Execute** (khuyến nghị cho task chuẩn)
```
/gsd:execute-phase N
  ├── Task 1: code → KFSP: sweep + guard + ux-parity → pre-commit → commit
  ├── Task 2: code → (tương tự)
  └── Task 3: code → (tương tự)
```

**Mode B: Ralph Loop** (cho task phức tạp, cần lặp nhiều)
```
ralph --monitor
  ├── Loop 1: code → test → fail → KFSP: guard (safety gate)
  ├── Loop 2: fix → test → fail → KFSP: guard
  ├── Loop 3: fix → test → PASS ✅ → KFSP: sweep + pre-commit → commit
  └── DONE
```

**Mode C: Manual** (không GSD, không Ralph)
```
Agent code → /kfsp:sweep → /kfsp:ux-parity → /kfsp:pre-commit → commit → /kfsp:guard
```

#### 🔹 GIAI ĐOẠN 4: KIỂM TRA (Verification)

| Bước | Tool | Lệnh | Mục đích |
|------|------|-------|----------|
| 4.1 | KFSP | `/kfsp:build-verify` | Gate tự động: analyze + test |
| 4.2 | KFSP | `/kfsp:test-brief` | Tạo test checklist cho PM |
| 4.3 | GSD | `/gsd:verify-work N` | Kiểm tra deliverables vs plan |
| 4.4 | PM | test theo test-brief | PM test |
| 4.5 | KFSP | `/kfsp:bug-log --log/--resolve` | Log + track bugs |

#### 🔹 GIAI ĐOẠN 5: ĐÓNG PHASE (Close)

| Bước | Tool | Lệnh | Mục đích |
|------|------|-------|----------|
| 5.1 | KFSP | `/kfsp:post-phase N` | Docs + sync + regression |
| 5.2 | KFSP | `/kfsp:sync-check` | Đồng bộ source copies |
| 5.3 | KFSP | `/kfsp:guard` | Health score cuối |
| 5.4 | KFSP | `/kfsp:doc-pilot --what-changed` | Docs cần cập nhật |
| 5.5 | GSD | `/gsd:complete-milestone` | Archive + tag release |
| 5.6 | KFSP | `/kfsp:dev-journal --log phase-complete` | Ghi kết quả |

---

## 3.3 Spec → Build → Verify: 2 Luồng Song Song

> **Vấn đề cốt lõi:** Có spec, có build, nhưng KHÔNG CÓ AI so sánh kết quả với spec ban đầu.
> Skills về test có đầy đủ trong CLAUDE.md, nhưng thực tế test ít được tạo vì thiếu CẦU NỐI.

### Kiến trúc Test/QA — 2 Luồng Song Song:

```
                    📋 SPEC / PLAN
                   (Source of Truth)
                  ↙                ↘
         DEV STREAM               QA STREAM
         ┌──────────┐            ┌──────────────────┐
         │ Step 1:  │            │ Step 0:          │
    ┌────│ TDD —    │            │ Từ spec → sinh:  │
    │    │ viết unit│            │ • Expected tests │
    │    │ test     │            │ • Expected       │
    │    │ TRƯỚC    │            │   changelog      │
    │    └────┬─────┘            │ • Pass/fail      │
    │         ↓                  │   criteria       │
    │    ┌──────────┐            └────────┬─────────┘
    │    │ Step 2:  │                     │
    │    │ Code →   │                     │ (Lưu trữ —
    │    │ Test     │                     │  chờ build)
    │    │ PASS     │                     │
    │    └────┬─────┘                     │
    │         ↓                           │
    │    ┌──────────┐                     │
    │    │ Step 3:  │                     │
    │    │ Build +  │                     │
    │    │ Install  │                     │
    │    └────┬─────┘                     │
    │         ↓                           ↓
    │    ┌─────────────────────────────────────┐
    │    │        VERIFICATION GATE             │
    │    │                                      │
    │    │  build-verify:                       │
    │    │  ├── CHECK 1-16: Automated gates    │
    │    │  └── CHECK 17: Spec Compliance ←────│── So sánh actual vs expected
    │    │                                      │
    │    │  test-brief:                        │
    │    │  ├── Step 0: Expected tests (từ QA) │
    │    │  └── Step 1-5: Build → PM test      │
    │    │                                      │
    │    │  Spec Compliance Report:             │
    │    │  ├── X/Y deliverables completed     │
    │    │  ├── Unplanned work flagged          │
    │    │  └── Missing items listed            │
    │    └───────────────┬─────────────────────┘
    │                    │
    │              ┌─────┴──────┐
    │              │  PASS?     │
    │              └─────┬──────┘
    │              YES ↙     ↘ NO
    │         ┌──────────┐  ┌──────────────┐
    │         │ test-brief│  │ Bug Loop:    │
    │         │ → PM test │  │ bug-log →    │
    │         └──────────┘  │ fix → verify │
    │                       │ (max 3 loops)│
    └───────────────────────│ rồi back ↑   │
                            └──────────────┘
```

### Bug Loop Control (Build ⇄ Dev):
```
Build FAIL → bug-log --log (BUG-YYYYMMDD-NN)
    ↓
Dev fix → sweep → pre-commit → commit
    ↓
Build lại → build-verify
    ↓
Vẫn FAIL? → Loop lại (max 3 lần)
    ↓
Quá 3 lần → incident-review → escalate PM
```

### Áp dụng Cross-Domain:

| Domain | Spec | Dev Stream | QA Stream | Verify |
|--------|------|------------|-----------|--------|
| **Product** | Feature spec, GSD plan | TDD → code → build | Unit test, functional test | Screenshot, build-verify |
| **Marketing** | Campaign brief | Content → design → publish | Audience check, message check | Analytics, A/B results |
| **Sales** | Deal playbook | Qualify → propose → negotiate | Criteria check, pricing check | Win/loss review |
| **HR** | Job spec | Source → screen → interview | Skills check, culture check | Hiring decision review |

---

## 4. GSD ↔ KFSP Mapping Chi Tiết

### 4.1 GSD Commands → KFSP Skills (Auto-Trigger)

| GSD Command | KFSP Skills PHẢI chạy | Thời điểm |
|-------------|----------------------|-----------|
| `/gsd:new-project` | `session-start`, `dev-journal --init` | Trước khi GSD chạy |
| `/gsd:discuss-phase N` | `pre-mortem` (nếu chưa chạy) | Trong discuss |
| `/gsd:plan-phase N` | `pre-mortem` → output risks vào plan | Trước plan |
| `/gsd:execute-phase N` | `sweep` + `guard` (sau mỗi task), `pre-commit` (trước commit) | Trong execution |
| `/gsd:verify-work N` | `build-verify` + `test-brief` | Song song |
| `/gsd:complete-milestone` | `post-phase` + `sync-check` + `guard` + `doc-pilot` | Trước close |
| `/gsd:quick` | `sweep` + `pre-commit` (tối thiểu) | Sau code |
| `/gsd:debug` | `dev-journal --log incident` + `sweep` (sau fix) | Khi debug |

### 4.2 GSD Executor → KFSP Agent Spawn Rules

```
┌─────────────────────────────────────────────────────────────────┐
│  GSD EXECUTOR — KFSP AGENT SPAWN RULES                          │
├─────────────────────────────────────────────────────────────────┤
│  AFTER EVERY TASK:                                               │
│  ├── kfsp-sweep (blast radius)              ← BẮT BUỘC          │
│  └── kfsp-guard (health score)              ← BẮT BUỘC          │
│                                                                  │
│  IF TASK CHANGES UI:                                             │
│  └── kfsp-ux-parity (design match)          ← BẮT BUỘC          │
│                                                                  │
│  BEFORE EACH COMMIT:                                             │
│  └── kfsp-pre-commit (validation)           ← BẮT BUỘC (gate)   │
│                                                                  │
│  IF GUARD SCORE < 70:                                            │
│  └── STOP execution → kfsp-incident-review  ← AUTOMATIC          │
│                                                                  │
│  AFTER ALL TASKS DONE:                                           │
│  ├── kfsp-build-verify                      ← BẮT BUỘC          │
│  ├── kfsp-test-brief                        ← BẮT BUỘC          │
│  └── kfsp-doc-pilot --what-changed          ← BẮT BUỘC          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Ralph ↔ KFSP Mapping Chi Tiết

### 5.1 Ralph Lifecycle → KFSP Skills

| Ralph Event | KFSP Skills | Cách tích hợp |
|-------------|-------------|----------------|
| `ralph-enable` (init) | `session-start` | Chạy trước ralph-enable |
| `ralph --monitor` (start) | `sentinel --baseline` | Snapshot trước loop |
| Mỗi loop iteration | `guard --quick` | Safety gate — < 70 → break |
| Mỗi loop commit | `pre-commit` | Validation gate |
| Loop PASS | `sweep` + `ux-parity` | Verify kết quả |
| Ralph kết thúc | `build-verify` + `sentinel --compare` | Before/after |

### 5.2 Ralph Config — PHẢI include KFSP

#### `.ralph/AGENT.md`:
```markdown
## KFSP Safety Gates (BẮT BUỘC)
1. After each loop: /kfsp:guard --quick (score ≥ 70 to continue)
2. Before commit: /kfsp:pre-commit (PASS to commit)
3. After all loops: /kfsp:build-verify + /kfsp:sweep
```

### 5.3 Full Stack: Ralph + GSD + KFSP

```
/gsd:plan-phase N → PLAN.md
    ↓
ralph-enable → tasks from PLAN.md → .ralph/fix_plan.md
    ↓
/kfsp:sentinel --baseline
    ↓
ralph --monitor (loops with KFSP guard gates)
    ↓
/kfsp:sentinel --compare → /kfsp:build-verify → /kfsp:test-brief
    ↓
/gsd:verify-work N → /kfsp:post-phase N
```

---

## 6. Agent Spawn Rules — Ai Đọc Gì

### Tài liệu agent PHẢI đọc (theo priority)

| Priority | File | Khi nào đọc |
|----------|------|-------------|
| P0 | Project's `CLAUDE.md` | Luôn luôn |
| P1 | `ORCHESTRATION.md` (file này) | Session mới |
| P2 | `README.md` (skill set) | Lần đầu dùng KFSP |
| P3 | `.planning/PROJECT.md` (GSD) | Khi cần project context |
| P4 | `.ralph/PROMPT.md` (Ralph) | Khi chạy Ralph loop |
| P5 | Skill spec cụ thể | Khi chạy skill đó |

### Khi GSD executor spawn task agent — include:
```
KFSP RULES:
- After task: /kfsp:sweep on changed files
- Before commit: /kfsp:pre-commit
- If UI: /kfsp:ux-parity
- Log decisions: /kfsp:dev-journal --log decision
```

### Khi Ralph spawn loop agent — trong `.ralph/AGENT.md`:
```
POST-CHANGE CHECKS:
- /kfsp:guard --quick (≥ 70 to continue)
- /kfsp:pre-commit (before commit)
```

---

## 7. Enforcement — 10 Lớp Đảm Bảo Tuân Thủ

| # | Lớp | Kiểm tra gì | Tool |
|---|-----|-------------|------|
| 1 | **Session Start** | GSD/Ralph/KFSP đã cài? State? | `/kfsp:session-start` |
| 2 | **Pre-Mortem** | Risks → test cases → plan | `/kfsp:pre-mortem` |
| 3 | **GSD Plan** | Plan có KFSP checks? | `/gsd:plan-phase` |
| 4 | **Execution Guard** | Mỗi task → sweep + guard | `kfsp-guard` agent |
| 5 | **Pre-Commit Gate** | Validation trước commit | `/kfsp:pre-commit` |
| 6 | **Build Verify** | Analyze + test + coverage | `/kfsp:build-verify` |
| 7 | **Test Brief** | PM test checklist | `/kfsp:test-brief` |
| 8 | **Post-Phase** | Docs + sync + regression | `/kfsp:post-phase` |
| 9 | **Release Gate** | Score /100, Go/No-Go | `/kfsp:release-gate` |
| 10 | **Audit** | Milestone complete? | `/gsd:audit-milestone` |

### Enforcement Chain:
```
session-start → pre-mortem → plan-phase → execute (guard) → pre-commit
    → build-verify → test-brief → post-phase → release-gate → audit → dev-journal
```

---

## 8. Workflow Templates

### 8.1 New Project (Full Stack)
```bash
/kfsp:session-start → /gsd:new-project → /kfsp:dev-journal --init
/gsd:discuss-phase 1 → /kfsp:pre-mortem → /gsd:plan-phase 1
/gsd:execute-phase 1  # (auto: sweep, guard, pre-commit)
/kfsp:build-verify → /kfsp:test-brief → /gsd:verify-work 1
/kfsp:post-phase 1 → /kfsp:guard → /gsd:complete-milestone
```

### 8.2 Quick Task
```bash
/kfsp:session-start --quick → /gsd:quick → /kfsp:build-verify --quick
```

### 8.3 Complex Task (Ralph)
```bash
/kfsp:session-start → /gsd:plan-phase N → /kfsp:pre-mortem
/kfsp:sentinel --baseline → ralph --monitor → /kfsp:sentinel --compare
/kfsp:build-verify → /kfsp:test-brief → /gsd:verify-work N
```

### 8.4 Bug Fix
```bash
/kfsp:bug-log --list → /gsd:debug → fix → /kfsp:sweep → /kfsp:pre-commit
→ commit → /kfsp:bug-log --resolve → /kfsp:dev-journal --log bug-fix
```

### 8.5 Resume Work
```bash
/kfsp:session-start → /gsd:resume-work → continue
```

---

## 9. Fallback — Khi Thiếu Tool

| Thiếu | Hậu quả | Thay thế |
|-------|---------|----------|
| GSD | Không plan/verify | Agent tự tạo plan markdown, dùng build-verify thay verify-work |
| Ralph | Không AFK | GSD execute-phase, agent chạy liên tục |
| KFSP | ⚠️ NGUY HIỂM | Không guard/sweep/sentinel → cảnh báo user, gợi ý cài |

---

## 10. CLAUDE.md Template — Copy vào Project Mới

```markdown
## ⚠️ BẮT BUỘC: Orchestration Rules

### Tool Detection (Session Start)
Agent PHẢI kiểm tra 3 hệ thống khi bắt đầu session:
1. **KFSP Skill Set** — `ls ~/.claude/commands/kfsp/ | wc -l` ≥ 17
2. **GSD** — `/gsd:help` có output HOẶC `.planning/PROJECT.md` tồn tại
3. **Ralph** — `which ralph` HOẶC `.ralph/PROMPT.md` tồn tại

Nếu thiếu → thông báo user + gợi ý cài.

### Workflow Enforcement
- GSD plan PHẢI include KFSP pre-mortem output
- GSD execute PHẢI chạy KFSP sweep + guard sau mỗi task
- Ralph loop PHẢI chạy KFSP guard sau mỗi iteration
- Mỗi commit PHẢI qua KFSP pre-commit gate
- Mỗi build PHẢI qua KFSP build-verify + test-brief

### Tài liệu agent đọc
Đọc `ORCHESTRATION.md` trong kfsp-skill-set repo.
```

---

## 11. Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-03-15 | v1.0 | Initial release — KFSP + GSD + Ralph integration guide |

## 12. References

| Tài liệu | Vị trí |
|-----------|--------|
| KFSP README | `kfsp-skill-set/README.md` |
| KFSP INSTALL | `kfsp-skill-set/INSTALL.md` |
| GSD | [github.com/gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) |
| Ralph | [github.com/frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) |
| Testing Rules | `docs/15_TESTING_RULES.md` |
