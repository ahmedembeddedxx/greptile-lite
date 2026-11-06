# greptile-lite

**I replaced my $100/month Greptile subscription with this one skill, and it caught more real bugs.**

Same workflow you know from Greptile (inline comments on the exact lines, a summary with a confidence score, a merge call), running on the Claude Code you already pay for. Any language, any stack, any GitHub repository. No extra seat, no extra bill.

## Why it holds up

- **Reads the code around the change**, not just the hunk: every caller and reader of what changed, plus your repo's own rules (`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`).
- **Verifies before it comments.** Each finding needs a concrete input that breaks, and is checked against the code to try to disprove it. No "consider renaming", no noise.
- **Checks what a diff-only bot misses:** stale callers, docs that no longer match, and public interfaces whose other consumers (apps, SDKs, CLIs, services) no longer match.
- **Comments only.** Never approves, never merges, never posts twice.

## What it does

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

Correctness, broken callers and contracts, data and migrations, concurrency and time, security, failure paths, tests that cannot fail, and whether docs and other consumers of a changed interface still match. Every finding is verified against the code before it is kept; speculative and style comments are dropped.

## Scoring

- 5/5 correct and tested; 4/5 only P2s, safe to merge
- 3/5 any open P0 or P1 (do not merge)
- 2/5 or lower: several P0/P1 or the wrong approach

It comments only (never approves, requests changes or merges) and never posts the same review twice.
