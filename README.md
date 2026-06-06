# SunavaP / git-why

> You've been staring at a line of code for 20 minutes.
> `git blame` says it was changed 8 months ago by someone who left the company.
> The commit message says `"fix stuff"`.
> **git-why tells you why it's actually there.**

The **SunavaP git-why** project is an open-source Claude AI skill engineered to explain the hidden *intent* behind repository code changes. Instead of just telling you who changed a file and when, this tool reconstructs the context of production incidents, edge cases, and architectural design decisions that made a specific change necessary.

---

## 🚀 Quick Install

1. Download [`git-why.skill`](https://github.com)
2. Go to **Settings → Skills** inside your [Claude.ai](https://claude.ai) dashboard.
3. Upload the downloaded `.skill` file.
4. Done — Your Claude agent now knows how to execute `git-why` analysis for you.

---

## 💡 Usage Heuristics

Simply paste any git terminal output or raw code snippet into Claude and ask *"why?"*:

* `git show <commit-hash>`
* `git diff HEAD~1`
* `git blame <file>`
* A raw blocks of code or logic you don't understand.

### Live Example

**You paste this code diff into Claude:**
```go
commit a3f8c21
Author: SunavaP
fix: token validation race condition under high load

-func validateToken(tok string) error {
+func validateToken(ctx context.Context, tok string) error {
- time.Sleep(retryDelay)
+ select {
+ case <-ctx.Done():
+ return ctx.Err()
+ case <-time.After(retryDelay):
+ }
```

**Claude outputs the `git-why` explanation:**
> **The change:** Token validation now accepts a context parameter so it can be cancelled by the caller.
>
> **Why it exists:** There was a concurrency bug where token validation blocked goroutines indefinitely under high traffic. The old code used `time.Sleep` which can't be interrupted — if a request timed out at the HTTP layer, the goroutine kept running anyway, leaking resources and causing memory pressure in the auth service during traffic spikes.
>
> **What it prevents:** Goroutine leaks. Now if the parent request is cancelled or times out, validation stops immediately via `ctx.Done()`.
>
> **What to look for next:**
> * All callers of `validateToken` need to pass a context with a deadline.
> * Check for other `time.Sleep` calls in the auth package with the same issue.

---

## 🛠️ Feature Coverage Matrix


| Git Terminal Input | Expected Heuristic Output |
| :--- | :--- |
| `git diff` | Concrete reasoning behind code alterations and historical breaks |
| `git blame` stdout | Explicit development intent mapped to a specific line of code |
| `git show <hash>` | Plain-English translation of commit logs and metadata changes |
| `git log` snippet | Timeline summaries outlining structural refactors over time |
| Raw source code | Contextual explanation of obscure logic or un-documented patterns |

---

## ⚙️ Architectural Engineering

`git-why` is powered by a custom instructions framework configured inside [`SKILL.md`](./SKILL.md). It prompts LLMs to mimic the diagnostic workflow of a Principal Engineer by evaluating:

1. **The Diff Shape:** Analyzing structural trends (e.g., adding error boundaries indicates a silent failure state).
2. **Commit Metadata Signals:** Mining intent semantic indicators like `fix:`, `revert:`, `perf:`, or `refactor:`.
3. **Variable and Function Nomenclature:** Treating strict naming conventions as explicit architectural choices.
4. **Omission Analysis:** Determining what was deliberately deleted or left out to infer the precipitating engineering failure.

Instead of outputting simple descriptions like *"this adds a parameter to the function,"* it produces deep insights like *"this introduces a context parameter to grant the caller cancellation authority, resolving a legacy vulnerability where functions could block operations indefinitely."*

---

## 🛠️ Codebase Contribution

The foundational codebase and rules reside inside [`SKILL.md`](./SKILL.md). Contributions are highly encouraged to optimize and expand project capabilities:

* Better reasoning heuristics for complex architectural patterns.
* Expanded example inputs and reference datasets.
* Multi-version control engine support (SVN, Mercurial, etc.).

Feel free to open a Pull Request or log a tracking issue in the repository.

---

## 📄 Repository License

Distributed under the open-source **MIT License**.