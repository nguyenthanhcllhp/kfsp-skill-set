# KFSP Skill Set — Huong Dan Van Hanh v1.0

**Single Source of Truth** — Tai lieu van hanh cho team du an. Dung cho onboarding, training, coaching.
**Cap nhat:** 20/03/2026 | **Phien ban:** 1.0 | **Chu so huu:** Thanh (PM)

> Moi thanh vien du an (PM, AI dev, QA, stakeholder) PHAI doc tai lieu nay truoc khi bat dau lam viec voi KFSP Skill Set. Moi thay doi quy trinh PHAI cap nhat tai lieu nay.

---

## MUC LUC

1. [Tong quan he thong](#1-tong-quan-he-thong)
2. [Cai dat & Xac nhan](#2-cai-dat--xac-nhan)
3. [Quy trinh van hanh theo 6 giai doan](#3-quy-trinh-van-hanh-theo-6-giai-doan)
4. [Bang trigger tu dong](#4-bang-trigger-tu-dong)
5. [Chuoi enforcement (8 layers)](#5-chuoi-enforcement-8-layers)
6. [Workflow mau](#6-workflow-mau)
7. [Quy trinh Test/QA](#7-quy-trinh-testqa)
8. [Docs & Changelog tu dong](#8-docs--changelog-tu-dong)
9. [Test Brief — Format HTML bat buoc](#9-test-brief--format-html-bat-buoc)
10. [Bug Flow](#10-bug-flow)
11. [Phoi hop GSD + Ralph](#11-phoi-hop-gsd--ralph)
12. [Ap dung cho tung nganh](#12-ap-dung-cho-tung-nganh)
13. [Case Study: Flutter Migration P2.5](#13-case-study-flutter-migration-p25)
14. [Known Issues & Roadmap](#14-known-issues--roadmap)
15. [Decision Log](#15-decision-log)
16. [Tai lieu lien quan](#16-tai-lieu-lien-quan)

---

## 1. Tong quan he thong

### 1.1 Muc tieu

KFSP Skill Set la bo 20 slash commands doc quyen, mo hinh 5 tang bao ve cho quy trinh phat trien san pham do AI agent thuc hien. He thong dam bao:

- **Chat luong code** khong bi suy giam qua nhieu phien lam viec
- **Docs dong bo** voi code tai moi thoi diem
- **PM kiem soat** duoc tien do, chat luong, rui ro
- **Agent khong lam tat** — moi buoc deu co gate kiem tra
- **Kien thuc duoc luu lai** — moi quyet dinh, su co, bai hoc deu ghi nhat ky

### 1.2 Pham vi

| Thanh phan | So luong | Mo ta |
|------------|----------|-------|
| Slash commands | 20 | `/kfsp:*` — chay trong Claude Code |
| Agent files | 15 | `~/.claude/agents/kfsp-*.md` — subprocess chuyen biet |
| Tang bao ve | 5 | T1 Safety → T2 Code → T3 QA → T4 UX → T5 Business |
| Be mat tuong tac | 4 | User-facing, Backend, Third-party, Internal |

### 1.3 Mo hinh 5 tang

```
┌─────────────────────────────────────────────────────────┐
│  T5 🎯 BUSINESS IMPACT                                  │
│  doc-pilot · pre-mortem · product-health                │
├─────────────────────────────────────────────────────────┤
│  T4 👤 USER EXPERIENCE                                   │
│  ux-parity · ux-audit · doc-pilot                       │
├─────────────────────────────────────────────────────────┤
│  T3 🧪 QUALITY ASSURANCE                                 │
│  surface-test · sentinel · release-gate                 │
│  test-brief · bug-log · build-verify                    │
├─────────────────────────────────────────────────────────┤
│  T2 🔧 CODE INTEGRITY                                    │
│  sweep · pre-commit · sync-check · post-phase           │
├─────────────────────────────────────────────────────────┤
│  T1 🛡️ AGENT SAFETY                                      │
│  guard · dev-journal · incident-review · session-start  │
└─────────────────────────────────────────────────────────┘
```

### 1.4 Nguoi dung

| Vai tro | Ai | Cach dung |
|---------|-----|----------|
| **PM / Owner** | Thanh | Nhan test-brief HTML, bam Dat/Loi, review build report |
| **AI Developer** | Claude Agent | Chay skills tu dong theo trigger, viet code, build |
| **QA** | Thanh + Agent | Agent chay tu dong (surface-test, sentinel), PM test thu cong |
| **Stakeholder** | Dev team (phudh3) | Nhan handoff package, doc specs |

### 1.5 Gia tri cot loi

```
1. KHONG LAM TAT    — Moi buoc co gate, khong duoc skip
2. KHONG TU BIEN MINH — Phat hien minh dang viet "gan dung" → DUNG LAI
3. DOCS = CODE      — Thay doi code ma khong cap nhat docs = vi pham
4. PM LA GATE CUOI  — Agent khong tu pass, phai gui PM verify
5. KIEN THUC LUU LAI — Moi decision → dev-journal, moi incident → review
```

---

## 2. Cai dat & Xac nhan

### 2.1 Cai dat

```bash
# Buoc 1: Clone repo
git clone https://github.com/nguyenthanhcllhp/kfsp-skill-set.git /tmp/kfsp-skill-set

# Buoc 2: Copy commands
mkdir -p ~/.claude/commands/kfsp/
cp /tmp/kfsp-skill-set/commands/kfsp/*.md ~/.claude/commands/kfsp/

# Buoc 3: Copy agents
mkdir -p ~/.claude/agents/
cp /tmp/kfsp-skill-set/agents/kfsp/*.md ~/.claude/agents/ 2>/dev/null

# Buoc 4: Xac nhan
echo "Commands: $(ls ~/.claude/commands/kfsp/*.md | wc -l | tr -d ' ')"
echo "Agents: $(ls ~/.claude/agents/kfsp-*.md 2>/dev/null | wc -l | tr -d ' ')"
```

### 2.2 Xac nhan thanh cong

| Kiem tra | Ky vong | Lenh |
|----------|---------|------|
| So commands | >= 20 | `ls ~/.claude/commands/kfsp/*.md \| wc -l` |
| So agents | >= 15 | `ls ~/.claude/agents/kfsp-*.md \| wc -l` |
| Help chay duoc | Co output | Go Claude Code, nhap `/kfsp:help` |
| Session start | Chay xong | `/kfsp:session-start --quick` |

### 2.3 Cau truc folder sau khi cai

```
~/.claude/
├── commands/
│   └── kfsp/
│       ├── help.md
│       ├── guard.md
│       ├── dev-journal.md
│       ├── incident-review.md
│       ├── session-start.md
│       ├── sweep.md
│       ├── pre-commit.md
│       ├── sync-check.md
│       ├── post-phase.md
│       ├── surface-test.md
│       ├── sentinel.md
│       ├── release-gate.md
│       ├── test-brief.md
│       ├── bug-log.md
│       ├── build-verify.md
│       ├── ux-parity.md
│       ├── ux-audit.md
│       ├── doc-pilot.md
│       ├── pre-mortem.md
│       └── product-health.md
└── agents/
    └── kfsp-*.md (15 files)
```

### 2.4 Cap nhat phien ban moi

```bash
cd /tmp/kfsp-skill-set && git pull
cp commands/kfsp/*.md ~/.claude/commands/kfsp/
cp agents/kfsp/*.md ~/.claude/agents/ 2>/dev/null
echo "✅ Cap nhat xong — $(date)"
```

---

## 3. Quy trinh van hanh theo 6 giai doan

### Tong quan 6 giai doan

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  START   │──▶│   PLAN   │──▶│   DEV    │──▶│ BUILD &  │──▶│  CLOSE   │──▶│   SHIP   │
│          │   │          │   │          │   │  TEST    │   │          │   │          │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
 session-      pre-mortem     sweep          build-verify    post-phase     release-gate
 start         dev-journal    ux-parity      test-brief      sync-check     surface-test
               pre-mortem     pre-commit     bug-log         doc-pilot      product-health
                                                             guard
```

---

### Giai doan 1: START — Khoi dong session

| Hang muc | Chi tiet |
|----------|---------|
| **Skill** | `/kfsp:session-start` |
| **Input** | Git repo, memory files, CLAUDE.md |
| **Output** | Bao cao drift (code vs docs vs memory), trang thai git |
| **Ai lam** | Agent (tu dong dau moi session) |
| **SLA** | < 2 phut |

**Quy trinh:**

```
1. Kiem tra git branch + status
2. Doc memory files → hieu state hien tai
3. So sanh code vs docs → phat hien drift
4. Bao cao cho PM: co gi thay doi, can fix gi
5. Hoi PM: co pull code moi khong?
6. Neu co drift → fix TRUOC khi lam task khac
```

**Lenh:**
```bash
/kfsp:session-start              # Full mode — check tat ca
/kfsp:session-start --quick      # Nhanh — chi check git + memory
/kfsp:session-start --skip-pull  # Khong hoi pull code
```

**Luu y:** Moi session MOI deu PHAI chay. Khong duoc bat dau code ma chua biet context.

---

### Giai doan 2: PLAN — Len ke hoach

| Hang muc | Chi tiet |
|----------|---------|
| **Skills** | `pre-mortem`, `dev-journal` |
| **Input** | Mo ta feature/task tu PM |
| **Output** | Bao cao rui ro, test cases, expected changelog, Project Charter |
| **Ai lam** | Agent (voi input tu PM) |
| **SLA** | < 10 phut cho task nho, < 30 phut cho phase |

**Quy trinh:**

```
1. PM mo ta task can lam
2. Agent chay /kfsp:pre-mortem "[mo ta]"
   → Du bao rui ro 4 be mat
   → Kiem tra 6 chieu completeness
   → Liet ke edge cases
3. Sinh test cases + expected changelog TRUOC khi code
4. /kfsp:dev-journal --log decision "[quyet dinh thiet ke]"
5. Neu la phase moi → tao Project Charter (6 chieu)
```

**Kiem tra 6 chieu (bat buoc):**

| # | Chieu | Cau hoi can tra loi |
|---|-------|-------------------|
| 1 | Animation | Transition gi? Loading ra sao? Feedback visual nhu the nao? |
| 2 | Action | User tap/swipe/navigate duoc nhung gi? |
| 3 | Logic | Business rules? Edge cases? Error handling? |
| 4 | Customer Insight | Feature giai quyet van de gi cho user? |
| 5 | Value | Gia tri gi? Pro vs Free khac nhau ra sao? |
| 6 | UX/UI Test | So sanh voi Figma/prototype nhu the nao? |

**Output mau — Project Charter:**

```markdown
# Phase 2.5 — Market Tab Polish
## Goals: Fix gradient, tooltip, dark mode cho 3 tabs
## Scope: Tab 1 (Thi Truong), Tab 2 (Heatmap), Tab 3 (Dong tien)
## Deliverables: 3 files sua, 1 build report, 1 test brief
## Success Criteria: 0 bug P1, UX parity > 90%, PM approve
## Stakeholders: PM (Thanh), Dev (Claude)
## Resources: Figma design, RN source reference
```

---

### Giai doan 3: DEV — Phat trien

| Hang muc | Chi tiet |
|----------|---------|
| **Skills** | `sweep`, `ux-parity`, `pre-commit` |
| **Input** | Plan tu giai doan 2, test cases |
| **Output** | Code da kiem tra, san sang commit |
| **Ai lam** | Agent |
| **SLA** | Tuy do phuc tap task |

**Quy trinh TDD bat buoc:**

```
Buoc 1: Viet test (expected behavior)
Buoc 2: Chay test → FAIL (chua co code)
Buoc 3: Viet code
Buoc 4: Chay test → PASS
Buoc 5: Refactor
Buoc 6: Chay test → van PASS
```

**Sau moi lan sua code:**

```bash
/kfsp:sweep [file_da_sua]        # Kiem tra blast radius
/kfsp:ux-parity [screen]         # So sanh voi design (neu sua UI)
```

**Truoc moi commit:**

```bash
/kfsp:pre-commit                 # Gate — KHONG commit neu FAIL
```

**Pre-commit kiem tra:**
- `flutter analyze` → 0 errors
- `flutter test` → all pass
- Tieng Viet co dau → tat ca user-facing text
- Docs da cap nhat chua
- Dev journal da ghi decision chua

---

### Giai doan 4: BUILD & TEST — Xay dung va kiem thu

| Hang muc | Chi tiet |
|----------|---------|
| **Skills** | `build-verify`, `test-brief`, `bug-log` |
| **Input** | Code da commit, spec goc |
| **Output** | Build thanh cong, test brief HTML, bao cao tu test |
| **Ai lam** | Agent build + PM test |
| **SLA** | Build < 15 phut, PM test < 30 phut |

**Quy trinh:**

```
┌──────────────────────────────────────────────────────────┐
│                    BUILD & TEST FLOW                      │
│                                                           │
│  1. Agent build + install len simulator                   │
│  2. /kfsp:build-verify (16 gates)                        │
│     ├── Code compile?                                     │
│     ├── Tests pass?                                       │
│     ├── Docs cap nhat?                                    │
│     ├── Dev journal co entries?                           │
│     ├── Charter 6/6?                                     │
│     ├── Spec compliance: actual vs expected               │
│     └── 10+ gates khac...                                │
│  3. YEU CAU THANH bam navigate tren simulator            │
│  4. Chup screenshot → so sanh vs design                   │
│  5. /kfsp:test-brief → sinh HTML cho PM                  │
│  6. Giao PM test (5-10 muc)                              │
│  7. PM bam Dat/Loi → /kfsp:bug-log                      │
│  8. Loop fix neu co bug                                   │
└──────────────────────────────────────────────────────────┘
```

**Bao cao tu test bat buoc khi gui build:**

```
📸 Bao cao tu test:
- [✅] Man hinh Thi Truong: Index cards hien thi dung — /tmp/test_01.png
- [✅] Man hinh Heatmap: Gradient 5 muc dung mau — /tmp/test_02.png
- [❌] Man hinh Dong tien: Tooltip bi cat — /tmp/test_03.png
- [🔲] Chua test: Dark mode — can Thanh bam navigate
```

---

### Giai doan 5: CLOSE — Dong phase

| Hang muc | Chi tiet |
|----------|---------|
| **Skills** | `post-phase`, `sync-check`, `doc-pilot`, `guard` |
| **Input** | Phase hoan thanh, tat ca bugs da fix |
| **Output** | Build report, docs cap nhat, source sync |
| **Ai lam** | Agent |
| **SLA** | < 30 phut |

**Quy trinh:**

```bash
/kfsp:post-phase 2.5                    # Checklist hoan thanh phase
/kfsp:sync-check --fix                   # Dong bo source copies
/kfsp:doc-pilot --what-changed           # Cap nhat docs
/kfsp:guard                              # Health score tong the
```

**Post-phase kiem tra:**
- [ ] Charter 6/6 chieu da verify
- [ ] Tat ca deliverables da giao
- [ ] Build report 9 sections day du
- [ ] Dev journal co entries cho phase
- [ ] Test results tu PM
- [ ] Source sync giua cac ban copy

---

### Giai doan 6: SHIP — Phat hanh

| Hang muc | Chi tiet |
|----------|---------|
| **Skills** | `release-gate`, `surface-test`, `doc-pilot`, `product-health` |
| **Input** | Phase da close |
| **Output** | Diem Go/No-Go, bao cao chat luong |
| **Ai lam** | Agent + PM quyet dinh |
| **SLA** | < 1 gio |

**Quy trinh:**

```bash
/kfsp:release-gate                       # 12 gates, diem /100, Go/No-Go
/kfsp:surface-test all                   # Test contracts 4 be mat
/kfsp:doc-pilot --checklist              # HDSD da san sang?
/kfsp:bug-log --stats                    # Con bug nao chua fix?
/kfsp:product-health                     # Dashboard 8 chieu
```

**Tieu chuan ship:**
- Release gate >= 80/100
- 0 bug P1, <= 2 bug P2
- Docs dong bo 100%
- PM da approve

---

## 4. Bang trigger tu dong

Agent PHAI tu dong chay skills theo bang nay. KHONG can PM nhac.

| Khi nao | Chay gi | Thu tu |
|---------|---------|--------|
| Bat dau session moi | `session-start` | Dau tien — TRUOC moi task |
| User pull code moi | `session-start` (full mode) | Ngay khi user thong bao |
| Truoc khi code (feature moi) | `pre-mortem` → `dev-journal --log decision` | 1→2 |
| Sau khi sua UI code | `ux-parity` | Ngay sau khi sua |
| Sau khi sua bat ky code nao | `sweep` | Ngay sau khi sua |
| Truoc khi commit | `pre-commit` | Gate — KHONG commit neu FAIL |
| Build + install xong | Yeu cau PM bam navigate → chup screenshot → `build-verify` → `test-brief` | Hoi PM → 1→2→3 |
| PM tim bug | `bug-log --log` → track → `bug-log --resolve` | Log → Fix → Resolve |
| Hoan thanh phase | `post-phase N` → `doc-pilot --what-changed` → `sync-check` → `guard` | 1→2→3→4 |
| Truoc release | `release-gate` → `surface-test all` → `doc-pilot --checklist` → `bug-log --stats` | 1→2→3→4 |
| Refactor lon | `sentinel --baseline` → refactor → `sentinel --compare` | Before → After |
| Quyet dinh ky thuat | `dev-journal --log decision` | Ngay lap tuc |
| Co su co | `incident-review` → `dev-journal --log incident` | 1→2 |
| Bat dau phase moi | `pre-mortem` → Tao Charter → `dev-journal --log phase-start` | 1→2→3 |
| Can push git | HOI PM: remote nao, branch nao → cho OK → push | LUON hoi truoc |
| Truoc khi dev feature moi | Viet unit test TRUOC → test FAIL → viet code → test PASS | Bat buoc TDD |
| Sau moi build | TU DONG giao deliverable cho PM | KHONG cho PM hoi |
| Dinh ky (tuan) | `guard --deep` → `doc-pilot --status` → `dev-journal --digest` → `bug-log --stats` | 1→2→3→4 |

---

## 5. Chuoi enforcement (8 layers)

He thong co 8 lop kiem tra, moi lop la 1 gate. Vi pham bat ky lop nao → KHONG duoc di tiep.

```
Layer 1: session-start
  │  Kiem tra: git status, drift code/docs, memory files
  │  Block: Bat dau lam viec khi chua biet context
  ▼
Layer 2: pre-mortem
  │  Kiem tra: rui ro 4 be mat, 6 chieu completeness
  │  Block: Code ma chua danh gia rui ro
  ▼
Layer 3: pre-commit
  │  Kiem tra: analyze, test, docs, diacritics
  │  Block: Commit code loi hoac thieu docs
  ▼
Layer 4: build-verify
  │  Kiem tra: 16 gates + spec compliance
  │  Block: Gui build cho PM khi chua du dieu kien
  ▼
Layer 5: test-brief
  │  Kiem tra: Sinh HTML test cho PM
  │  Block: PM test ma khong co checklist cu the
  ▼
Layer 6: post-phase
  │  Kiem tra: Charter 6/6, deliverables, journal entries
  │  Block: Close phase khi chua hoan thanh
  ▼
Layer 7: guard
  │  Kiem tra: Health score tong the, agent safety
  │  Block: Tiep tuc khi health < 60/100
  ▼
Layer 8: release-gate
  │  Kiem tra: 12 gates, Go/No-Go decision
  │  Block: Ship khi diem < 80/100
```

**Moi layer kiem tra cac layer truoc da chay chua:**
- `build-verify` kiem tra `pre-commit` da chay
- `post-phase` kiem tra `build-verify` + `dev-journal` co entries
- `release-gate` kiem tra `post-phase` + `surface-test`
- `guard` kiem tra tat ca cac layer khac

---

## 6. Workflow mau

### 6.1 New Project — Du an moi

```
session-start
     │
     ▼
gsd:new-project              ← GSD tao roadmap, chia phases
     │
     ▼
gsd:discuss-phase 1          ← Thao luan cach lam phase 1
     │
     ▼
gsd:plan-phase 1             ← Tao task plan (toi da 3 task/plan)
     │
     ▼
pre-mortem "[phase 1]"       ← Du bao rui ro
     │
     ▼
gsd:execute-phase 1          ← Thuc thi (fresh context moi task)
     │ (trong moi task: sweep, ux-parity, pre-commit)
     ▼
build-verify → test-brief    ← Gate + GUI PM test
     │
     ▼
gsd:verify-work 1            ← Kiem tra ket qua
     │
     ▼
post-phase 1 → guard         ← Dong phase
     │
     ▼
gsd:complete-milestone       ← Archive + tag release
```

### 6.2 Quick Task — Task nhanh

```
session-start --quick
     │
     ▼
gsd:quick "[mo ta task]"     ← Lam nhanh, khong can planning lon
     │ (agent tu sweep + pre-commit)
     ▼
build-verify --quick         ← Gate nhanh
     │
     ▼
Giao PM (neu co UI change)
```

### 6.3 Bug Fix — Sua loi

```
bug-log --list                ← Xem danh sach bugs chua fix
     │
     ▼
Chon bug → phan tich root cause
     │
     ▼
Fix code
     │
     ▼
sweep [file_da_sua]           ← Kiem tra blast radius
     │
     ▼
pre-commit                    ← Validate truoc commit
     │
     ▼
Commit + Build
     │
     ▼
bug-log --resolve BUG-YYYYMMDD-NN  ← Dong bug
     │
     ▼
Gui PM re-test (chi test bug da fix)
```

### 6.4 Resume — Tiep tuc tu session truoc

```
session-start                 ← Check drift, doc memory
     │
     ▼
(Neu co drift → fix truoc)
     │
     ▼
gsd:resume-work               ← Doc plan, tiep tuc tu cho cu
     │
     ▼
Tiep tuc dev...
```

---

## 7. Quy trinh Test/QA

### 7.1 Tong quan luong test

```
┌─────────────┐
│   SPEC      │  PM cung cap spec/Figma/prototype
└──────┬──────┘
       ▼
┌─────────────┐
│  SINH TEST  │  Agent sinh test cases + expected changelog TRUOC khi code
│  CASES      │  → Widget test cho UI, unit test cho logic
└──────┬──────┘
       ▼
┌─────────────┐
│    TDD      │  Viet test → FAIL → Code → PASS → Refactor → PASS
└──────┬──────┘
       ▼
┌─────────────┐
│   BUILD     │  build-verify (16 gates + spec compliance)
│   VERIFY    │  So sanh actual output vs expected (tu spec)
└──────┬──────┘
       ▼
┌─────────────┐
│  PM TEST    │  test-brief HTML → PM test 5-10 muc → Dat/Loi
└──────┬──────┘
       ▼
┌─────────────┐
│  BUG FIX    │  bug-log → AI fix → regression test → re-build
│  LOOP       │  → re-test targeted (chi muc loi)
└──────┬──────┘
       ▼
┌─────────────┐
│  LOOP       │  Bug ID bat buoc, toi da 3 loops
│  CONTROL    │  Qua 3 loops → incident-review
└─────────────┘
```

### 7.2 Spec → Test Cases (truoc khi code)

Khi nhan spec tu PM, agent PHAI:

1. **Doc spec ky** — Figma, prototype, mo ta PM
2. **Sinh test cases** — moi screen/feature co it nhat:
   - Happy path (hoat dong binh thuong)
   - Edge cases (null, empty, error, offline)
   - Visual parity (so voi Figma)
3. **Sinh expected changelog** — liet ke files se thay doi, sections se sua
4. **Ghi vao dev-journal** — `dev-journal --log decision "Test plan cho [feature]"`

**Vi du test case tu spec:**

```
SPEC: "Heatmap hien thi 38 co phieu, 9 nganh, 5 muc mau"

TEST CASES:
- TC01: Render 38 stocks voi du lieu mock → 38 o hien thi
- TC02: Tap vao stock → navigate Chart page voi dung stockCode
- TC03: Tap vao sector header → drill-down hien thi chi tiet
- TC04: Breadcrumb "< Tat ca" → quay lai toan canh
- TC05: Error state → hien thi "Loi ket noi, thu lai"
- TC06: Loading → shimmer animation
- TC07: Color gradient: 0~5% xanh sang, >=5% xanh dam
- TC08: Pro lock tren "Gia tri GD" va "Von hoa"
```

### 7.3 TDD — Test-Driven Development

```dart
// Buoc 1: Viet test TRUOC
testWidgets('Heatmap render 38 stocks', (tester) async {
  await tester.pumpWidget(MaterialApp(
    home: HeatmapScreen(data: mockHeatmapData),
  ));
  expect(find.byType(EChartsWebView), findsOneWidget);
});

// Buoc 2: Chay → FAIL (chua co HeatmapScreen)
// Buoc 3: Viet code HeatmapScreen
// Buoc 4: Chay → PASS
// Buoc 5: Refactor
// Buoc 6: Chay → van PASS
```

### 7.4 Build Verify — 16 Gates

| # | Gate | Mo ta | Auto/Manual |
|---|------|-------|-------------|
| 1 | Compile | Code compile thanh cong | Auto |
| 2 | Analyze | `flutter analyze` → 0 errors | Auto |
| 3 | Unit tests | `flutter test` → all pass | Auto |
| 4 | Diacritics | Tieng Viet co dau | Auto scan |
| 5 | Docs sync | Docs da cap nhat | Auto check |
| 6 | Dev journal | Co entries cho phase | Auto check |
| 7 | Charter | 6/6 chieu day du | Auto check |
| 8 | Number format | Dau `.` thap phan, `,` phan nghin | Auto scan |
| 9 | Git status | Clean, dung branch | Auto |
| 10 | Source sync | Cac ban copy dong bo | Auto |
| 11 | Spec compliance | Actual vs expected changelog | Semi-auto |
| 12 | Screenshot | So sanh vs design | Manual (PM bam) |
| 13 | Animation | Co transition/loading/feedback | Manual review |
| 14 | Action | Moi element co handler | Manual review |
| 15 | Error handling | Loading, error, empty states | Manual review |
| 16 | Pro/Free | Phan biet dung | Manual review |

### 7.5 PM Test

PM nhan test-brief HTML → mo tren trinh duyet → bam tung muc:

- **Dat** (xanh) — Hoat dong dung
- **Loi** (do) — Co van de → form bug tu mo
- **Bo qua** (xam) — Khong lien quan
- **Hen lai** (vang) — Can context (VD: ngoai gio giao dich)

### 7.6 Loop Control

| Lan | Hanh dong |
|-----|----------|
| Loop 1 | PM bao loi → Agent fix → re-build → PM re-test |
| Loop 2 | Van loi → Agent phan tich ky hon → fix → re-test |
| Loop 3 | Van loi → `incident-review` → root cause analysis |
| > 3 | DUNG LAI — bao cao PM, can quyet dinh: redesign? skip? |

---

## 8. Docs & Changelog tu dong

### 8.1 Cac loai docs tu dong

| Loai | Sinh boi | Khi nao | File |
|------|---------|---------|------|
| **Changelog** | `build-verify` | Moi build | Git diff → summary thay doi |
| **Test brief** | `test-brief` | Moi build | `docs/test_cases/*.html` |
| **Build report** | `post-phase` | Moi phase | `docs/build_reports/YYYY-MM-DD_P{N}.md` |
| **Feature specs** | `doc-pilot` | Khi feature moi | `docs/04_FEATURE_SPECS.md` |
| **Debug log** | Agent | Khi co incident | `docs/06_DEBUG_LOG.md` |
| **Handoff status** | Agent | Moi build | `docs/08_HANDOFF_STATUS.md` |

### 8.2 Build Report — 9 sections bat buoc

```markdown
# Build Report — P2.5 Task 3

## 1. Goals (tu Charter)
Fix gradient, tooltip, dark mode cho Market tabs

## 2. Deliverables
- [x] market_tab_thi_truong.dart — Fix gradient
- [x] heatmap_chart.dart — Tooltip enhancement
- [ ] dark_mode_market.dart — In progress

## 3. Success Criteria
- 0 bug P1: ✅ dat
- UX parity > 90%: ✅ dat (95%)
- PM approve: 🔲 cho test

## 4. Files Changed
- lib/screens/market/market_tab_thi_truong.dart (+45, -12)
- lib/widgets/heatmap/heatmap_chart.dart (+23, -8)

## 5. Technical Decisions
- Dung ECharts `visualMap.pieces` thay vi `inRange` cho 5 muc mau
- Ly do: cho phep custom chinh xac tung doan, RN cung dung cach nay

## 6. Test Checklist cho PM
→ Xem test-brief HTML: docs/test_cases/P2.5_T3_test.html

## 7. Skills da chay
- sweep: 3 files anh huong, 0 breaking
- ux-parity: 95% match voi Figma
- pre-commit: PASS (0 errors, 12 tests pass)
- build-verify: PASS (16/16 gates)

## 8. Known Issues
- BUG-20260320-01: Tooltip bi cat khi stock name > 15 ky tu
  Severity: P3 | Status: Logged

## 9. Next Steps
- Fix BUG-20260320-01
- Test dark mode toan dien
- Chuan bi P3 (PriceBoard)
```

### 8.3 Changelog tu dong (git diff)

Agent tu dong tao changelog tu git diff moi build:

```
## Thay doi — Build 2026-03-20

### Them moi
- Tooltip enhanced voi don vi (trieu, ty)
- Flash animation cho price update

### Sua
- Gradient 9 muc → 5 muc (don gian hoa)
- Sector header h24/fs11 (lon hon de bam)

### Xoa
- Onboarding popup tam (se redesign)

### Files
- market_tab_thi_truong.dart (+45, -12)
- heatmap_chart.dart (+23, -8)
- heat_map_model.dart (+5, -15)
```

---

## 9. Test Brief — Format HTML bat buoc

### 9.1 Yeu cau giao dien

| Thuoc tinh | Yeu cau |
|------------|---------|
| Giao dien | SANG (nen `#F6F6F8`), KHONG dark mode |
| Ngon ngu | 100% tieng Viet CO DAU |
| Nut kiem tra | Dat (xanh) / Loi (do) / Bo qua (xam) / Hen lai (vang) |
| Tooltip | Hover vao nut → ghi chu huong dan |
| Dinh kem | Hinh + video minh hoa cho moi muc test |
| Luu tru | localStorage tu dong — dong mo trinh duyet van con |
| Form bug | Tich hop — tu tao ma `BUG-YYYYMMDD-NN` |
| Xuat | Nut "Sao chep Markdown" de paste vao chat |

### 9.2 Cau truc HTML

```html
<!-- Moi muc test co dang: -->
<div class="test-item" data-id="TC01">
  <div class="test-header">
    <span class="test-id">TC01</span>
    <span class="test-title">Heatmap hien thi 38 co phieu</span>
  </div>
  <div class="test-body">
    <p>Mo tab Ban Do Nhiet → kiem tra so luong o hien thi</p>
    <img src="screenshot_heatmap.png" alt="Expected">
  </div>
  <div class="test-actions">
    <button class="btn-pass">Dat</button>
    <button class="btn-fail">Loi</button>
    <button class="btn-skip">Bo qua</button>
    <button class="btn-defer">Hen lai</button>
  </div>
</div>
```

### 9.3 Cach chay preview

```bash
# Config trong .claude/launch.json:
{
  "version": "0.0.1",
  "configurations": [{
    "name": "test-brief",
    "runtimeExecutable": "npx",
    "runtimeArgs": ["serve", "-l", "3002"],
    "port": 3002
  }]
}

# Agent chay:
# preview_start "test-brief"
# PM mo http://localhost:3002/[ten_file].html
```

### 9.4 Du lieu luu

Test brief luu trang thai vao `localStorage`:
- Key: `kfsp-test-brief-[build-id]`
- Value: JSON chua trang thai moi muc (pass/fail/skip/defer)
- Tu dong luu khi PM bam bat ky nut nao
- Khoi phuc khi mo lai trang

---

## 10. Bug Flow

### 10.1 Luong xu ly bug toan trinh

```
PM bam "Loi" tren test-brief
        │
        ▼
┌─────────────────────────┐
│  Form bug tu mo          │
│  - Mo ta loi             │
│  - Muc do (P1/P2/P3)    │
│  - Screenshot (dinh kem) │
│  - Ma tu tao:            │
│    BUG-YYYYMMDD-NN       │
└───────────┬─────────────┘
            ▼
Agent nhan bug (tu test-brief hoac chat)
        │
        ▼
/kfsp:bug-log --log "BUG-20260320-01: Tooltip bi cat"
        │
        ▼
Agent phan tich → fix code
        │
        ▼
/kfsp:sweep [file] → /kfsp:pre-commit
        │
        ▼
Viet regression test cho bug
        │
        ▼
Re-build → /kfsp:build-verify
        │
        ▼
Test-brief cap nhat → chi hien thi muc loi
        │
        ▼
PM re-test targeted
        │
        ▼
Dat → /kfsp:bug-log --resolve BUG-20260320-01
        │
        ▼
Loi → Quay lai fix (loop 2)
```

### 10.2 Lenh bug-log

```bash
/kfsp:bug-log --log "Mo ta bug" --severity P1   # Log bug moi
/kfsp:bug-log --list                              # Xem danh sach bugs
/kfsp:bug-log --list --open                       # Chi xem bugs chua fix
/kfsp:bug-log --resolve BUG-20260320-01           # Dong bug
/kfsp:bug-log --stats                             # Thong ke: bao nhieu open/closed
```

### 10.3 Muc do nghiem trong

| Muc do | Mo ta | SLA fix |
|--------|-------|---------|
| **P1 Critical** | App crash, mat du lieu, sai tinh toan tai chinh | < 2 gio |
| **P2 Major** | Chuc nang chinh khong hoat dong, UX sai lon | < 1 ngay |
| **P3 Minor** | Loi giao dien nho, text sai, color khong dung | < 3 ngay |
| **P4 Trivial** | Goi y cai tien, khong anh huong hoat dong | Backlog |

### 10.4 Loop control

- **Bug ID bat buoc** — moi bug phai co ma `BUG-YYYYMMDD-NN`
- **Toi da 3 loops** — fix → re-test → fix → re-test → fix → re-test
- **Qua 3 loops** → `incident-review` — phan tich root cause, co the can redesign
- **Regression test** — moi bug fix PHAI co test moi de dam bao khong tai dien

---

## 11. Phoi hop GSD + Ralph

### 11.1 Ba he thong

```
GSD = BO NAO      → Biet phai lam GI, theo thu tu NAO, verify XONG chua
Ralph = DOI CHAN   → Chay tu dong, lap di lap lai cho den khi xong
KFSP = AO GIAP    → Bao ve khoi sai sot o MOI buoc (5 tang, 20 skills)
```

### 11.2 Phat hien availability

```bash
# KFSP
ls ~/.claude/commands/kfsp/*.md | wc -l    # >= 20

# GSD
ls ~/.claude/commands/gsd/*.md | wc -l     # > 0
# hoac: co file .planning/PROJECT.md

# Ralph
which ralph                                # co output
# hoac: co folder .ralph/
```

### 11.3 Chon workflow mode

| KFSP | GSD | Ralph | Workflow |
|------|-----|-------|----------|
| Co | Co | Co | **Full Orchestration** — day du nhat (khuyen nghi) |
| Co | Co | Khong | **GSD + KFSP** — plan+execute co bao ve, khong AFK |
| Co | Khong | Co | **Ralph + KFSP** — lap tu dong co bao ve |
| Co | Khong | Khong | **KFSP Only** — bao ve chat luong, agent tu quan ly |
| Khong | Co | Co | ⚠️ KHONG CO BAO VE — cai KFSP truoc |

### 11.4 Spawn rules cho agent con

Khi GSD hoac Ralph spawn agent con (subprocess):
1. Agent con PHAI kiem tra KFSP co san khong
2. Agent con PHAI chay `sweep` sau khi sua code
3. Agent con PHAI chay `pre-commit` truoc khi commit
4. Agent con KHONG duoc chay `release-gate` (chi main agent)

### 11.5 AFK Mode voi Ralph

Khi PM muon de may tu chay (VD: qua dem):

```bash
# Chuan bi
ralph-enable                    # Kich hoat Ralph cho project
# Sua .ralph/PROMPT.md → paste task tu GSD plan
# Sua .ralph/fix_plan.md → danh sach task cu the

# Chay
ralph --monitor                 # Loop tu dong + dashboard theo doi

# KFSP skills tu dong chay trong moi loop:
# - sweep sau moi thay doi
# - pre-commit truoc moi commit
# - guard cuoi moi loop
```

---

## 12. Ap dung cho tung nganh

### 12.1 Bang dich thuat khai niem

| Khai niem goc (Product) | Marketing | Sales | HR |
|--------------------------|-----------|-------|-----|
| Feature | Chien dich (Campaign) | Deal / Pipeline stage | Vi tri tuyen dung |
| Sprint | Flight (dot chay) | Quarter review | Dot phong van |
| Bug | Loi content/link chet | Objection / Lost deal | Ung vien rut |
| Build | Ban content/asset | Proposal/Quotation | JD + Evaluation form |
| Test brief | Checklist review content | Review pipeline | Danh gia ung vien |
| Release | Launch | Closing | Offer letter |
| User (end-user) | Doi tuong muc tieu | Khach hang | Ung vien |
| Figma/Prototype | Mockup content | Slide deck | Template phong van |

### 12.2 Vi du: Marketing Campaign

```
┌─────────────────────────────────────────────────────────────┐
│  MARKETING — Chien dich Tet 2026                            │
│                                                              │
│  START:    session-start → doc memory, chien dich truoc       │
│  PLAN:    pre-mortem "Chien dich Tet 2026"                   │
│            → Rui ro: budget over, content delay, sai KV       │
│            → 6 chieu: Visual/CTA/Logic/Insight/Value/Test    │
│  DEV:     Tao content → sweep (kiem tra brand consistency)    │
│            → ux-parity (so voi brand guideline)               │
│  TEST:    build-verify → test-brief cho Marketing Manager    │
│            → Kiem tra: dung KV, dung CTA, link hoat dong      │
│  CLOSE:   post-phase → doc-pilot (cap nhat campaign docs)    │
│  SHIP:    release-gate → Go/No-Go launch                     │
└─────────────────────────────────────────────────────────────┘
```

### 12.3 Vi du: Sales Pipeline

```
START:   session-start → xem pipeline hien tai
PLAN:    pre-mortem "Deal ABC Corp Q2" → rui ro mat deal
DEV:     Tao proposal → sweep (so sanh voi template chuan)
TEST:    build-verify → checklist review proposal
         → Dung gia? Dung scope? Terms ro rang?
CLOSE:   post-phase → cap nhat CRM
SHIP:    Gui proposal → follow up
```

### 12.4 Vi du: HR Hiring

```
START:   session-start → xem vi tri dang tuyen
PLAN:    pre-mortem "Tuyen Flutter Dev" → rui ro: it ung vien, luong cao
DEV:     Tao JD + Evaluation form → sweep (kiem tra tieu chi nhat quan)
TEST:    build-verify → HR Manager review JD
         → Dung yeu cau? Luong canh tranh? JD hap dan?
CLOSE:   post-phase → dang tuyen, cap nhat tracker
SHIP:    Launch recruitment campaign
```

---

## 13. Case Study: Flutter Migration P2.5

### 13.1 Boi canh

- **Du an:** KFSP Flutter Migration — Phase 2.5 (Market Tab Polish)
- **Muc tieu:** Fix gradient heatmap, tang cuong tooltip, polish dark mode
- **Thoi gian:** 1 ngay (20/03/2026)

### 13.2 Quy trinh thuc te

**Buoc 1 — START:**
```
/kfsp:session-start
→ Branch: thanh_sg
→ Last commit: 1c365ba (16/03)
→ Drift: docs 08_HANDOFF_STATUS.md lech voi code (159 files vs ghi 102)
→ Fix drift truoc khi bat dau
```

**Buoc 2 — PLAN:**
```
/kfsp:pre-mortem "Fix gradient 5 muc, tooltip enhancements, dark mode"
→ Rui ro 1: Gradient moi co the lam ECharts render cham
→ Rui ro 2: Dark mode chua test toan dien → co the bi trang/den
→ 6 chieu: ✅ tat ca applicable
→ Test cases: 8 TCs sinh tu spec
→ Expected changelog: 3 files (heat_map_model, heatmap_chart, market_tab)

/kfsp:dev-journal --log decision "Dung visualMap.pieces thay vi inRange"
```

**Buoc 3 — DEV:**
```
# Viet test truoc
testWidgets('Heatmap 5 color levels render correctly', ...);

# Code
- heat_map_model.dart: 9 muc → 5 muc
  - 0~5%: mau sang goc (#09DA64/#F23468)
  - >=5%: mau dam (#069946/#D93360)
- heatmap_chart.dart: tooltip them don vi (trieu/ty)
- market_tab_thi_truong.dart: dark mode fixes

# Kiem tra
/kfsp:sweep heat_map_model.dart
→ 3 files anh huong, 0 breaking changes

/kfsp:ux-parity heatmap
→ 95% match voi Figma (5% la onboarding chua lam)

/kfsp:pre-commit
→ PASS: 0 errors, 15 tests pass, docs OK
```

**Buoc 4 — BUILD & TEST:**
```
# Build
rsync → Desktop → flutter build ios → install simulator

# Gate
/kfsp:build-verify
→ 16/16 gates PASS
→ Spec compliance: 3/3 files thay doi dung expected

# Yeu cau PM test
"Anh bam vao tab Ban Do Nhiet giup em"
→ Thanh bam → Agent chup screenshot → verify

# Test brief
/kfsp:test-brief
→ Sinh P25_T3_test.html (8 muc test)
→ PM test: 7 Dat, 1 Loi

# Bug
BUG-20260320-01: Tooltip bi cat khi stock name > 15 ky tu
Severity: P3
/kfsp:bug-log --log "BUG-20260320-01: Tooltip overflow"
```

**Buoc 5 — Fix bug loop:**
```
# Loop 1
Fix: them maxWidth + ellipsis cho tooltip text
Regression test: testWidgets('Tooltip truncate long names', ...);
Re-build → /kfsp:build-verify → PM re-test
→ PM: Dat ✅

/kfsp:bug-log --resolve BUG-20260320-01
```

**Buoc 6 — CLOSE:**
```
/kfsp:post-phase 2.5
→ Charter 6/6: ✅
→ Deliverables: 3/3 files
→ Dev journal: 2 entries (1 decision, 1 bug-fix)

/kfsp:sync-check --fix
→ Source sync: current_source ↔ Desktop ✅

/kfsp:doc-pilot --what-changed
→ Cap nhat: 08_HANDOFF_STATUS.md, 04_FEATURE_SPECS.md

/kfsp:guard
→ Health: 88/100
```

### 13.3 Ket qua

| Metric | Gia tri |
|--------|---------|
| Thoi gian | 6 gio (ke ca fix bug) |
| Files thay doi | 3 (dung expected) |
| Tests viet moi | 3 widget tests |
| Bugs | 1 (P3, da fix loop 1) |
| UX parity | 95% |
| Health score | 88/100 |
| PM approve | ✅ |

---

## 14. Known Issues & Roadmap

### 14.1 Known Issues

| # | Van de | Muc do | Trang thai | Ghi chu |
|---|--------|--------|-----------|---------|
| 1 | Spec compliance check chua tu dong hoa trong build-verify | P3 | Open | Hien tai agent so sanh thu cong |
| 2 | test-brief chua doc tu spec file tu dong | P3 | Open | Agent phai tu sinh tu context |
| 3 | guard khong kiem tra cross-project drift | P4 | Open | Chi kiem tra trong 1 project |
| 4 | sentinel chua ho tro screenshot diff tu dong | P3 | Open | Can tool compare images |
| 5 | bug-log chua tich hop voi Asana/Jira | P4 | Backlog | Hien chi luu local |

### 14.2 Roadmap

| Phien ban | Du kien | Noi dung |
|-----------|---------|----------|
| v3.3 | Q2 2026 | Spec-first test generation tu dong, build-verify doc spec file |
| v3.4 | Q2 2026 | Screenshot diff (sentinel visual), cross-project guard |
| v4.0 | Q3 2026 | Multi-agent orchestration, parallel skill execution |
| v4.1 | Q3 2026 | Tich hop Asana/Jira cho bug-log va test-brief |
| v5.0 | Q4 2026 | Self-learning: agent tu de xuat skill moi tu patterns |

---

## 15. Decision Log

| Ngay | Quyet dinh | Ly do | Anh huong |
|------|-----------|-------|----------|
| 2026-03-13 | Feature Completeness Rule — 6 chieu | Feature thieu chieu = khong dung duoc cho user that | Tat ca skills kiem tra 6 chieu |
| 2026-03-13 | Data Accuracy Rule — zero tolerance | KFSP la app tai chinh, sai = NĐT mat tien | pre-commit + build-verify kiem tra |
| 2026-03-13 | Simulator Testing Rule | Agent khong the tu tap tren iOS Simulator | Quy trinh: yeu cau PM bam → chup screenshot |
| 2026-03-13 | Git Remote Safety Rule | Agent tu push nhầm lên GitHub thay vi GitLab | Moi thao tac git remote phai hoi PM |
| 2026-03-15 | Phase-as-Project Rule | Phase khong co charter → thieu kiem soat | Moi phase phai co 6 chieu charter |
| 2026-03-15 | Dev Journal Compliance | Journal khong ghi → mat kien thuc | 8 layers enforcement |
| 2026-03-19 | Unit Test First Rule (TDD) | Feature khong co test → khong dam ship | pre-commit block commit khi khong co test |
| 2026-03-20 | Spec-first test generation | Test sinh tu spec → coverage tot hon, PM verify dung | test-brief + build-verify se doc spec |

---

## 16. Tai lieu lien quan

| Tai lieu | Duong dan | Mo ta |
|----------|----------|-------|
| **ARCHITECTURE.md** | `kfsp-skill-set/ARCHITECTURE.md` | Kien truc 5 tang, 6 giai doan, cross-domain translation |
| **ORCHESTRATION.md** | `kfsp-skill-set/ORCHESTRATION.md` | Phoi hop KFSP + GSD + Ralph |
| **README.md** | `kfsp-skill-set/README.md` | Quick start, danh sach 20 skills |
| **INSTALL.md** | `kfsp-skill-set/INSTALL.md` | Huong dan cai dat chi tiet + troubleshooting |
| **INTERACTION_SURFACE_MAP.md** | `kfsp-skill-set/references/` | 4 be mat tuong tac chi tiet |
| **15_TESTING_RULES.md** | `kfsp_flutter_migration/docs/` | Quy tac test brief HTML, bug flow |
| **CRM_Operations_Manual.md** | `Business/4-Operations/Automation_Hub/docs/` | Tham khao format Single Source of Truth |

### Ghi chu cap nhat

- Moi thay doi quy trinh → cap nhat tai lieu nay
- Moi phien ban skill set moi → review + cap nhat sections lien quan
- PM review tai lieu nay it nhat 1 lan/thang
- Agent doc tai lieu nay khi `/kfsp:session-start` lan dau

---

> **Tao boi:** Claude Agent | **Review boi:** Thanh (PM) | **Ngay:** 20/03/2026
