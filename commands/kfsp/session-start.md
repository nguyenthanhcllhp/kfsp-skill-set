---
name: kfsp:session-start
description: New session startup — check git changes, detect code-doc drift, update memory/docs. Run at the start of every Flutter dev session.
argument-hint: "[--quick] [--skip-pull]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Agent
  - TodoWrite
  - Edit
  - Write
  - AskUserQuestion
---
<objective>
KFSP Session Start — Quy trình bắt đầu phiên làm việc mới.

**Vấn đề:** Khi bắt đầu session mới, agent thường không biết:
1. Code đã thay đổi từ git (dev push code mới)
2. Docs/memory bị lệch so với code thực tế
3. Có branch mới hoặc conflict cần xử lý

**Mục tiêu:** Đảm bảo agent có context chính xác TRƯỚC khi bắt đầu bất kỳ task nào.

**Khi nào chạy:**
- Đầu MỌI session Flutter dev
- Sau khi user nói "tôi vừa pull code" hoặc "code mới từ git"
- Khi nghi ngờ code đã thay đổi từ phiên trước

**Flags:**
- `--quick` → Chỉ check git status + memory, skip doc audit
- `--skip-pull` → Không hỏi user về git pull, chỉ check state hiện tại
</objective>

<instructions>
## Step 0: Read Context + Detect Tools

### 0a: Đọc memory:
```
Read: memory/MEMORY.md
Read: memory/flutter-migration.md
```
Ghi nhận: state, source path, known bugs.

### 0b: Detect Orchestration Tools (BẮT BUỘC)
```bash
KFSP_COUNT=$(ls ~/.claude/commands/kfsp/*.md 2>/dev/null | wc -l | tr -d ' ')
GSD_COUNT=$(ls ~/.claude/commands/gsd/*.md 2>/dev/null | wc -l | tr -d ' ')
RALPH=$(command -v ralph 2>/dev/null && echo "installed" || ([ -f ".ralph/PROMPT.md" ] && echo "enabled" || echo "not found"))
echo "KFSP: $KFSP_COUNT | GSD: $GSD_COUNT | Ralph: $RALPH"
```

**Workflow mode:**
- KFSP ≥ 17 + GSD + Ralph → **Full Orchestration**
- KFSP ≥ 17 + GSD → **GSD + KFSP**
- KFSP ≥ 17 only → **KFSP Only**
- Thiếu KFSP → ⚠️ Cảnh báo user

> Chi tiết: `ORCHESTRATION.md` trong kfsp-skill-set repo.

## Step 1: Git Status Check

```bash
cd /tmp/dev_flutter_kfsp 2>/dev/null || cd /tmp/kfsp_flutter 2>/dev/null

# Branch hiện tại
git branch --show-current

# Remote status
git remote -v 2>/dev/null
git fetch --dry-run 2>/dev/null

# Local changes (uncommitted)
git status --short 2>/dev/null | head -30

# Last N commits (local)
git log --oneline -10 2>/dev/null

# Ahead/behind remote
git status -sb 2>/dev/null | head -1
```

## Step 2: Hỏi User về Git Changes

**BẮT BUỘC hỏi user** (trừ khi `--skip-pull`):

> "Bạn có pull code mới từ git không? Hoặc có ai push code mới lên không?"

Tùy câu trả lời:

### Nếu CÓ code mới (user đã pull hoặc cần pull):
```bash
# Check what changed
git log --oneline HEAD@{1}..HEAD 2>/dev/null  # commits since last session
git diff --stat HEAD@{1}..HEAD 2>/dev/null     # files changed
git diff --name-only HEAD@{1}..HEAD 2>/dev/null # file list
```

Nếu user CHƯA pull:
```bash
# Show what's available
git fetch origin 2>/dev/null
git log --oneline HEAD..origin/$(git branch --show-current) 2>/dev/null
```

Hỏi user: "Có X commits mới trên remote. Bạn muốn pull không?"

### Nếu KHÔNG có code mới:
→ Skip to Step 3

## Step 3: Detect Changes (so sánh với memory)

So sánh code thực tế vs memory/flutter-migration.md:

### 3a: File count change
```bash
cd [source_path]
find lib -name "*.dart" | wc -l
find lib/screens -type d -maxdepth 1 | sort
find lib/core -type d -maxdepth 2 | sort
```

So sánh với số liệu trong memory. Nếu khác → flag.

### 3b: New files/folders
```bash
# Files modified in last 24h (or since last known date)
find lib -name "*.dart" -newer pubspec.lock -type f 2>/dev/null | head -30
```

### 3c: Route changes
```bash
grep -n "GoRoute\|ShellRoute\|StatefulShellRoute" lib/router/app_router.dart 2>/dev/null || \
grep -n "GoRoute\|ShellRoute\|StatefulShellRoute" lib/config/routes/app_router.dart 2>/dev/null
```

So sánh routes trong code vs routes trong docs/02_ARCHITECTURE_GUIDE.md.

### 3d: New dependencies
```bash
# Compare pubspec.yaml dependencies
grep -A 100 "^dependencies:" pubspec.yaml | grep -B 100 "^dev_dependencies:" | grep ":" | grep -v "#"
```

### 3e: New screens/features
```bash
# List all screen files
find lib/screens -name "*_screen.dart" -o -name "*_page.dart" | sort
# List all features
find lib/features -maxdepth 1 -type d 2>/dev/null | sort
```

## Step 4: Code-Doc Drift Report

Đọc các docs chính và so sánh với code:

