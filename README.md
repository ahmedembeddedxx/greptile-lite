# greptile-lite

A Greptile-style code review skill for Claude Code. It reviews a GitHub pull request (or your local branch), finds real bugs with concrete failure scenarios, grades them P0/P1/P2, scores the PR out of 5, and posts inline comments on the changed lines plus one summary comment.

## Install

```bash
git clone https://github.com/ahmedembeddedxx/greptile-lite.git
mkdir -p ~/.claude/skills && cp -r greptile-lite/skills/greptile-lite ~/.claude/skills/
```

Needs the GitHub CLI (`gh`) signed in to an account that can comment on the repository.

## Use

In Claude Code, inside the repository:

- `/greptile-lite 123` reviews PR #123 and posts the review.
- `/greptile-lite` reviews the current branch against its base and prints the review.
- `--no-post` prints only, `--strict` also reports small cleanups, `--max N` caps inline comments (default 10).

## What it checks

Correctness, broken callers and contracts, data and migrations, concurrency and time, security, failure paths, tests that cannot fail, and whether docs and other clients (API, SDK, MCP, frontend) still match the change. Every finding is verified against the code before it is kept; speculative and style comments are dropped.

## Scoring

- 5/5 correct and tested; 4/5 only P2s, safe to merge
- 3/5 any open P0 or P1 (do not merge)
- 2/5 or lower: several P0/P1 or the wrong approach

It comments only (never approves, requests changes or merges) and never posts the same review twice.
