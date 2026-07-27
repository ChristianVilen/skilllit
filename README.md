# skilllit

Agent skills and subagent definitions for Claude Code.

## Install

Both directories are consumed as symlinks, so edits here are live immediately — no copy
step, no sync.

```sh
ln -s "$PWD/skills" ~/.claude/skills
ln -s "$PWD/agents" ~/.claude/agents
```

## Layout

- `skills/` — one directory per skill, each with a `SKILL.md` carrying YAML frontmatter
  (`name`, `description`). Large supporting material goes in `references/` and is loaded
  on demand rather than inlined into `SKILL.md`.
- `agents/` — subagent definitions as markdown with frontmatter (`name`, `description`,
  `tools`, optional `model`).

Skills marked `disable-model-invocation: true` are user-invoked only, via `/<name>`. They
won't be offered automatically.

## Dependencies

Some skills shell out to tools you need to install separately:

- `git-surgeon` — https://github.com/raine/git-surgeon
- `workmux` — used by the `workmux`, `worktree`, and `coordinator` skills
- `playwright-cli` — used by the `playwright-cli` skill

## Sources

- https://github.com/obra/superpowers
- https://github.com/mattpocock/skills
