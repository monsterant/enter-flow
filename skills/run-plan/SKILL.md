---
name: run-plan
description: plan-it 계획을 단계마다 구현 에이전트와 단계 리뷰어로 진행하고, 끝에 3관점 리뷰를 돌린다. 리뷰 전용 모드도 있다. autoEnterEnter 메뉴 2·3·5·6에서만 쓴다.
disable-model-invocation: true
---

# run-plan

**한국어 요약 (원본과 달라진 점).** obra/superpowers의 subagent-driven-development(MIT, `ORIGIN.md`)를
enter-flow 플러그인 흐름에 맞춰 고친 것이다. `autoEnterEnter`가 이 파일을 읽어서 따른다.
문서 위치·기준 브랜치·커밋 형식은 `../autoEnterEnter/references/settings.md`의 DOCS·BASE·COMMIT을 쓴다.
(주의: 아래 영어 본문의 BASE/HEAD는 **스냅샷**이다. 기준 브랜치는 "base branch"로 적는다)
- 구현자는 **커밋하지 않는다.** 단계 범위는 전·후 **스냅샷**(`scripts/snapshot`)으로 잡는다.
- 커밋은 메인(컨트롤러)이 계획의 **커밋 경계**에서만, COMMIT 설정이 정한 형식으로 한다(6절).
- 계획 제목은 `### N단계:`와 `### Task N` 둘 다 읽는다.
- 마지막 리뷰는 **3관점 리뷰어 + 검증자**(`final-reviewer-prompt.md`, `verifier-prompt.md`).
- **리뷰 전용 모드**: 고치지 않고 3관점 리뷰만 돌린다.
- worktree를 만들지 않고, 기준 브랜치·main·master에서는 멈춘다. push·PR·merge는 하지 않는다.
- 작업 파일은 `.run-plan/`(git 제외), 끝나면 장부만 `DOCS/plans/<계획>-ledger.md`로 남긴다.
- 사용자에게 보이는 보고는 사용자가 쓰는 언어로 쓴다. 아래 영어 본문은 에이전트 운영 규칙이라 원문을 유지한다.

## Subagent-Driven Development (원문)

Execute plan by dispatching a fresh implementer subagent per task, a task review (spec compliance + code quality) after each, and a broad whole-branch review at the end.

**Why subagents:** You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Core principle:** Fresh subagent per task + task review (spec + quality) + broad final review = high quality, fast iteration

**Narration:** between tool calls, narrate at most one short line — the
ledger and the tool results carry the record.

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping. The only reasons to stop are the four named below, or all tasks complete. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the plan, so execute it.

