---
name: pr-bot-review
description: >-
  Triage automated-reviewer (GitHub Copilot and other review bots) comments on a
  pull request. Fetches the bot review comments, grounds each one against the
  actual code, proposes local fixes for the valid ones, then asks whether you or
  the skill should reply to and resolve the threads. Use when checking a PR for
  Copilot / bot review feedback.
allowed-tools: >-
  Read, Edit, Grep, Glob,
  Bash(gh pr view *), Bash(gh pr diff *), Bash(gh repo view *),
  Bash(gh api repos/*/pulls/*), Bash(gh api graphql *),
  Bash(git diff *), Bash(git log *), Bash(git status *)
---

# pr-bot-review

Triage automated-reviewer comments on a PR, propose fixes for the valid ones, and
(with confirmation) reply to / resolve the threads.

This skill targets **bot reviewers** — GitHub Copilot and any other automated
reviewer — identified by `user.type == "Bot"` in the GitHub API. It deliberately
ignores human reviewers.

## Ground rules

- **Never commit or push.** Apply fixes to the working tree only and show the
  diff. Committing is the user's separate decision.
- **Replies and thread resolution are outward-facing writes** to an AA-visible
  PR. They happen only in Phase 3, only after the user explicitly chooses to let
  the skill do them.
- **Ground every comment in the real code** before judging it. Bot comments are
  noisy and sometimes wrong; do not trust the comment text on its own.
- Use gh's literal `{owner}/{repo}` placeholders in `gh api` paths so the skill
  works in any repo without a separate owner/name lookup.

## Phase 0 — Resolve the target PR and fetch

1. **Determine the PR number.**
   - If the user passed a number as an argument, use it.
   - Otherwise resolve the PR for the current branch:
     `gh pr view --json number,title,headRefName,baseRefName`.
   - If neither yields a PR, stop and tell the user (e.g. "no PR found for the
     current branch — pass a PR number").

2. **Fetch the three sources** (the GraphQL call is the only one that exposes
   `isResolved`, `isOutdated`, and the thread `id` needed to resolve later):
   - Inline review comments (REST):
     `gh api repos/{owner}/{repo}/pulls/<n>/comments --paginate`
   - Review summaries (REST):
     `gh api repos/{owner}/{repo}/pulls/<n>/reviews --paginate`
   - Review threads (GraphQL) — fetch `id`, `isResolved`, `isOutdated`, and each
     thread's comments (`databaseId`, `author.login`, `path`, `line`,
     `originalLine`, `body`):

     ```bash
     gh api graphql -f query='
       query($owner:String!,$repo:String!,$pr:Int!){
         repository(owner:$owner,name:$repo){
           pullRequest(number:$pr){
             reviewThreads(first:100){
               nodes{
                 id isResolved isOutdated
                 comments(first:50){
                   nodes{
                     databaseId path line originalLine body
                     author{ login __typename }
                   }
                 }
               }
             }
           }
         }
       }' -f owner=<owner> -f repo=<repo> -F pr=<n>
     ```

3. **Filter and partition.**
   - Keep only threads whose comments come from a **bot** author
     (`__typename == "Bot"`, equivalently REST `user.type == "Bot"`).
   - **Drop threads already marked `isResolved`** — they're settled; don't
     re-litigate them.
   - Set aside **`isOutdated`** threads into a separate "stale" bucket (their
     line numbers no longer map to current code).
   - If nothing survives, report that and stop.

## Phase 1 — Ground and triage

For each surviving (non-stale) bot comment:

1. Open the referenced code at `path:line` in the working tree and read enough
   surrounding context to judge the comment on its merits.
2. Classify it:
   - 🟢 **Real issue** — a genuine bug/correctness/security/maintainability
     problem that should be fixed.
   - 🟡 **Valid nit** — a legitimate but optional style/clarity point.
   - 🔴 **Wrong / out of scope / already handled** — incorrect, irrelevant, or
     the code already does the right thing. Give a one-line reason.
3. Note whether the comment carries a ```suggestion block (those are
   near-verbatim applies; the rest need reasoning).

Present a **ranked report**: 🟢 first, then 🟡, then 🔴 (with reasons), then a
separate **Stale (verify manually)** section listing the outdated threads —
those are **not** auto-fixed.

The report must end with an **action plan**: for each 🟢 and 🟡, a one-or-two-line
sketch of the intended fix (file, approach, expected blast radius). Then **stop
and wait for the user's approval.** Do not edit any file before the user has
approved the plan — approval may be full ("go"), partial ("only #1"), or a
redirection; apply exactly what was approved and nothing more.

## Phase 2 — Apply the approved fixes locally

1. Apply only the fixes the user approved in the Phase 1 action plan. For
   comments with a ```suggestion block, apply it closely; for reasoned ones,
   implement the fix in the codebase's style.
2. Follow the repo's conventions and run its own format/lint/test scripts where
   relevant (never raw `npx` in repos that forbid it).
3. **Do not commit or push.** Show the resulting `git diff` for review.
4. 🔴 items get no code change.

## Phase 3 — Reply and resolve (only with confirmation)

Summarize the intended GitHub writes, then ask the user how to proceed per the
two buckets:

- **Addressed threads** (🟢 / accepted 🟡 that you fixed): a short reply noting it
  was addressed, then resolve the thread.
- **🔴 threads** (bot was wrong / out of scope): a short reply explaining why no
  change was made. Leave these **unresolved** unless the user says otherwise.

Ask explicitly: **"Reply/resolve these yourself, or should I?"**

- If the user will do it themselves: output the per-thread reply text and the
  thread IDs so they can act, and stop.
- If the skill should do it, perform the writes:
  - **Reply** to a comment (REST):
    ```bash
    gh api repos/{owner}/{repo}/pulls/<n>/comments/<comment_databaseId>/replies \
      -f body='<reply text>'
    ```
  - **Resolve** a thread (GraphQL — REST cannot resolve):
    ```bash
    gh api graphql -f query='
      mutation($threadId:ID!){
        resolveReviewThread(input:{threadId:$threadId}){
          thread{ id isResolved }
        }
      }' -f threadId='<thread node id>'
    ```
  - Report back which threads were replied to and resolved.

## Notes

- `allowed-tools` here pre-approves these commands so the triage runs without
  permission prompts; it does **not** restrict the toolset. The real gate on the
  outward-facing writes is the Phase 3 confirmation, not a permission prompt.
- `gh api graphql` covers both the read query and the resolve mutation — they
  share a command shape and can't be separated by permission rules, which is why
  the Phase 3 confirmation is the load-bearing check.
