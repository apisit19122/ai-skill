---
name: github-commit-push
description: >
  Use this skill whenever the user wants to commit or push code via Git.
  Covers the commit flow (git add, commitizen, commitlint) and the push rebase flow (rebase-pr).
  Trigger when the user mentions: "commit", "push", "rebase", "rebase-pr", "PR", "pull request",
  "git add", "send code", "upload code", "push branch", or says they've finished writing code.
  Important: Use this skill even if the user just says "done" or "going to commit" without explicitly mentioning git.
---

# GitHub Commit & Push Workflow Skill

## Overview

This workflow has 2 main parts:
1. **Commit Flow** — git add → commit via commitizen → confirm
2. **Push Rebase Flow** — rebase-pr → push → display PR URL

Always read config files first: `./commitlint.config.cjs` and `./cz.config.cjs` (located in the project root)

> **⚠️ Language Rule: All commit messages (short description, long description, and any generated text in the commit) MUST be written in English only — no exceptions.**

---

## Commit Flow

### Step 1 — git add .

Inform the user that all files are being staged:

```
$ git add .
```

### Step 2 — Run scripts "commit"

Inform the user:
```
$ npm run commit   (or yarn commit / pnpm commit depending on the project)
```

### Step 3 — Select type of change

1. Read `cz.config.cjs` to get all available types
2. Analyze staged changes (from `git diff --cached --stat`) and **select the most appropriate type automatically** — no need to ask the user
3. Inform the user which type was selected and briefly explain why

> Example: "Selected `feat` because this adds a new feature"

### Step 4 — Ticket number

**⚠️ Critical rule: If the user has not provided a ticket number, stop and ask before proceeding — this step must never be skipped**

Pattern: `\d{1,5}` (1–5 digit number e.g. `1234`, `567`)

Ask:
```
🎫 Please provide the Ticket Number (1–5 digits, e.g. 1234):
```

Wait for the answer before moving to the next step.

### Step 5 — Short description

1. Run `git diff --cached --stat` and `git diff --cached` to inspect staged changes
2. Read `cz.config.cjs` to find `maxHeaderWidth` or `subjectLimit` (maximum length of the short description)
3. Analyze the changes and generate a short description that:
   - Uses imperative tense (e.g. "add", "fix", "update" — not "added", "fixed")
   - Does not exceed the length defined in config
   - Summarizes the key changes

> Show the suggested description to the user

### Step 5b — Long description (if the project supports it)

Check `cz.config.cjs` for a long description field (e.g. `messages.body`, or `skipQuestions` that does not include `body`).

**If a long description field exists:**
1. Re-analyze the staged changes
2. Write a long description covering:
   - Main files changed
   - Reason or context for the change (if inferable)
   - Impact or anything else worth noting

> Show the suggested long description to the user

### Step 6 — Confirm commit

Display the full commit message that will be used, then immediately answer `y`:

```
✅ Confirming commit → y
```

Do not ask the user again — answer y directly.

---

## Push Rebase Flow

**Always runs automatically right after the Commit Flow completes — even if the user only said "commit". Do not ask for permission or confirmation to proceed.**

### Step 1 — Run rebase-pr

Run via scripts command:
```
$ rebase-pr
```

> If the command is not found in the project, try `npm run rebase-pr` or `yarn rebase-pr`  
> If not found at all, notify the user and wait for instructions

### Step 2 — Select base branch mode

**Check the current project/repo name first** (via `git remote get-url origin` or folder name)

- If the repo is **OSS-PORTAL**: ask the user whether to choose mode 1 or 2
- **All other repos**: answer `1` (auto-detect) immediately

```
Select mode (1/2): 1
```

**If mode 2 is selected** — always ask for the base branch name first:

```
🌿 Please specify the base branch name:
```

Wait for the user's answer, then pass that branch name into the script.

### Step 3 — Rebase and confirm push

After a successful rebase, the script will ask whether to push:

```
❓ Do you want to push '<branch>'? (y/n): y
```

Answer `y` immediately — no need to ask the user.

### Step 4 — Display PR URL

After the push completes, retrieve information and display the PR URL:

1. Run `git remote get-url origin` to get the repo URL
2. Convert SSH → HTTPS if needed: `git@github.com:ORG/REPO.git` → `https://github.com/ORG/REPO`
3. Get the current branch: `git rev-parse --abbrev-ref HEAD`
4. Display the URL to open a PR:

```
🔗 PR URL:
https://github.com/<ORG>/<REPO>/compare/<BASE_BRANCH>...<CURRENT_BRANCH>?expand=1
```

> Use the base branch from the rebase step (main, master, or whatever the user specified)

---

## Code Conflict

**⛔ Stop immediately** — do not proceed

Notify the user:
```
⚠️ Conflict detected! Pausing — waiting for your instructions.
Please resolve the conflict and let me know when to continue.
```

Wait for the user's command only. Do not suggest how to resolve the conflict unless asked.

---

## Reading Config Files

### cz.config.cjs — What to look for

| Field | Purpose |
|-------|---------|
| `types` | List of commit types available in step 3 |
| `subjectLimit` or `maxHeaderWidth` | Maximum length of the short description |
| `skipQuestions` | If `body` is absent from the array → project supports long description |
| `messages` | Labels shown in each prompt step |

### commitlint.config.cjs — What to look for

| Field | Purpose |
|-------|---------|
| `rules['header-max-length']` | Validates short description length |
| `rules['type-enum']` | Valid types (cross-check with cz.config) |

---

## Commit Message Format Example

```
<type>(<scope>): [<ticket>] <short description>

<long description (if applicable)>
```

Example:
```
feat(auth): [1234] add OAuth2 login support

- Added Google OAuth2 provider configuration
- Updated login page to include social login buttons
- Stored token in httpOnly cookie for security
```

---

## Notes

- **PR URL format (GitHub)**: `https://github.com/<ORG>/<REPO>/compare/<base>...<branch>?expand=1`
- **OSS-PORTAL** is a special repo — always ask the user before selecting the base branch mode