**Rulings, not stalls.** A running plan does not wait on a human. Conflicts,
ambiguities, plan defects, a cap you would have asked to exceed — decide
them. The spec is the binding authority, the plan is its argument, and your
judgment settles what neither answers. Record every decision in the ledger as
`Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep
going. A wrong ruling costs rework your human partner can see and undo; a
session parked on a question costs their whole day and buys nothing.

Four things stop you, and only these: an irreversible or destructive
operation; a security-sensitive action; a side effect outside this worktree
that norms say you ask about first (a merge, a push to a shared branch, a
publish); and a plan so broken that every path forward is a guess. For those,
stop and ask.

## 리뷰 전용 모드 (run-plan 추가)

`autoEnterEnter` 메뉴 3의 "자동"과 메뉴 6(코드 리뷰)이 부르는 모드다. 계획도 구현자도 없이
마지막 3관점 리뷰만 돌리고, **고치지 않고** 결과 목록만 돌려준다.

1. 대상 정하기
   - 기본: 현재 브랜치. BASE = `git merge-base <base branch> HEAD`, HEAD = 새 `bash scripts/snapshot`
     (커밋하지 않은 변경까지 포함)
   - PR 번호를 받으면: `gh pr diff <번호>`를 파일로 저장해 그 파일을 diff 파일로 쓴다 (읽기만)
   - 경로를 받으면: 위 기본 범위에서 그 경로만 (`git diff -U10 <BASE> <HEAD> -- <경로>`)
2. diff 파일 만들기: 계획 파일이 없으므로 `review-package`의 OUTFILE 인자로
   `<repo-root>/.run-plan/review-only/<yyyy-MM-dd>-<브랜치>.diff`를 직접 준다.
   (`sdd-workspace`는 계획 파일이 있어야 해서 쓰지 않는다. PLAN_FILE 자리에는 아무 실제 파일이나 넣어도
   OUTFILE을 주면 쓰이지 않는다 — 이슈 번호가 있으면 그 계획 파일, 없으면 `README.md` 같은 레포 파일)
   폴더를 만들 때 `.run-plan/.gitignore`(내용 `*`)가 없으면 함께 만든다. 없으면 diff 파일이 git에 잡힌다.
3. [final-reviewer-prompt.md](final-reviewer-prompt.md)로 관점 A·B·C 리뷰어를 한 메시지에서 동시에 띄운다.
4. "높음"이 있으면 [verifier-prompt.md](verifier-prompt.md)로 검증자를 띄운다.
5. 돌려주는 것: 관점별 결과(심각도 / 파일:줄 / 문제 / 근거 / 검증 결과), 점검 안 된 관점, 문제 없던 항목.
   장부·커밋·수정 없음. `.run-plan/review-only/`의 diff 파일은 끝나면 지운다.

에이전트 수: 리뷰어 3 + (높음이 있으면) 검증자 1.

## When to Use

```
Only when `autoEnterEnter` calls it:
- menu 2 / menu 5, auto mode → full run (a plan-it plan in DOCS/plans/)
- menu 3 auto mode, menu 6 → review-only mode (리뷰 전용 모드 above)
Never start on your own.
```

## The Process

```dot
digraph process {
    rankdir=TB;

    subgraph cluster_per_task {
        label="Per Task";
        "Dispatch implementer subagent (./implementer-prompt.md)" [shape=box];
        "Implementer asks questions?" [shape=diamond];
        "Answer questions, provide context" [shape=box];
        "Implementer implements, tests, self-reviews (no commit)" [shape=box];
        "Generate review package, dispatch task reviewer (./task-reviewer-prompt.md)" [shape=box];
        "Spec ✅ and quality approved?" [shape=diamond];
        "Fix loop: read references/fix-loop.md before every round" [shape=box];
        "Append completion to ledger, mark todo complete" [shape=box];
    }

    "Setup: branch check, ledger check, read plan, pre-flight review" [shape=box];
    "More tasks remain?" [shape=diamond];
    "Final review and finish: read references/final-review.md" [shape=box];
    "Final review clean: copy ledger to DOCS/plans, delete this plan's workspace" [shape=box];
    "Return to autoEnterEnter wrap-up step" [shape=box style=filled fillcolor=lightgreen];

    "Setup: branch check, ledger check, read plan, pre-flight review" -> "Dispatch implementer subagent (./implementer-prompt.md)";
    "Dispatch implementer subagent (./implementer-prompt.md)" -> "Implementer asks questions?";
    "Implementer asks questions?" -> "Answer questions, provide context" [label="yes"];
    "Answer questions, provide context" -> "Implementer implements, tests, self-reviews (no commit)";
    "Implementer asks questions?" -> "Implementer implements, tests, self-reviews (no commit)" [label="no"];
    "Implementer implements, tests, self-reviews (no commit)" -> "Generate review package, dispatch task reviewer (./task-reviewer-prompt.md)";
    "Generate review package, dispatch task reviewer (./task-reviewer-prompt.md)" -> "Spec ✅ and quality approved?";
    "Spec ✅ and quality approved?" -> "Append completion to ledger, mark todo complete" [label="yes"];
    "Spec ✅ and quality approved?" -> "Fix loop: read references/fix-loop.md before every round" [label="no"];
    "Fix loop: read references/fix-loop.md before every round" -> "Append completion to ledger, mark todo complete" [label="addressed, or parked at the cap"];
    "Append completion to ledger, mark todo complete" -> "More tasks remain?";
    "More tasks remain?" -> "Dispatch implementer subagent (./implementer-prompt.md)" [label="yes"];
    "More tasks remain?" -> "Final review and finish: read references/final-review.md" [label="no"];
    "Final review and finish: read references/final-review.md" -> "Final review clean: copy ledger to DOCS/plans, delete this plan's workspace";
    "Final review clean: copy ledger to DOCS/plans, delete this plan's workspace" -> "Return to autoEnterEnter wrap-up step";
}
```

## Setup

Work happens in the current working tree on the issue branch
(`<type>/<issue-number>`). Run `git branch --show-current` first: if it is
the base branch, `main`, or `master` (base branch = BASE in
`../autoEnterEnter/references/settings.md`), stop and tell your human
partner — never implement there.
Do not create worktrees.

Conversation memory does not survive compaction. In real sessions,
controllers that lost their place have re-dispatched entire completed task
sequences — the single most expensive failure observed. Track progress in
a ledger file, not only in todos.

- Each plan owns a workspace: at skill start, run this skill's
  `bash scripts/sdd-workspace PLAN_FILE` — it prints the plan's git-ignored
  directory (under `<repo-root>/.run-plan/`), home to
  every artifact for THIS plan: ledger, briefs, reports, review packages.
  Another plan's directory is never yours to read or write.
- Check for this plan's ledger at `<workspace>/progress.md`. If one exists,
  read [references/resuming.md](references/resuming.md) before anything
  else: it says whose ledger it is and where to resume.
- Create the ledger with its identity as the first line:
  `# SDD ledger — plan: <plan file path>`.
