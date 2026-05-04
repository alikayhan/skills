# skills

My personal collection of reusable skills for AI coding agents.

Each skill is a single markdown file containing a prompt plus a small bit of YAML frontmatter describing when and how to use it. The skills are plain prompts — portable across any coding agent that supports custom instructions, slash commands, or user-defined skills. Point your agent at the file (or copy its contents into whatever "custom instruction" slot your tool provides) and you're done.

## Skills

| Skill | What it does | Install |
| --- | --- | --- |
| [commit-and-push](commit-and-push/SKILL.md) | Stage unstaged changes into meaningful, well-scoped commits and push to the remote. | `npx skills@latest add alikayhan/skills -s commit-and-push` |
| [open-pr](open-pr/SKILL.md) | Create a GitHub PR for the current branch with a structured summary and test plan. | `npx skills@latest add alikayhan/skills -s open-pr` |
| [self-review](self-review/SKILL.md) | Senior-engineer self code review of a PR or branch — issues, suggestions, notes. | `npx skills@latest add alikayhan/skills -s self-review` |
| [refine-code](refine-code/SKILL.md) | Review changed code for reuse, quality, and efficiency, then fix any issues found. | `npx skills@latest add alikayhan/skills -s refine-code` |
| [deep-talk](deep-talk/SKILL.md) | Relentless interview that walks the full decision tree of a plan or design until shared understanding. | `npx skills@latest add alikayhan/skills -s deep-talk` |

## Install

Install everything in this repo at once:

```sh
npx skills@latest add alikayhan/skills
```

Or pick individual skills from the table above. Add `-g` to install globally (user-level) instead of into the current project, and `-a <agent>` to target a specific agent (`-a '*'` for all detected agents). See `npx skills@latest --help` for the full flag list.

If you'd rather not use the installer, just clone the repo and reference each `SKILL.md` from wherever your agent loads custom prompts:

```sh
git clone git@github.com:alikayhan/skills.git
```

## Adding a new skill

1. Create `skill-name/SKILL.md` with YAML frontmatter (`name`, `description`, and any agent-specific fields you need) followed by the prompt body.
2. Add a row to the table above.
3. Commit and push.
