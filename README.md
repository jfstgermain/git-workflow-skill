# git-workflow-skill

A Pi skill for AI-drafted, human-reviewed Git work.

It drafts branch names, commit messages, and GitHub pull request descriptions from the actual diff, then hands the final step back to you. The model already selected in your Pi session writes the text, so there is no provider configuration and no separate CLI.

## Install

```bash
pi install git:git@github.com:jfstgermain/git-workflow-skill
```

## What it does

- Drafts a `feature/` or `bugfix/` branch name, including the Jira ticket, and waits for approval before `git switch -c`.
- Stores the current Jira ticket in repo-local git config (`git-workflow.ticket`) and reuses it in every branch name until changed.
- Accepts a per-branch ticket override ("branch this on IGIA-312") without changing the stored default.
- Drafts a gitmoji commit message from the staged diff, in French with English technical terms.
- Drafts a PR title and body, then creates it with `gh pr create`.
- Reviews before push with `git diff @{u}..HEAD`.

## Prompts the skill handles

The skill responds to natural language. Exact wording is not required; these are the request types it recognises.

### Staging

| Request | Behaviour |
|---|---|
| "stage these files" | `git add` the named files only |
| "stage part of this file" | Opens `git add -p` in the overlay, or asks you to run it |

### Commits

| Request | Behaviour |
|---|---|
| "commit this" / "commit these changes" | Stages the intended changes, drafts a gitmoji message, opens the review, commits after you save |
| "commit only these files" | Stages only the named files, then the same flow |
| "write a commit message" | Drafts the message only, nothing is committed |
| "make several commits from this" | One logical change per commit, each message drafted and reviewed |

### Branches

| Request | Behaviour |
|---|---|
| "create a branch for this" | Drafts a `feature/` or `bugfix/` name with the ticket, waits for approval, runs `git switch -c` |
| "name this branch" | Drafts the name only |
| "branch this on IGIA-312" | Per-branch ticket override; the stored default does not change |

### Jira ticket

| Request | Behaviour |
|---|---|
| "set the ticket to IGIA-290" / "the ticket is IGIA-290" | Stores IGIA-290 as the repo default |
| "use IGIA-290 for this branch" / "this one is IGIA-290" | Applies to that branch only |
| "clear the ticket" / "no ticket" | Removes the stored default |

### Pull requests

| Request | Behaviour |
|---|---|
| "open a PR" / "create the pull request" | Checks `gh auth`, drafts title and body, opens the body for editing, creates a draft PR |
| "open a PR, ready for review" | Same, but not a draft |
| "push and open a PR" | Pushes the branch first, then the PR flow |
| "write the PR description" | Drafts title and body only |

### Push and review

| Request | Behaviour |
|---|---|
| "push this" | Shows what will leave, then pushes after approval |
| "what am I about to push?" | Shows `git diff @{u}..HEAD` |
| "review the changes" | Opens the diff, staged or unstaged |

### Out of scope

The skill stops and asks instead of acting on these:

- Interactive auth (`gh auth login`) and editor sessions, which must run in a terminal or the overlay.
- Mid-rebase or conflicted repositories. Resolve first.
- History rewrites on pushed, reviewed, or shared branches.
- Unrelated git operations such as rebase, cherry-pick, or stash.

### Model-facing prompt templates

The prompts the skill sends to the model live in `references/prompts.md`:

- **Commit**: message only, gitmoji plus semantic prefix, present tense, what and why, most important changes, title at or under 50 characters, body wrapped at 72, no line starting with `#`, French with English technical terms.
- **Branch**: `bugfix/` or `feature/` prefix, then the ticket, then a short English kebab-case description, output only the name.

Worked examples are in `references/templates.md`.

## Conventions

- Branch: `feature/<TICKET>-<kebab-description>` or `bugfix/<TICKET>-<kebab-description>`, English, ticket uppercase, for example `feature/IGIA-290-add-export`.
- Commit: gitmoji plus semantic prefix (`✨ feat`, `🐛 fix`), present tense, title at or under 50 characters, body wrapped at 72, no line starting with `#`, French with English technical terms.
- PR body: Résumé, Motivation, Tests, referencing the Jira ticket.

## Review handoff

The user always performs the final review. The agent never commits behind your back.

- Commits need an editor. With [pi-interactive-shell](https://www.npmjs.com/package/pi-interactive-shell) installed, the agent opens `git commit -v` in a TUI overlay through the `interactive_shell` tool, and you inspect the diff, edit the message, and save. Without it, the agent prints `git commit -v -e -F /tmp/git-workflow.msg` for you to run in a terminal.
- Pushes do not need a terminal, so the agent may run `git push` once you explicitly approve it.

The approval gate lives in git's own UI. See `skills/git-workflow/SKILL.md` for the full rules.

## Layout

```text
git-workflow-skill/
├── package.json
├── README.md
└── skills/
    └── git-workflow/
        ├── SKILL.md
        └── references/
            ├── prompts.md
            └── templates.md
```
