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
