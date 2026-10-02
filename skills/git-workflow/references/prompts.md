# Prompt templates

Exact prompt shapes for commit messages and branch names. These were tuned for this workflow, so keep the wording.

Replace `{{diff}}` with the output of `git diff --staged`, `{{commits}}` with `git log --oneline -20`, and `{{ticket}}` with the Jira ticket from `git config --local --get git-workflow.ticket` (or `none`).

## Commit message

````text
Write a commit message for my changes.
Only respond with the commit message. Do not add notes, a preamble, or code fences.
Start the title with a gitmoji that matches the change, followed by a semantic commit prefix.
Explain what changed and why.
Focus on the most important changes.
Use the present tense.
Hard wrap lines at 72 characters.
Keep the title at or under 50 characters, counting the gitmoji as one character.
Do not start any line with the hash symbol.
Write it in French. Keep technical terms in English.

Here is the git diff:

```
{{diff}}
```
````

## Branch name

````text
Generate a branch name for the given changes. A branch name is a brief description of the changes in the diff when a diff is given, or a summary of the commit messages when only commits are given.

Use the prefix "bugfix/" when the changes primarily fix bugs or correct incorrect behavior. Use the prefix "feature/" when the changes primarily add new functionality or enhancements.

If a ticket is provided, place it right after the prefix in uppercase, then a hyphen, then a short English kebab-case description. Example: "feature/IGIA-290-add-email-notifications". If the ticket is "none", omit the ticket segment.

Output only the branch name, nothing else.

Ticket: {{ticket}}

Here is the git diff:

```
{{diff}}
```

And here are the commit messages:

{{commits}}
````

## Notes

- The commit prompt asks for French with English technical terms. Branch names stay English kebab-case.
- The title shape is `<gitmoji> <type>: <subject>`, for example `✨ feat: ajoute l'export CSV`.
- The `#` rule matters: git treats lines starting with `#` as comments and would drop them.
- The ticket lives in repo-local git config so it survives across tool calls and sessions.
- Resolve `{{ticket}}` as the per-branch override when the request names one, otherwise the stored default from `git-workflow.ticket`.
