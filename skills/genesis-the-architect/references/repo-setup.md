# Repository setup (one-time, for humans)

`branching-and-pr.md` tells the agent how to behave, but it is only text. This file lists the settings that make GitHub enforce the same rules. Do these once per repo. The agent does not do them.

`<default>` means the repo's default branch.

## Order of setup

1. Add the CI workflow and let it run once on a PR. GitHub only offers a status check as "required" after it has run recently, and the name you pick must match the job name exactly.
2. Add the repository files and hooks.
3. Turn on branch protection last. If you do it first, you will lock yourself out of merging the CI workflow itself.

## GitHub branch protection on the default branch

Set via Settings > Branches > Add rule, or via `gh api` (examples below). GitHub also offers rulesets under Settings > Rules, which cover the same ground. Use whichever your org prefers, but not both for the same branch.

- Require a pull request before merging.
- Require 1 approval from a non-author.
- Dismiss stale approvals on new commits.
- Require status checks to pass: lint, type-check, tests, build, secret scan.
- Require branches to be up to date before merging.
- Require conversation resolution before merging.
- Do not allow bypassing the above settings (enforce for admins).
- Restrict who can push to matching branches (empty list means nobody pushes directly). Organization repos only; see the notes below.
- Disallow force pushes.
- Disallow branch deletions.

Merge settings, under Settings > General > Pull Requests:

- Allow squash merging only. Disable merge commits and rebase merging.
- Automatically delete head branches after merge.

### Notes on these settings

- **Solo maintainer.** GitHub does not count an author's approval of their own PR. If you are the only human on the repo, "1 approval from a non-author" plus "enforce for admins" means nobody can merge. Pick one: set required approvals to 0 and keep the PR requirement, or add a second reviewer account.
- **Stale approvals.** The agent merges the default branch into its PR branch when the default has moved. That adds a commit, which dismisses the approval, so the reviewer has to approve again. This is the intended behavior of the setting, but expect the friction on busy repos.
- **Push restrictions.** "Restrict who can push" is only available on repos owned by an organization. On a personal repo, "require a pull request" already blocks direct pushes.
- **Check names.** The names in the `gh api` example below (`lint`, `type-check`, `test`, `build`, `secret-scan`) are placeholders. Replace them with your CI job names.

### Setting it with gh api

Run from a checkout of the repo. `gh api` fills in `:owner` and `:repo` from the current directory. Try it on a scratch repo first.

Merge settings:

```bash
gh api --method PATCH repos/:owner/:repo \
  -F allow_squash_merge=true \
  -F allow_merge_commit=false \
  -F allow_rebase_merge=false \
  -F delete_branch_on_merge=true
```

Branch protection (classic API). For an organization repo, replace `"restrictions": null` with `{"users": [], "teams": [], "apps": []}` to block direct pushes entirely:

```bash
gh api --method PUT repos/:owner/:repo/branches/<default>/protection --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["lint", "type-check", "test", "build", "secret-scan"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "restrictions": null,
  "required_conversation_resolution": true,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF
```

## Repository files

- `.github/pull_request_template.md`: matches the PR description structure in `branching-and-pr.md`.
- `.github/CODEOWNERS`: assigns required reviewers by path.
- `CONTRIBUTING.md`: a pointer only. Use one line, for example: `See branching-and-pr.md for branching, commits, and PR rules.` Adjust the path to wherever you keep that file. Do not restate the rules here. `branching-and-pr.md` says CONTRIBUTING.md wins on conflicts, so a second copy of the rules will drift and quietly override the agent's file.
- `.gitignore`: excludes secrets, env files, build artifacts.

## Local pre-commit hooks

Install via Husky, pre-commit, or `.git/hooks/`. The `.git/hooks/` folder is not version-controlled, so prefer Husky or pre-commit and document the one setup command in the README. At minimum:

- `pre-commit`: run lint and type-check on staged files.
- `commit-msg`: reject titles over 72 chars or starting with `wip`, `.`, or empty.
- `pre-push`: block pushes to the default branch; run tests.

Hooks run on each person's machine and can be skipped with `--no-verify`. They catch mistakes early. CI is the real enforcement.

## CI pipeline (required status checks)

Every PR must run and pass:

- Lint
- Type-check
- Unit tests
- Build
- Secret scan (e.g. `gitleaks`, `trufflehog`)

Mark these as required in branch protection so GitHub blocks the merge button until they are green.

Optional additions:

- A check that the PR title is under 72 chars. With squash-merge, the PR title becomes the commit message on the default branch, and the `commit-msg` hook never sees it.
- GitHub secret scanning with push protection (Settings > Code security). Availability on private repos depends on your plan.
- Configure CI to run on draft PRs. The agent opens a draft early and pushes follow-up commits, and if CI skips drafts, the first run only happens at `gh pr ready`.
