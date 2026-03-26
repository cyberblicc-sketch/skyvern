# Claude Code Action — Repository Recommendations

This document lists the repositories in the `cyberblicc-sketch` organization that would benefit from the Claude Code Action workflows already used in this (`skyvern`) repository, along with the exact files to add to each repo and setup instructions.

---

## What is the Claude Code Action?

This repo uses two GitHub Actions workflows powered by [Anthropic's Claude Code Action](https://github.com/anthropics/claude-code-action):

| Workflow | File | Purpose |
|---|---|---|
| **Claude Code** | `.github/workflows/claude.yml` | Allows collaborators to tag `@claude` in any issue comment or PR review to trigger AI-assisted coding, debugging, or documentation |
| **Claude Code Review** | `.github/workflows/claude-code-review.yml` | Automatically reviews PRs from first-time / external contributors and posts inline feedback |

Both workflows require a single GitHub Actions secret: `CLAUDE_CODE_OAUTH_TOKEN`.

---

## Repositories That Would Benefit

### ✅ Already configured
| Repo | Languages | Status |
|---|---|---|
| [skyvern](https://github.com/cyberblicc-sketch/skyvern) | Python, TypeScript | `claude.yml` + `claude-code-review.yml` already present |

---

### 🔴 High priority — active codebases with multiple contributors expected

#### [Gargoyle](https://github.com/cyberblicc-sketch/Gargoyle)
- **Languages:** Python, JavaScript, CSS, HTML
- **What it does:** AI-powered shadow consultant for legal, regulatory, and compliance analysis
- **Why it benefits:** Python + JS multi-file project that would benefit from automated PR review and on-demand coding assistance to keep legal logic accurate and well-tested.

#### [Rebate-wire-ok](https://github.com/cyberblicc-sketch/Rebate-wire-ok)
- **Languages:** TypeScript (Next.js / React)
- **What it does:** Rebate wire SaaS application
- **Why it benefits:** Active TypeScript/React codebase with a database schema. Claude can help review component logic, catch type errors, and assist with schema changes.

#### [Hive-shadow](https://github.com/cyberblicc-sketch/Hive-shadow)
- **Languages:** Python
- **What it does:** Elite Crypto IP + Signals + Markets SaaS
- **Why it benefits:** Python service with a Dockerfile. Claude can help review trading logic, suggest improvements to signal processing code, and auto-review any external contributions.

---

### 🟡 Medium priority — active codebases that would benefit when scaled up

#### [MeasureAR](https://github.com/cyberblicc-sketch/MeasureAR)
- **Languages:** Swift, Java/Kotlin, TypeScript (React Native)
- **What it does:** Cross-platform AR measuring app for iOS and Android
- **Why it benefits:** Multi-language mobile project (Swift + Java + RN). Claude can review platform-specific code and help bridge native modules.

---

### ⚪ Not yet applicable — empty repositories (add workflows once code is pushed)

| Repo | Description |
|---|---|
| [Telegram-babyloco-bot](https://github.com/cyberblicc-sketch/Telegram-babyloco-bot) | Telegram bot |
| [osint-platform](https://github.com/cyberblicc-sketch/osint-platform) | OSINT command center |
| [Loco-operator-ml](https://github.com/cyberblicc-sketch/Loco-operator-ml) | ML operator project |
| [Cash-printer](https://github.com/cyberblicc-sketch/Cash-printer) | Auto video clipper and poster |
| [Cash-printer-x](https://github.com/cyberblicc-sketch/Cash-printer-x) | Video clip automation |
| [Agent-claw](https://github.com/cyberblicc-sketch/Agent-claw) | Agent project (README only) |

---

## How to Add Claude Code Action to a Repo

### Step 1 — Obtain and store the OAuth token

> **Security note:** The `CLAUDE_CODE_OAUTH_TOKEN` grants access to Claude AI services on your behalf. Treat it like a password: never commit it to source control, never share it across organizations, and rotate it immediately if you suspect it has been exposed.

1. Sign in at [claude.ai](https://claude.ai) and navigate to **Settings → Claude Code → OAuth Tokens** (or visit [claude.ai/settings/claude-code](https://claude.ai/settings/claude-code) directly).
2. Click **Generate new token**, give it a descriptive name (e.g. `github-actions-<repo-name>`), and copy the token value — you will not be able to view it again.
3. Go to **Settings → Secrets and variables → Actions** in the target GitHub repo.
4. Click **New repository secret**, name it `CLAUDE_CODE_OAUTH_TOKEN`, paste the token value, and save.

### Step 2 — Create the workflow files

Create the directory `.github/workflows/` in the target repo and add the two files below.

#### `.github/workflows/claude.yml`

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  pull_request_review:
    types: [submitted]
jobs:
  claude:
    if: |
      (
        (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
        (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
        (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude'))
      ) && (
        github.event.sender.type == 'Bot' ||
        github.event.comment.author_association == 'OWNER' ||
        github.event.comment.author_association == 'MEMBER' ||
        github.event.comment.author_association == 'COLLABORATOR' ||
        github.event.review.author_association == 'OWNER' ||
        github.event.review.author_association == 'MEMBER' ||
        github.event.review.author_association == 'COLLABORATOR'
      )
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
      actions: read
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 1
      - name: Run Claude Code
        id: claude
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          additional_permissions: |
            actions: read
```

#### `.github/workflows/claude-code-review.yml`

```yaml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]
jobs:
  claude-review:
    # Only auto-review PRs from external contributors (not maintainers)
    if: |
      github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR' ||
      github.event.pull_request.author_association == 'FIRST_TIMER' ||
      github.event.pull_request.author_association == 'NONE'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 1
      - name: Run Claude Code Review
        id: claude-review
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          # plugin_marketplaces pins to the default branch of the claude-code plugin repo.
          # Check https://github.com/anthropics/claude-code/releases for the latest stable tag
          # and pin to a specific ref (e.g. @vX.Y.Z) for reproducible builds.
          plugin_marketplaces: 'https://github.com/anthropics/claude-code.git'
          plugins: 'code-review@claude-code-plugins'
          prompt: '/code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}'
```

### Step 3 — Test it

1. Open any issue or PR in the repo.
2. Leave a comment that includes `@claude` followed by a question or instruction (e.g. `@claude can you explain this function?`).
3. The workflow will trigger and Claude will respond inline.

---

## References

- [Claude Code Action docs](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [Claude CLI reference](https://code.claude.com/docs/en/cli-reference)
- [Anthropic API keys](https://console.anthropic.com/)
