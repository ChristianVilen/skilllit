---
name: reviewing-code-comments
description: Use when a user asks to review, remove, or tighten unnecessary, stale, verbose, or AI-sounding code comments in a branch, PR, commit range, staged diff, or working tree.
---

# Reviewing Code Comments

Keep only comments that add information the code cannot express clearly. Repository rules override this default.

## Workflow

1. **Read the rules.** Check applicable `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, style guides, and nested instructions. Skip generated, vendored, and deletion-only files.

2. **Pin the target.** Honor any base, range, or PR target from the user. Otherwise use the PR base when available, then the remote default. Never replace an explicit target with `origin/HEAD`. Confirm the target resolves. Fetch a remote target when allowed; otherwise report that it may be stale. Ask only when the target is still ambiguous.

   Match the diff to the request:

   - Branch or PR commits: `git diff <base>...HEAD`
   - Staged changes: `git diff --cached`
   - Unstaged changes: `git diff`
   - WIP branch: include its commits, staged and unstaged changes, and in-scope untracked source files unless the user narrows the scope.

3. **Follow authorization.** Review, check, and audit requests are read-only. Clean up, fix, remove, and tighten requests authorize comment edits in scope. Ask only when the request is unclear.

4. **Read context.** Use the diff to find added or modified comment blocks, then read each complete block and its surrounding logic. Check available code, callers, tests, and requirements before using `VERIFY`. Treat unchanged comments beside changed code as separate stale-comment candidates.

5. **Assign a verdict.**

| Verdict | Use when |
| --- | --- |
| **KEEP** | The comment gives a reason, trade-off, warning, or contract the code cannot express. Contracts include units, sentinel values, invariants, ownership, lifetime, ordering, concurrency, compatibility, and error behavior. Keep directives and documentation required by repository rules or tooling. Keep a `TODO` only when it is actionable. |
| **REWRITE** | The information matters and is known, but the comment is unclear, too long, partly obvious, or relies on an inaccessible reference. State the fact directly; keep a durable reference only as supporting context. |
| **DROP** | The comment restates code, labels a section, duplicates a name or type, makes a vague `TODO`, or adds no useful fact. |
| **VERIFY** | The comment may be stale, contradicts the code or requirements, or cannot be judged without missing information. Do not guess or edit it until the conflict is resolved. |

6. **Tighten wording.** Use plain, direct sentences. Remove filler, promotional terms, forced lists, repeated em dashes, and vague praise. A word alone is not proof of AI writing. Keep comments factual.

## Output and edits

By default, report `DROP`, `REWRITE`, and `VERIFY` findings by file, plus reviewed and kept counts. Include individual `KEEP` results when the user asks for an exhaustive classification.

```text
file:line   VERDICT   reason
            proposed text (REWRITE only)
```

For a cleanup, apply `DROP` and `REWRITE`, leave `VERIFY` unchanged, and report edits and open questions. Do not change unrelated code or staging unless asked. Report files that now have both staged and unstaged changes.
