---
name: git-workflow
description: Prepare and review Git work with AI-drafted, human-edited text. Use when asked to commit changes, create or name a branch, push, or open or update a GitHub pull request, or when asked to write a commit message, branch name, or PR description.
license: MIT
metadata:
  version: "0.1.0"
---

# Git Workflow

Draft branch names, commit messages, and pull request descriptions from the actual diff, then hand the final step back to the user for review. The model already selected in the current Pi session writes the text. There is no separate provider configuration.

## The core rule

Draft, show, hand off. The user performs the final review and applies the change.

- Never finalize a commit on your own. A commit opens an editor. Open it for the user and let them review, edit, and save.
- Never push without explicit approval. A push does not need a terminal, so you may run it once approved.

Pi's bash tool cannot run interactive programs. `git commit -v`, `git add -p`, `lazygit`, and difftools need a real TTY, and fail or block through `bash`. Run them with the `interactive_shell` tool from pi-interactive-shell (call `enable_interactive_shell` once if the tool is not yet available), or hand the command to the user to run in a terminal outside Pi.

## When to use

- "commit this", "commit only these files", "write a commit message"
- "create a branch for this", "name this branch"
- "open a PR", "push and open a PR", "write the PR description"
- "what am I about to push?"

## When not to use

- The user already wrote the message. Run the command as given.
- The repository is mid-rebase or conflicted. Resolve first.
- History rewrites on pushed, reviewed, or shared branches. Ask before touching.

## Jira ticket

The ticket is stored in repo-local git config, so it survives across tool calls and sessions until it changes.

- Read: `git config --local --get git-workflow.ticket`
- Set or change: `git config --local git-workflow.ticket IGIA-290`
- Clear: `git config --local --unset git-workflow.ticket`

When the user says "use ticket IGIA-290", "set the ticket to IGIA-290", or "this is for IGIA-290", run the set command and confirm. When they say "clear the ticket" or "no ticket", unset it. The ticket then applies to every branch created for the rest of the session and beyond, until it is changed.

Validate the format before storing: uppercase project key, hyphen, number, matching `^[A-Z][A-Z0-9]+-[0-9]+$`.

Environment variables do not persist between tool calls, which is why the ticket lives in git config and not in the shell.

### Per-branch override

Separate changing the default from overriding one branch:

- "set the ticket to IGIA-312" or "the ticket is now IGIA-312" changes the stored default for every future branch.
- "use IGIA-312 for this branch", "branch this on IGIA-312", or "this one is IGIA-312" applies to the single branch created by that request only. Do not write it to git config.

After an override, the next branch without one falls back to the stored default. Validate an override against the same format before using it.

The pull request references the ticket embedded in the branch name when present, falling back to the stored default.

## Workflow

### 1. Inspect

```bash
git status -sb
git diff --staged
git diff
git log --oneline -10
```

Base the wording on the actual diff, not on file names or the conversation alone.

### 2. Branch

Only when the user asks for one, or when the repository works one branch per task.

- Read the ticket with `git config --local --get git-workflow.ticket`.
- Use the branch prompt in `references/prompts.md`. Prefix with `bugfix/` when the change primarily fixes bugs or corrects behavior, and `feature/` when it primarily adds functionality or enhancements, then the ticket and a short English kebab-case description, for example `feature/IGIA-290-add-email-notifications`.
- If no ticket is set and the repository uses Jira, ask for it. Omit the ticket segment only when the repository has none.
- If the user names a ticket with the request, for example "branch this on IGIA-312", use it for this branch only and leave the stored default unchanged.
- Show the name and wait for approval.
- On approval, run `git switch -c <name>`.

### 3. Stage

- Whole files the user names: `git add <path>`.
- Hunk-level staging is interactive. Open it for the user with `interactive_shell({ command: "git add -p", mode: "interactive" })`, or ask them to run `git add -p` in a terminal outside Pi.
- Never stage files the user did not mention. Never stage generated output, lockfiles, or anything that looks like a secret.

### 4. Commit message

- Use the commit prompt in `references/prompts.md`. Return only the message: no notes, no preamble, no code fences. The title shape is `<gitmoji> <type>: <subject>`, for example `✨ feat: ajoute l'export CSV`.
- Rules: semantic commit prefix; present tense; explain what changed and why; focus on the most important changes; title at or under 50 characters; body wrapped at 72; no line starts with `#`.
- Language: French, keeping technical terms in English. No emoji.
- Write the proposed message to a file:

```bash
git diff --staged > /tmp/git-workflow.diff
cat > /tmp/git-workflow.msg <<'EOF'
<proposed message>
EOF
```

- Open the review in a pi overlay so the user can inspect the diff, edit the message, and save:

```typescript
interactive_shell({
  command: "git commit -v -e -F /tmp/git-workflow.msg",
  mode: "interactive",
  reason: "Review the diff and edit the commit message"
})
```

- Use `mode: "interactive"`, never dispatch. Dispatch auto-closes on quiet and can terminate the editor while the user is still reading.
- Tell the user the overlay is open, wait for them to confirm, then verify with `git log -1 --stat` and report the resulting commit.

If `interactive_shell` is unavailable, print the command for the user to run in a terminal outside Pi instead:

```text
Run this in a terminal to review the diff and edit the message:

  git commit -v -e -F /tmp/git-workflow.msg
```

### 5. Push

Only after the user confirms. Show what will leave the machine first:

```bash
git log --oneline @{u}..HEAD
git diff @{u}..HEAD
```

Then propose `git push -u origin <branch>`. Run it only when the user explicitly approves in this turn.

### 6. Pull request

- Check auth first: `gh auth status`. If it fails, ask the user to run `gh auth login` once in a terminal, then retry. Never attempt to authenticate non-interactively.
- Title: reuse the commit subject when the branch has a single commit, otherwise summarize the branch.
- Body sections: Résumé, Motivation, Tests, in the same language as the commits. See `references/templates.md`.
- Write the body to a file, show it, and get approval. To let the user edit it first:

```typescript
interactive_shell({ command: "nvim /tmp/git-workflow.pr.md", mode: "interactive", reason: "Edit the PR description" })
```

- After approval, create the PR. `gh pr create` needs no TTY, so run it directly:

```bash
gh pr create --title "<title>" --body-file /tmp/git-workflow.pr.md --draft
```

Draft PRs by default unless the user says it is ready for review.

## Conventions

- Branch: `feature/<TICKET>-<kebab-description>` or `bugfix/<TICKET>-<kebab-description>`. English, kebab-case, ticket uppercase.
- Commit: gitmoji plus semantic prefix (`✨ feat`, `🐛 fix`, `♻️ refactor`), present tense, title at or under 50 characters, body wrapped at 72, no line starting with `#`. Written in French with technical terms in English.
- PR body: Résumé, Motivation, Tests, in the same language as the commits. Reference the Jira ticket.
- One logical change per commit. Split unrelated changes instead of bundling them.

See `references/templates.md` for the gitmoji map and worked examples, and `references/prompts.md` for the commit and branch prompt templates.

## Safety

- Never push, force-push, or open a PR without explicit approval.
- Never rewrite pushed, reviewed, or shared history without asking.
- Never commit without the user reviewing. Open the editor with `interactive_shell` and let the user save. Do not pass a pre-baked `-m` to skip review unless the user asked for it.
- Always show the exact command before it runs.
- If asked to regenerate, produce a new candidate. Do not silently rewrite the previous one.
- Keep secrets, credentials, and personal data out of commit messages and PR bodies.
