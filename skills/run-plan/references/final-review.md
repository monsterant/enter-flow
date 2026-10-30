# Final Review and Finish

run-plan의 일부. Every task in the plan is
complete.

The final whole-branch review gets a package too: run
`bash scripts/review-package PLAN_FILE MERGE_BASE HEAD` (MERGE_BASE = the commit the
branch started from, `git merge-base <base branch> HEAD`; HEAD = a fresh
`bash scripts/snapshot`, so any uncommitted work is included) and include the
printed path in every final reviewer's dispatch, so each reads one file
instead of re-deriving the branch diff with git commands.

The final review is three perspective reviewers plus a verifier
(run-plan 변경 — 원본의 리뷰어 1명 대신):

1. Decide whether perspective A gets the money-calculation item by the
   keyword check in [final-reviewer-prompt.md](../final-reviewer-prompt.md).
2. Dispatch the three reviewers A, B, C **in one message** so they run in
   parallel, on the most capable available model, with the same diff file.
   Hand them the ledger's deferred-minor and parked lines.
3. Collect every "높음" finding and dispatch the verifier
   ([verifier-prompt.md](../verifier-prompt.md)). Unconfirmed 높음 drop to
   "중간(확인 필요)" and are not fixed automatically.
4. Fix targets = confirmed 높음 + 중간 that are not "확인 필요".
   낮음 and "확인 필요" go to the ledger as deferred and into the final
   message.

If there are fix targets, dispatch ONE fix subagent
with the complete fix-target list — not one fixer per finding.
Per-finding fixers each rebuild context and re-run suites; a real
session's final-review fix wave cost more than all its tasks combined.
The fix subagent does not commit (same rules as the implementer template).
Take a snapshot before dispatching it (FIX_BASE) and after (HEAD), then run
exactly one scoped re-review of the fix wave
(`bash scripts/review-package PLAN_FILE FIX_BASE HEAD`,
[re-review-prompt.md](../re-review-prompt.md)). Afterwards commit the fix
wave as one extra commit (COMMIT format, "final review fixes")
(SKILL.md section 6).
Adjudicate any residual findings as in the fix loop's breaker
(references/fix-loop.md): park with rulings, or rule on the load-bearing
ones and ledger what you decided. Only the four things that stop a running
plan (top of SKILL.md) stop you here. There is no second fix wave —
residual load-bearing findings surface to your human partner in the
final message (see Finish).

### Finish

Before you delete anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, breaker adjudications, all of them — into
your final message under "Rulings I made", in the order you made them, each
with what it costs if wrong. The list is exhaustive: if the ledger holds a
ruling, the list holds it. That list is the only place the decisions you
took on your human partner's behalf reach them — they read it and rework
whatever you got wrong. A ruling that dies with the workspace was a decision
made in secret.

run-plan 변경 — 마지막 메시지는 사용자가 쓰는 언어로, 아래 순서로 쓴다. `autoEnterEnter`가 1번으로 사용자 확인을 받는다.

    ### 1. 설계와 다르게 정한 것 (확인 필요)
    | # | 설계 문서에는 | 실제로 정한 것 | 이유 | 틀렸으면 드는 비용 |
    (Ruling 중 설계 문서·계획과 다른 결정만. 없으면 "없음")

    ### 2. 그 밖에 내가 정한 것
    (나머지 Ruling 전부, 정한 순서대로)

    ### 3. 커밋
    | 커밋 | 제목 | 단계 |

    ### 4. 남은 지적
    (낮음, "중간(확인 필요)", 점검 안 된 관점)

    ### 5. 테스트
    (마지막으로 실행한 명령과 결과. 실행하지 못한 것은 "확인 못 함")

When the final whole-branch review is clean and its fixes are committed,
copy the ledger to `DOCS/plans/<plan-basename>-ledger.md` (next to the
plan file; use the full path, do not cd into DOCS), then delete this
plan's workspace (`rm -rf <workspace>`). Sibling directories belong to other
plans; leave them alone.

Do not push, open a PR, or merge. If `autoEnterEnter` called you, return to
the wrap-up step of that menu.
