# Agent Behavior

- After finishing editing code, run linting and formatting scripts if available. Note: do not run between edits if working on a multifile task
- Initiate grill-me skill if given task seems broad
- When tasked to write documentation, use humanizer skill
- When a command fails, show the error output and explain what went wrong before retrying
- Always show file content for review before creating new files in unfamiliar areas
- Before creating a new utility function, search the codebase for existing implementations
- Do not modify files outside the scope of the current task unless explicitly asked
- Do not automatically add tests unless explicitly requested by the user. Only write test code when the user specifically asks for tests

## Shell Commands

- NEVER use `cd` to change directory before running a command. ALWAYS use the `working_dir` parameter on the shell tool instead. This is a hard rule, not a preference.
  - WRONG: `cd /path/to/dir && npm run build`
  - RIGHT: shell(command="npm run build", working_dir="/path/to/dir")
- Prefer tool-native operations (file read/write tools) over shell commands when possible

## Git Operations

- Use the `git-surgeon` skill for all staging, committing, unstaging, discarding, and hunk-level operations
- Never use interactive git commands (e.g. `git add -p`, `git rebase -i`)
- Always show changes and get confirmation before committing or pushing

## Package Manager

- Always prefer scripts defined in the project's manifest (package.json, Makefile, etc.) over running tools directly
- Check the manifest first before using npx or equivalent

## Allowed commands

### Allowed

- Read-only filesystem operations (ls, cat, head, tail, find, etc.)
- Git read operations (status, log, diff, branch --list)
- Project-defined scripts (npm scripts, make targets, etc.)
- Docker read operations (ps, logs, inspect)
- jq for JSON processing
- Test execution via project scripts

### Requires confirmation

- File modifications (cp, mv, rename)
- Dependency installation (npm ci, pip install, etc.)
- Infrastructure operations (CDK, Terraform, etc.)
- git commit / git-surgeon commit (always show changes and ask for permission first)

### Never allowed

- rm -rf or recursive deletes
- git push (without explicit permission)
- Destructive database operations
- Installing global packages
