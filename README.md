# claude-skills

My personal collection of custom [Claude Code](https://claude.com/claude-code) skills.

Skills live under `~/.claude/skills/` and are invoked via slash commands (e.g. `/commit-and-push`). This repo is the canonical source for mine — clone or symlink the skill folders into `~/.claude/skills/` to use them.

## Skills

| Skill | Invocation | What it does |
| --- | --- | --- |
| [commit-and-push](commit-and-push/SKILL.md) | `/commit-and-push` | Stage unstaged changes into meaningful, well-scoped commits and push to the remote. |
| [open-pr](open-pr/SKILL.md) | `/open-pr` | Create a GitHub PR for the current branch with a structured summary and test plan. |
| [self-review](self-review/SKILL.md) | `/self-review [pr or branch]` | Senior-engineer self code review of a PR or branch — issues, suggestions, notes. |
| [deep-talk](deep-talk/SKILL.md) | `/deep-talk` | Relentless interview that walks the full decision tree of a plan or design until shared understanding. |

## Install

Clone this repo and symlink each skill into `~/.claude/skills/`:

```sh
git clone git@github.com:alikayhan/claude-skills.git ~/src/claude-skills
mkdir -p ~/.claude/skills
for skill in commit-and-push open-pr self-review deep-talk; do
  ln -s ~/src/claude-skills/$skill ~/.claude/skills/$skill
done
```

Restart Claude Code and the skills will show up as slash commands.

## Adding a new skill

1. Create `skill-name/SKILL.md` with YAML frontmatter (`name`, `description`, optional `argument-hint`, `disable-model-invocation`, `user-invocable`).
2. Symlink it into `~/.claude/skills/`.
3. Commit and push.
