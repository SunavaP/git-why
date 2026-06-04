# git-why

> You've been staring at a line of code for 20 minutes.  
> `git blame` says it was changed 8 months ago by someone who left the company.  
> The commit message says `"fix stuff"`.  
> **git-why tells you why it's actually there.**

A Claude AI skill that explains the *intent* behind code changes — not just who changed it and when, but the production incident, edge case, or design decision that made it necessary.

---

## Install

1. Download [`git-why.skill`](https://github.com/yourusername/git-why/releases/latest/download/git-why.skill)
2. Go to **Settings → Skills** in [Claude.ai](https://claude.ai)
3. Upload the `.skill` file
4. Done — Claude now knows how to run `git-why` for you

---

## Usage

Paste any of the following into Claude and ask *"why?"*:

- `git show <commit-hash>`
- `git diff HEAD~1`
- `git blame <file>` output
- A raw code snippet you don't understand

### Example

**You paste:**
```
commit a3f8c21
Author: Sarah Chen
fix: token validation race condition under high load

-func validateToken(tok string) error {
+func validateToken(ctx context.Context, tok string) error {
-    time.Sleep(retryDelay)
+    select {
+    case <-ctx.Done():
+        return ctx.Err()
+    case <-time.After(retryDelay):
+    }
```

**Claude explains:**

> **The change:** Token validation now accepts a context so it can be cancelled by the caller.
>
> **Why it exists:** There was a concurrency bug where token validation blocked goroutines indefinitely under high traffic. The old code used `time.Sleep` which can't be interrupted — if a request timed out at the HTTP layer, the goroutine kept running anyway, leaking resources and causing memory pressure in the auth service during traffic spikes.
>
> **What it prevents:** Goroutine leaks. Now if the parent request is cancelled or times out, validation stops immediately via `ctx.Done()`.
>
> **What to look for next:**
> - All callers of `validateToken` need to pass a context with a deadline
> - Check for other `time.Sleep` calls in the auth package with the same issue

---

## What it covers

| Input | What you get |
|---|---|
| `git diff` | Why the change was made, what broke before |
| `git blame` output | Intent behind a specific line |
| `git show <hash>` | Plain-English commit explanation |
| `git log` snippet | Summary of what changed and why |
| Raw code | Why it was written this way |

---

## How it works

`git-why` is a [Claude Skill](https://claude.ai/skills) — a set of instructions that teach Claude to reason like a senior engineer when reading git output.

When you paste a diff, it:
1. Reads the **diff shape** (error handling added → something failed silently)
2. Mines the **commit message** for intent signals (`fix:`, `revert:`, `perf:`)
3. Reads **variable and function names** as design decisions
4. Infers from **what's missing** (no error handling before → production incident)
5. Produces a structured explanation: the change, why it exists, what it prevents, and what to check next

It won't just say *"this adds a parameter to the function."*  
It says *"this adds a context parameter so the caller can cancel the operation — previously the function could block indefinitely."*

---

## Philosophy

Most code archaeology tools tell you *what* changed. `git-why` tells you *why* — because that's the question you're actually asking at 2am when something broke.

The skill is designed to reason the way a senior engineer does: infer intent from naming, diff patterns, commit message conventions, and what was deliberately left out.

---

## Contributing

The skill lives in [`SKILL.md`](./SKILL.md). Contributions welcome:

- Better reasoning heuristics
- More example input/output pairs
- Support for specific languages or diff formats (SVN, Mercurial, etc.)

Open a PR or file an issue.

---

## License

MIT
