# skills

Agent skills and subagent definitions for Claude Code.

## Install

Both directories are consumed as symlinks, so edits here are live immediately — no copy
step, no sync.

```sh
ln -s "$PWD/skills" ~/.claude/skills
ln -s "$PWD/agents" ~/.claude/agents
```

The links are absolute. Moving or renaming this repo breaks both silently — skills and
agents just stop appearing, with no error. Re-point them afterwards:

```sh
ln -sfn "$PWD/skills" ~/.claude/skills
ln -sfn "$PWD/agents" ~/.claude/agents
```

## Layout

- `skills/` — one directory per skill, each with a `SKILL.md` carrying YAML frontmatter
  (`name`, `description`).
- `agents/` — subagent definitions as markdown with frontmatter (`name`, `description`,
  `tools`, optional `model`).

Skills marked `disable-model-invocation: true` are user-invoked only, via `/<name>`. They
won't be offered automatically.

## Adding a skill

```
skills/<name>/
├── SKILL.md
└── references/        # optional
```

`SKILL.md` needs frontmatter with `name` matching the directory, and a `description`.

The `description` is the only thing the agent sees when deciding whether to invoke the
skill, so write it for matching, not for humans: name the symptoms and the phrasings that
should trigger it. A vague description means the skill never fires.

Keep `SKILL.md` short. Anything long belongs in `references/`, with `SKILL.md` acting as a
router that says which file to read for which situation — the agent then loads one file
instead of the whole body. `react-best-practices/` is the worked example: an 80-line router
over eight reference files, where the old single-file version pulled 2116 lines into context
for any React question.

## Dependencies

Some skills shell out to tools you need to install separately:

- `git-surgeon` — https://github.com/raine/git-surgeon
- `workmux` — used by the `workmux`, `worktree`, and `coordinator` skills
- `playwright-cli` — used by the `playwright-cli` skill

## Sources

- https://github.com/obra/superpowers
- https://github.com/mattpocock/skills