- The ledger is your recovery map: the snapshot trees and boundary commits
  it names exist in git even when your context no longer remembers creating
  them. After compaction, trust the ledger, `git log`, and `git status` over
  your own recollection. Uncommitted work from finished tasks is still in
  the working tree — never discard it.
- `git clean -fdx` will destroy the workspace (it's git-ignored scratch); if
  that happens, recover from `git log`.

Read the plan once, note its context and Global Constraints, and create a
todo per task. If the plan names a Spec, read that too: the spec is the
authority the plan argues from, and conflicts inside the plan resolve
against it. A plan with no reachable spec gets a ledger note saying so —
rulings made without one are provisional.

Before dispatching Task 1, read [references/preflight.md](references/preflight.md)
and run the conflict scan it describes. Its table and rulings go in the
ledger.

## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

**Mechanical implementation tasks** (isolated functions, clear specs, 1-2 files): use a fast, cheap model. Most implementation tasks are mechanical when the plan is well-specified.

**Integration and judgment tasks** (multi-file coordination, pattern matching, debugging): use a standard model.

**Architecture and design tasks**: use the most capable available model.
The final whole-branch review is one of these — dispatch it on the most
capable available model, not the session default.

**Review tasks**: choose the model with the same judgment, scaled to the
diff's size, complexity, and risk. A small mechanical diff does not need the
most capable model; a subtle concurrency change does. Scoped re-reviews of
small fix diffs take a cheap-to-mid tier.

**Fix-loop escalation (rounds 4-5)**: use a model at least one tier above
the implementer that got stuck.

**Always specify the model explicitly when dispatching a subagent.** An
omitted model inherits your session's model — often the most capable and
most expensive — which silently defeats this section.

**Turn count beats token price.** Wall-clock and context cost scale with how
many turns a subagent takes, and the cheapest models routinely take 2-3× the
turns on multi-step work — costing more overall. Use a mid-tier model as the
floor for reviewers and for implementers working from prose descriptions.
When the task's plan text contains the complete code to write, the
implementation is transcription plus testing: use the cheapest tier for
that implementer. Single-file mechanical fixes also take the cheapest tier.

**Task complexity signals (implementation tasks):**
- Touches 1-2 files with a complete spec → cheap model
- Touches multiple files with integration concerns → standard model
- Requires design judgment or broad codebase understanding → most capable model

## The Task Loop

**Small same-shape work:** when the plan lists several small edits of the
same kind across files, read [references/batching.md](references/batching.md)
before dispatching them.

Everything you paste into a dispatch prompt — and everything a subagent
prints back — stays resident in your context for the rest of the session
and is re-read on every later turn. Hand artifacts over as files.

**Waiting on dispatched subagents:** never poll a wait interface with
short timeouts, and never sit in one silent, open-ended wait either.
While you have local work — ledger updates, packaging the next review,
reading reports — keep working; child results arrive on their own.
When you are genuinely idle, wait in bounded stretches (five to ten
minutes, where your platform allows), and between stretches post one
line of status and reconcile your live children: list them, and chase
any that finished without reporting. A bounded stretch keeps nearly
all of a long wait's efficiency while guaranteeing a stuck or lost
child is noticed within minutes, not at the end of the session.

### 1. Dispatch the implementer

Record BASE before dispatching: run this skill's `bash scripts/snapshot` —
it prints a tree hash of the whole working tree, new untracked files
included, without touching the index. Implementers do not commit, so BASE
is a snapshot, not a commit. The review package and fix-round diffs need it.

- **Task brief:** before dispatching an implementer, run this skill's
  `bash scripts/task-brief PLAN_FILE N` — it extracts the task's full text to a
  uniquely named file and prints the path. Compose the dispatch so the
  brief stays the single source of
  requirements. Your dispatch should contain: (1) one line on where this
  task fits in the project; (2) the brief path, introduced as "read this
  first — it is your requirements, with the exact values to use verbatim";
  (3) interfaces and decisions from earlier tasks that the brief cannot
  know; (4) your resolution of any ambiguity you noticed in the brief;
  (5) the report-file path and report contract. Exact values (numbers,
  magic strings, signatures, test cases) appear only in the brief. Never
  make a subagent read the whole plan file.
- **Report file:** name the implementer's report file after the brief
  (brief `…/task-N-brief.md` → report `…/task-N-report.md`) and put it in
  the dispatch prompt. The implementer writes the full report there and
  returns only status, files changed, a one-line test summary, and concerns.
- A dispatch prompt describes one task, not the session's history. Do not
  paste accumulated prior-task summaries ("state after Tasks 1-3") into
  later dispatches — a real session's dispatch hit 42k chars of which 99%
  was pasted history. A fresh subagent needs its task, the interfaces it
  touches, and the global constraints. Nothing else.
- The dispatch carries the no-subagents contract (it is in the
  implementer template): the implementer never dispatches subagents —
  not helpers, and never a reviewer. Review arrives from you, after the
  report. In real sessions, every reviewer a worker spawned duplicated
  the task review the controller dispatched anyway — a full extra
  review seat per task.
- If an earlier task parked a finding in the area this task touches, carry
  a pointer to that ledger entry in the dispatch.
- Record the implementer's agent identity from the dispatch result —
  fix-loop rounds 1-3 resume this agent.
- Never dispatch multiple implementation subagents in parallel (conflicts).

Template: [implementer-prompt.md](implementer-prompt.md)

### 2. Handle the report

Implementer subagents report one of four statuses. Handle each appropriately:

**DONE:** Take HEAD with `bash scripts/snapshot`, then generate the review package (`bash scripts/review-package PLAN_FILE BASE HEAD`, from this skill's directory — it prints the unique file path it wrote; BASE is the snapshot you recorded before dispatching the implementer), then dispatch the task reviewer with the printed path.

**Any other status** (DONE_WITH_CONCERNS, NEEDS_CONTEXT, BLOCKED): read
[references/handling-reports.md](references/handling-reports.md) and follow
it, every time you get one.

If the implementer asks questions — before starting or mid-task — answer
clearly and completely, provide additional context if needed, and don't
rush it into implementation.

### 3. Review the task

Per-task reviews are task-scoped gates. The broad review happens once, at the
final whole-branch review. Never skip the task review, and never accept a
report missing either verdict — spec compliance AND task quality are both
required. Implementer self-review never replaces the task review; both are
needed.

- Hand the reviewer its diff as a file: run this skill's
  `bash scripts/review-package PLAN_FILE BASE HEAD` and pass the reviewer the file path
  it prints (or, without bash: `git diff --stat BASE HEAD` and
  `git diff -U10 BASE HEAD`, redirected to one uniquely named file). The
  output never enters your own context, and the reviewer sees the stat
  summary and full diff with context in one Read call. BASE and HEAD are
  the snapshots taken before and after this task, so earlier tasks'
  uncommitted work is excluded. Never dispatch a task reviewer without a
  diff file.
- **Reviewer inputs:** the task reviewer gets three paths — the same brief
  file, the report file, and the review package — plus the global
  constraints that bind the task.
- The global-constraints block you hand the reviewer is its attention
  lens. Copy the binding requirements verbatim from the plan's Global
  Constraints section or the spec: exact values, exact formats, and the
  stated relationships between components ("same layout as X", "matches
  Y"). The reviewer's template already carries the process rules (YAGNI,
  test hygiene, review method) — the constraints block is for what THIS
  project's spec demands.
- Do not add open-ended directives like "check all uses" or "run race tests
  if useful" without a concrete, task-specific reason
- Do not ask a reviewer to re-run tests the implementer already ran on the
  same code — the implementer's report carries the test evidence
- Do not pre-judge findings for the reviewer — never instruct a reviewer to
  ignore or not flag a specific issue. If you believe a finding would be a
  false positive, let the reviewer raise it and adjudicate it in the review
  loop. If the prompt you are writing contains "do not flag," "don't treat X
  as a defect," "at most Minor," or "the plan chose" — stop: you are
  pre-judging, usually to spare yourself a review loop.
If the reviewer reports "⚠️ Cannot verify from diff" items, read
[references/cannot-verify.md](references/cannot-verify.md) before you mark
the task complete.

Template: [task-reviewer-prompt.md](task-reviewer-prompt.md)

### 4. The fix loop

The loop triggers when the review reports spec ❌, any Critical or Important
finding, or a ⚠️ item you confirmed as a real gap.

When it triggers, read [references/fix-loop.md](references/fix-loop.md) and
follow it. Read it again before every round, even if you read it earlier in
this session: it holds the round cap, the breaker, and the ledger line each
round writes, and your memory of it does not survive compaction.

### 5. Complete the task

When the review comes back clean — or every open finding is parked with a
ruling at the cap — append the completion line to the ledger in the same
message as your other bookkeeping:

- `Task <N>: complete (snapshot <base7>..<head7>, review clean)`
- `Task <N>: complete (snapshot <base7>..<head7>, <K> parked)` after a
  tripped breaker

Then mark the todo complete. If the plan has a commit-boundary line right
after this task, commit now (next section) before moving on. Never move to
the next task while the review has open Critical/Important issues that are
neither fixed nor parked-with-ruling at the cap.

### 6. Commit at the plan's commit boundaries

The plan marks commit boundaries between tasks with a line such as
`── 커밋 ①: <title> (1~2단계) ──`. Only the controller commits, and
only at these boundaries — implementers never commit.

1. Use the message format chosen by COMMIT in
   `../autoEnterEnter/references/settings.md` (repo commit skill → rule
   docs → last 20 commits → Conventional Commits) and ledger which source
   decided it. Start from the title the plan's boundary line gives. AI
   attribution (Co-Authored-By etc.) follows the same source; if no source
   says anything, follow Claude Code's default. Any approval step in a repo
   commit skill is skipped: run-plan runs only in auto mode, where local
   commits are allowed.
2. Stage by file name — exactly the files in
   `git diff --name-only <snapshot at previous boundary> <snapshot now>`
   (the first boundary's "previous" is the BASE snapshot of Task 1, recorded
   in the ledger as `Start snapshot: <tree>`). This keeps out changes that
   were already in the working tree before run-plan started. Never
   `git add -A` or `git add .`. Leave out `.env`, secrets, and personal
   settings files. Ledger the boundary snapshot with the commit line.
3. Commit, then run `git log -1 --stat` and check that only the intended
   files went in.
4. Ledger: `Commit <①>: <sha7> <subject> (tasks <a>~<b>, snapshot <tree7>)`.

Fixes from the final whole-branch review become one extra commit in the COMMIT
format, titled as "<issue title> — final review fixes" in the
user's language (e.g. `fix: address final review for order unlock`).
Never amend or rebase.

## Final Review

When every task is complete, read
[references/final-review.md](references/final-review.md) and follow it. It
covers the whole-branch review, its single fix wave, and how you finish.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Close enough on spec compliance" | Reviewer found spec gaps = not done. Fix or hit the cap and adjudicate — those are the only exits. |
| "I'll fix it myself, dispatching is overhead" | Controller fixes pollute your context and skip review. Resume the implementer. |
| "One more round will converge" | Past the cap, rounds don't converge — the failure is structural. Adjudicate and route. |
| "The reviewer will just find something new anyway" | Scoped re-reviews verify fixes; they cannot wander. New findings on untouched code go to the ledger, not the loop. |
| "This finding is obviously wrong, I'll drop it" | You adjudicate only at the cap, and every ruling is a ledger entry. Silent discards are forbidden. |
| "The fix was small, skip the re-review" | Unreviewed fixes are how regressions land. Every round ends with a scoped re-review. |
| "Reviews slow the loop down" | The loop without reviews is just unverified churn. Reviews are the loop's brakes and steering. |
| "Ledger bookkeeping is overhead" | The ledger is what survives compaction. Controllers without one have re-dispatched entire completed task sequences. |
| "The implementer spawned its own reviewer — free extra assurance" | It's a duplicate seat reviewing the same diff; the task review is the gate. A worker-spawned reviewer is a defect to flag, not rigor. |
