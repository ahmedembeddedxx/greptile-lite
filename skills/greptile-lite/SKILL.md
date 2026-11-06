---
name: greptile-lite
description: Greptile-style AI code review for any repository, any language or stack. Reviews a GitHub pull request (or a local diff), finds real bugs with concrete failure scenarios, grades them P0/P1/P2, gives a score out of 5, and posts inline comments on the changed lines plus one summary comment, the way Greptile does. Use when asked to "review this PR", "greptile this", "greptile lite", "drop review comments", or to score a PR before merging.
---

# Greptile Lite: review a PR and drop comments

Works on any codebase: language, framework and stack are read from the repo itself. Needs `git` and the GitHub CLI (`gh`) for posting; without `gh`, use `--no-post`.

Goal: the review a careful senior engineer would leave. Few comments, each one a real problem with a concrete scenario, anchored to the exact line. No style nits, no praise, no restating the diff.

## Inputs

- A PR number or URL (`/greptile-lite 123`, `/greptile-lite https://github.com/o/r/pull/123`), or nothing: then review the current branch against its base (`git diff $(git merge-base HEAD origin/<base>)...HEAD` plus uncommitted changes) and print the review instead of posting it.
- Optional flags: `--no-post` (print only), `--strict` (also report P2 cleanups), `--max N` (cap inline comments, default 10).

## 1. Gather context (read before judging)

1. `gh pr view <n> --json title,body,baseRefName,headRefName,isDraft,files,commits` and `gh pr diff <n>`.
2. Check out the head read-only (`git fetch origin pull/<n>/head:pr-<n>` and `git show pr-<n>:<path>`), never in a dirty working tree.
3. For every changed function, type, route, query or config key: open the full file, and find its callers and readers (`git grep`). A change is judged by what depends on it, not only by the hunk.
4. Read the repo's own rules if present: `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `.github/pull_request_template.md`. Their rules are review criteria.
5. Read existing review threads (`gh pr view <n> --comments`, `gh api repos/{o}/{r}/pulls/<n>/comments`) so you do not repeat a point already raised or already fixed.

## 2. Hunt, by lens

Go through each lens on every hunk. Write down candidate findings with file, line and a one-line scenario.

- **Correctness**: inverted or off-by-one conditions, wrong units or timezones, null/None/undefined paths, missing `await`, wrong default, a branch that can never run, mixed bases (comparing values measured differently), a key collision (looking things up by a non-unique name).
- **Callers and contracts**: a changed signature, return shape or meaning that an untouched caller still relies on; a removed field that another consumer (a client app, SDK, CLI, plugin or dependent service) still reads.
- **Data and state**: writes without a transaction, partial failure leaving bad state, migrations that lock or cannot be rolled back, caches keyed on the wrong thing, idempotency of retries.
- **Concurrency and time**: races, shared mutable state, timeouts that hang, tests or code that depend on today's date or wall-clock timing.
- **Security**: authz on new routes, user input reaching SQL/shell/templates, secrets in code or logs, data from one user reaching another.
- **Failure paths**: what happens when the optional dependency, network call or parse fails: degrade with a message, or crash/empty answer.
- **Tests**: does a test actually fail if the change is reverted? Tests that only assert mocks, or pinned values updated without reason.
- **Docs and consumer alignment** (only where the repo has them): if a user can now see or do something different, are the README, docs or changelog updated? If a public interface changed (an endpoint, function, schema, CLI flag or config key), do the other code paths and consumers that expose or read it still match?

## 3. Verify every candidate (this is what makes it Greptile-quality)

For each candidate, before keeping it:
- Re-read the code around it and its callers. Can you state the exact input or state that produces the wrong result?
- Try to refute it: is it handled elsewhere (a guard upstream, a constraint in the database, a caller that never passes that value)?
- If you can run it cheaply (a unit test, a one-line script), do.
Keep only findings that survive. Drop "might", "could consider", naming and formatting.

## 4. Grade and score

- **P0**: breaks users or data now (wrong numbers shown, data loss, security hole, crash on a normal path, irreversible migration, leaked secret).
- **P1**: will break under a realistic case, or breaks a caller/contract, or a missing check the change needs.
- **P2**: worth fixing, low risk (edge case, missing test, small doc/client drift).

Score out of 5:
- **5/5**: correct, tested, nothing to fix.
- **4/5**: only P2s. Safe to merge.
- **3/5**: any open P1 or P0 (an open P0/P1 caps the score at 3 and means do not merge).
- **2/5 or lower**: several P0/P1, or the approach is wrong.

## 5. Post (unless `--no-post` or a local diff)

One GitHub review with inline comments, plus the summary as its body. Build a JSON file and post it once:

```bash
gh api repos/{owner}/{repo}/pulls/<n>/reviews --method POST --input review.json
```

```json
{
  "commit_id": "<head sha you reviewed>",
  "event": "COMMENT",
  "body": "<summary, below>",
  "comments": [
    {"path": "src/module.ext", "line": 42, "side": "RIGHT",
     "body": "**P1**: <what is wrong>. <Scenario: concrete input -> wrong result>. <Fix in one line, or a ```suggestion block>."}
  ]
}
```

Rules for inline comments: anchor to a line that is in the diff (RIGHT side for added lines); one problem per comment; use a ```suggestion block only when the fix is a drop-in replacement of those lines; at most `--max` comments, most severe first.

Summary body shape:

```
**Greptile Lite review: 4/5**

<2-4 sentences: what the PR does and what it leaves unchanged.>

| Severity | Finding | File |
|---|---|---|
| P2 | <one line> | `path:line` |

**Docs:** updated / owed: <page> / not applicable.  **Consumers:** aligned / owed: <what> / not applicable.
**Merge:** yes (4/5+, no P0/P1) / no, until <finding> is fixed.
```

If nothing survives verification, post a short summary (score, what you read, "no issues found") and no inline comments.

Never post twice: if the API call times out, list the PR's reviews and check for yours before retrying. Do not approve, request changes, or merge unless the user asks; `event: COMMENT` only.

## 6. Re-review after fixes

When asked to review again: read the new commits only, check each earlier finding (resolved or not, reply on its thread), hunt the new hunks, and post a fresh summary with the new score.

## 7. Report back

In chat, give: the score, each finding as `path:line, P#, one line`, the docs/clients verdict, and the link to the posted review.
