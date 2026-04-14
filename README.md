# skills

My personal collection of reusable skills for AI coding agents.

Each skill is a single markdown file containing a prompt plus a small bit of YAML frontmatter describing when and how to use it. The skills are plain prompts — portable across any coding agent that supports custom instructions, slash commands, or user-defined skills. Point your agent at the file (or copy its contents into whatever "custom instruction" slot your tool provides) and you're done.

## Skills

| Skill | What it does |
| --- | --- |
| [commit-and-push](commit-and-push/SKILL.md) | Stage unstaged changes into meaningful, well-scoped commits and push to the remote. |
| [open-pr](open-pr/SKILL.md) | Create a GitHub PR for the current branch with a structured summary and test plan. |
| [self-review](self-review/SKILL.md) | Senior-engineer self code review of a PR or branch — issues, suggestions, notes. |
| [deep-talk](deep-talk/SKILL.md) | Relentless interview that walks the full decision tree of a plan or design until shared understanding. |

## Install

Clone the repo and wire up whichever skills you want however your agent expects them:

```sh
git clone git@github.com:alikayhan/skills.git
```

Each skill is self-contained in its own folder — copy, symlink, or reference the `SKILL.md` file from wherever your agent loads custom prompts.

## Adding a new skill

1. Create `skill-name/SKILL.md` with YAML frontmatter (`name`, `description`, and any agent-specific fields you need) followed by the prompt body.
2. Add a row to the table above.
3. Commit and push.
