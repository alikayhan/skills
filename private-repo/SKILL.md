---
name: private-repo
version: 1.0.0
description: Bootstrap the current folder as a private GitHub repo — extensive .gitignore, agent-readable README, git init on main, initial commit, push, and squash-only merge settings. Use when asked to start/create a private repo for a project.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional repo name] [optional initial commit message]"
tags:
  - coding
  - git
---

# Private Repo Bootstrap

Turn the current working directory into a pushed, configured, private GitHub repository. Run the steps in order; skip any step whose result already exists. Never overwrite files the user already wrote.

## 0. Preflight

- `gh auth status` must show a logged-in account with `repo` scope; abort with the fix (`gh auth login`) if not.
- `git config user.name` / `user.email` must resolve (global or local); abort with the fix if not.
- Repo name: first argument if given, else the folder name, kebab-cased.

## 1. .gitignore (create only if missing)

Write an extensive .gitignore assembled from these blocks:

- **Always**: macOS (`.DS_Store`, `._*`, `.Spotlight-V100`, `.Trashes`), secrets (`.env`, `.env.*`, `!.env.example`, `*.pem`, `*.key`, `*.p12`, `credentials*.json`, `secrets*`), IDE (`.vscode/`, `.idea/`, `*.swp`, `*~`), logs and scratch (`*.log`, `tmp/`, `scratch/`).
- **By detected stack** (check which file types exist): Python (`__pycache__/`, `*.py[cod]`, `.venv/`, `venv/`, `*.egg-info/`, `dist/`, `build/`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`, `.coverage`, `htmlcov/`, `.ipynb_checkpoints/`), Node (`node_modules/`, `npm-debug.log*`, `yarn-error.log*`, `.next/`, `out/`), Rust (`target/`), Go (`bin/`), Swift/Xcode (`.build/`, `DerivedData/`, `*.xcuserstate`), Terraform (`.terraform/`, `*.tfstate*`).
- **Generated artifacts**: local databases (`*.db`, `*.sqlite*`, `*.db-journal`, `*.db-wal`, `*.db-shm`) and any directory in this project that holds regenerable output (rendered reports, build output, downloaded data). Identify these by inspecting the project, and note in the README that they are local.

## 2. README.md (create only if missing)

Write an agent-readable README: a reader (human or agent) must be able to operate the project from it alone.

Required sections: one-paragraph purpose · requirements/dependencies with exact paths and key names (never the key values) · commands with copy-pasteable invocations · project layout (file → one-line role) · known limits or caveats. Add a status/roadmap table if the project has phases.

Style: concrete over promotional; exact paths, commands, and env var names; no AI/tool/agent attribution of any kind.

## 3. Init, commit, push

```sh
git init -b main                # skip if already a repo; if on master, git branch -m master main
git add -A
git commit -m "<message>"
gh repo create <name> --private --source . --push
```

- Initial commit message: second argument if given, else `init: start <project name>'s journey` — mind the possessive: a plural name ending in s takes a bare apostrophe (`early birds` → `early birds'`).
- Plain message only — no Co-Authored-By, no generated-with trailers.
- Before committing, eyeball `git status --short` for anything that looks like a secret or a large generated file; fix the .gitignore first if so.
- If the repo already exists with an `origin` remote, skip creation and just push + continue to settings.

## 4. Repository settings

```sh
gh api -X PATCH repos/<owner>/<name> \
  -F allow_merge_commit=false -F allow_rebase_merge=false -F allow_squash_merge=true \
  -f squash_merge_commit_title=PR_TITLE -f squash_merge_commit_message=BLANK
```

Result: default branch `main`, squash merges only, squash commit message is the PR title and nothing else. Print the returned JSON fields to confirm, then report the repo URL.
