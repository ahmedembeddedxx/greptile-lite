# greptile-lite: free AI code review for every pull request

**Greptile-style PR reviews inside Claude Code. Inline comments, P0/P1/P2 severity, a score out of 5, and a merge call, for $0 extra.**

> I replaced my $100/month Greptile subscription with this one skill, and it caught more real bugs.

One command, `/greptile-lite 123`, and your PR gets the review a careful senior engineer would leave: a few comments, each one a real bug with the exact input that breaks it, anchored to the line. Any language, any framework, any GitHub repository. No extra seat, no extra bill, no code sent to another vendor.

### What a review looks like

```
Greptile Lite review: 3/5

| Severity | Finding                                                        | File               |
|----------|----------------------------------------------------------------|--------------------|
| P1       | Retry re-sends the charge: no idempotency key on the 2nd call  | billing/charge.ts:88 |
| P2       | New `limit` param is not in the API docs                       | docs/api.md        |

Merge: no, until the P1 is fixed.
```

## Why it beats a diff-only bot

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

## Compared with a paid review bot

| | greptile-lite | Paid AI review bot |
|---|---|---|
| Price | $0 on top of Claude Code | per seat, per month |
| Inline comments + summary | yes | yes |
| Reads callers and repo rules, not just the diff | yes | varies |
| Every finding verified with a concrete failing case | yes | varies |
| Your code stays with your own model provider | yes | sent to the vendor |
| Customise the rules | edit one Markdown file | vendor settings |

## Contributing

Issues and pull requests are welcome. The whole reviewer is one file, `skills/greptile-lite/SKILL.md`: change a lens, a severity rule or the comment format and open a PR. If it saved you a subscription, a star helps others find it.

## License

MIT

