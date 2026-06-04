# Contributing to git-why

Thanks for wanting to improve git-why! Contributions are welcome and appreciated.

## What to contribute

### Better reasoning heuristics
The core of the skill is the reasoning approach in `SKILL.md`. If you've found a class of diffs that git-why explains poorly, open an issue with an example and a better explanation. PRs that add new heuristics to the "Reasoning approach" section are very welcome.

### More examples
The `SKILL.md` example section teaches Claude by demonstration. More high-quality input → output pairs make the skill significantly better. Good examples:
- Cover a diff pattern not already shown (e.g. lock removal, feature flag addition, schema migration)
- Show the *why* clearly, not just *what*
- Are anonymised (no real names, company names, or internal URLs)

### Language / format support
Currently focused on unified diffs and standard `git` output. PRs to handle SVN, Mercurial, or Perforce diff formats welcome.

## How to contribute

1. Fork the repo
2. Edit `SKILL.md` directly — it's just Markdown
3. Test your change by uploading the updated skill to Claude.ai and running it against a real diff
4. Open a PR with a before/after example showing the improvement

## Reporting issues

Open a GitHub issue with:
- The diff or git output you pasted
- What git-why said
- What it should have said instead

That's it — no CLA, no lengthy process.