```
Read: Product/kfsp_flutter_migration/docs/08_HANDOFF_STATUS.md (progress)
Read: Product/kfsp_flutter_migration/docs/04_FEATURE_SPECS.md (features)
Read: Product/kfsp_flutter_migration/docs/02_ARCHITECTURE_GUIDE.md (routes, patterns)
```

Check:
- Features trong code nhưng KHÔNG trong docs → 🔴
- Features trong docs nhưng KHÔNG trong code → ⚠️
- Progress % trong docs khác với thực tế → ⚠️
- Routes trong code khác với docs → 🔴
- New dependencies không trong docs → ⚠️

## Step 5: Auto-Fix Drift (nếu phát hiện)

Nếu phát hiện drift:

1. **Hỏi user:** "Phát hiện X khác biệt giữa code và docs. Bạn muốn tôi cập nhật docs không?"
2. Nếu user đồng ý → cập nhật:
   - `docs/08_HANDOFF_STATUS.md` — progress, features, bugs
   - `docs/04_FEATURE_SPECS.md` — feature list
   - `docs/02_ARCHITECTURE_GUIDE.md` — routes, patterns
   - `memory/flutter-migration.md` — current state
   - `memory/MEMORY.md` — summary

3. Nếu `--quick` → chỉ cập nhật memory, skip docs

## Step 5.5: Phase-as-Project + Dev Journal Check (2026-03-15+)

### Check Project Charter exists for current phase:
```bash
PHASE=$(grep -r "Phase hiện tại" memory/flutter-migration.md 2>/dev/null | grep -oP "P\d+" | head -1)
if [ -n "$PHASE" ]; then
  CHARTER="Product/kfsp_flutter_migration/docs/build_reports/${PHASE}_project_charter.md"
  [ -f "$CHARTER" ] && echo "✅ Charter: $CHARTER" || echo "⚠️ NO CHARTER for $PHASE — cần tạo trước khi code"
fi
```

### Check Dev Journal has entries:
```bash
JOURNAL_DIR="Product/kfsp_flutter_migration/journal"
MONTH=$(date +%Y-%m)
ENTRIES=$(find "$JOURNAL_DIR/$MONTH/" -name "*.md" 2>/dev/null | wc -l | tr -d ' ')
echo "📔 Dev journal entries (${MONTH}): $ENTRIES"
[ "$ENTRIES" -eq 0 ] && echo "⚠️ No journal entries — remember to log decisions"
```

### Add to report (Step 7):
```markdown
## Phase Compliance
- Project Charter: ✅/⚠️ [path or "cần tạo"]
- Dev Journal: X entries this month
- Build Reports: Y reports this phase
```

## Step 6: Quick Health Check

Chạy mini version của `guard`:
```bash
# Build check
cd [source_path]
flutter analyze --no-pub 2>&1 | tail -5

# File count
echo "Dart files: $(find lib -name '*.dart' | wc -l)"
echo "Assets: $(find assets -type f 2>/dev/null | wc -l)"
echo "Test files: $(find test -name '*_test.dart' 2>/dev/null | wc -l)"
```

## Step 7: Output Report

```markdown
# 🚀 KFSP Session Start Report
**Date:** [timestamp]
**Branch:** [branch name]
**Source:** [path]

## Orchestration Tools
| Tool | Status | Mode |
|------|--------|------|
| KFSP | ✅/❌ [N] commands | Bảo vệ |
| GSD | ✅/⚠️ [N] commands | Planning |
| Ralph | ✅/⚠️ | Auto-loop |
| **Workflow:** | [Full / GSD+KFSP / KFSP Only / ⚠️ Unprotected] | |

## Git Status
- Branch: [name] (ahead/behind: X/Y)
- Uncommitted changes: X files
- New commits since last session: Y

## Code Changes Detected
| Area | Memory Says | Code Says | Drift? |
|------|------------|-----------|--------|
| Dart files | N | M | ✅/⚠️ |
| Screens | [list] | [list] | ✅/⚠️ |
| Routes | [list] | [list] | ✅/⚠️ |
| Dependencies | N | M | ✅/⚠️ |

## Doc-Code Drift
| Doc | Status | Issues |
|-----|--------|--------|
| 08 Handoff | ✅/⚠️ | [details] |
| 04 Features | ✅/⚠️ | [details] |
| 02 Architecture | ✅/⚠️ | [details] |
| Memory | ✅/⚠️ | [details] |

## Health Quick Check
- Analyzer: X errors, Y warnings
- Dart files: N
- Test coverage: N files

## Recommended Next Steps
1. [prioritized actions based on findings]

## Ready to Work ✅
Context is up-to-date. Proceed with: [current phase/task from memory]
```

## Nếu `--quick` mode:
Skip Steps 4-5 (doc audit). Only do:
- Step 1 (git status)
- Step 2 (ask user)
- Step 3 (detect changes — file count only)
- Step 6 (health check)
- Abbreviated report

## 🔒 Git Remote Safety Check (2026-03-13+)

During Step 1 (Git Status), ALSO verify:
```bash
git remote -v
```
- App code (`flutter_kfsp_app`): `origin` MUST = `https://gitlab.com/phudh3/flutter_kfsp_app.git`
- Skills (`kfsp-skill-set`): `origin` MUST = GitHub `nguyenthanhcllhp/kfsp-skill-set`
- If remote was renamed/changed → 🔴 FLAG immediately → ask Thanh
- **NEVER** push to any remote without Thanh's explicit permission
- **NEVER** assume GitHub = GitLab or vice versa
</instructions>
</content>
</invoke>