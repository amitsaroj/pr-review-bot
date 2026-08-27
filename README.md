# Automatic PR Review Bot

Automatically reviews every pull request opened against `main` using Claude,
and posts a structured review comment directly on the PR — no manual
checking required.

## What happens automatically

1. A developer opens a PR (or pushes new commits to an existing PR) targeting `main`.
2. GitHub Actions runs `pr-review.yml`, which fetches the full diff.
3. The diff is sent to Claude for a full code review.
4. Claude posts a comment on the PR with:
   - A verdict (✅ looks good / ⚠️ needs changes / 🔴 blocking issues)
   - File-by-file notes
   - A checklist of suggested fixes

## Setup (5 minutes)

### 1. Copy these files into your repo
```
your-repo/
├── .github/
│   └── workflows/
│       └── pr-review.yml
└── scripts/
    ├── package.json
    └── pr-review.js
```

### 2. Add your Anthropic API key as a repo secret
1. Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `ANTHROPIC_API_KEY`
4. Value: your key from [console.anthropic.com](https://console.anthropic.com)

`GITHUB_TOKEN` is provided automatically by GitHub Actions — no setup needed.

### 3. Commit and push
That's it. The next PR opened against `main` will trigger a review automatically.

## Customizing

Open `scripts/pr-review.js` and edit the `CONFIG` object at the top:

| Setting | What it controls |
|---|---|
| `ignoredPatterns` | File patterns to skip (lockfiles, generated code, etc.) |
| `maxPatchCharsPerFile` | Skip reviewing any single file diff larger than this |
| `maxTotalDiffChars` | Total diff size budget sent to Claude per PR |
| `model` | Which Claude model to use |
| `maxTokens` | Max length of the generated review |

To review PRs targeting a different branch (or multiple branches), edit the
`branches:` list in `.github/workflows/pr-review.yml`.

## Notes
- This posts a **regular PR comment**, not a formal GitHub "review" — so it works smoothly alongside required human reviewers and branch protection rules.
- The bot does not block merges by itself. If you want blocking behavior, add branch protection rules requiring the workflow to pass, or extend the script to fail the job on a 🔴 verdict.
- Costs: each PR review uses the Anthropic API and will incur usage costs based on diff size.
