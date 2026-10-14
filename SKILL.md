---
name: full-send
description: Say "full send": fix, verify, open a PR, babysit, and merge.
when-to-use: The user says full send, /full-send, or $full-send.
user-invocable: true
---

# Full Send

The user said "full send". The words after that are the task. Finish it. The person who sent you will not look at this again, so the pull request has to be right when you merge it.

Done means the pull request is merged and you can say, in one sentence, what you verified on the merged head. If you cannot, the pull request stays open.

## Stop without merging

Leave the pull request open and report its URL plus the specific doubt when any of these are true:

- The task is ambiguous in a way that changes the result, and the repo does not already answer it.
- You cannot run a real verification for this change.
- Required CI is red, a required review is missing, or branch protection blocks the merge.
- An unresolved comment needs a product decision.
- After the last pass, confidence is not high. Confidence is defined in the merge step.

## 1. Take the task

Restate the outcome in one sentence before editing. Read the code the change will touch. Do the smallest change that completes that outcome, in the repo's existing style.

Work on a branch that is not the default branch. Continue a feature branch that already holds this task. If the worktree has unrelated uncommitted work, stop.

## 2. Fix

Implement the task. Keep the diff limited to the outcome.

## 3. Verify before the pull request

Run the checks this repo already uses for the files you touched: tests, typecheck, lint, build. For a web UI change, exercise the changed flow in a browser, including the empty and error states that share the components you touched. A command you did not run is not a pass.

Fix what fails and re-run it. If you cannot get a real pass, stop.

## 4. Open the pull request

Commit, push, and open the pull request against the default branch with `gh pr create`.

The body stands alone. Use these sections:

- What changed
- How you verified it, naming the commands and the result
- Residual risk

If this session exposes `link_pull_request`, call it with the full pull request URL immediately after creation.

## 5. Babysit

Stay with the pull request until it is quiet. Pending checks are not quiet.

Make later fixes on the pull request branch. Use a separate worktree when the current checkout is on another branch or has unrelated dirty files.

On each pass, read:

```bash
gh pr view <number> --json state,mergeable,mergeStateStatus,reviewDecision,statusCheckRollup,url,headRefOid
gh pr checks <number>
```

And the unresolved review threads:

```bash
gh api graphql -f owner='<owner>' -f name='<repo>' -F number=<number> -f query='
query($owner:String!,$name:String!,$number:Int!){
  repository(owner:$owner,name:$name){
    pullRequest(number:$number){
      reviewThreads(first:100){
        pageInfo { hasNextPage endCursor }
        nodes { isResolved }
      }
    }
  }
}'
```

Paginate while `hasNextPage` is true. Count a thread only from the live query.

If the host can watch in the background, watch instead of busy-polling. Wake on a failed check, a new review, a merge conflict, or the pull request becoming quiet. Stay silent while nothing has changed.

Act on what you find, then watch the new head:

- Failed or errored CI: read `gh run view <run-id> --log-failed`, fix the cause, push.
- A check that failed once and looks flaky: rerun it once with `gh run rerun <run-id> --failed`. A second failure is a real failure. Fix the cause.
- Merge conflict: rebase onto the current base, resolve by reading both sides, push with `--force-with-lease`.
- An unresolved review thread, from a person or a bot: when the finding is valid, fix it, push, reply with the commit SHA, and resolve the thread. When the finding is wrong, reply with the technical reason, then resolve the thread. Resolve a thread only after that reply exists.

`CHANGES_REQUESTED` uses the same rule on the review body.

The pull request is quiet only when all of these hold on the current head:

- `state` is `OPEN`
- `mergeable` is `MERGEABLE` (`UNKNOWN` is not quiet; wait and query again)
- no check conclusion is `FAILURE`, `ERROR`, `CANCELLED`, `TIMED_OUT`, `STARTUP_FAILURE`, or `STALE`
- no required check is still queued or in progress
- `reviewDecision` is not `CHANGES_REQUESTED`
- no review thread has `isResolved: false`

A skipped or neutral check does not block. An empty check rollup means the repo ran no CI. Say that in the report. Local verification is then the only gate.

Bots often comment after CI turns green. Look at recent merged pull requests for the review bots this repo actually runs. Wait while those checks or reviews are still pending. If the repo has no such bot, do not invent a wait.

## 6. Last verification

After the pull request is quiet, do this once on that head:

- Re-read the full diff against the one-sentence outcome.
- Re-run the local check that covers the riskiest part of the diff.
- Update the pull request body when the diff or the verification changed.

A green CI run does not replace this pass.

## 7. Merge

Merge only when every line below is true:

- The diff does the task and nothing load-bearing beyond it.
- The pull request is quiet on this head.
- You saw the last verification pass on this head.
- The residual risk is minor, and you can name it.

If any line is false, follow **Stop without merging**.

Use the first merge method the repo allows: squash, then merge commit, then rebase.

```bash
gh repo view --json squashMergeAllowed,mergeCommitAllowed,rebaseMergeAllowed
gh pr merge <number> --squash --delete-branch
```

Use `--merge` or `--rebase` when squash is not allowed. Do not pass `--admin`. Do not force-push the default branch.

Confirm `state` is `MERGED`. If merge succeeded and branch deletion failed, the task is still done.

Report the pull request URL, the merge commit, what you verified, and the residual risk. Keep it short.
