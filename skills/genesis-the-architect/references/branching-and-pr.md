# Branching and PR guidelines

Applies to every git and GitHub operation the agent performs. `<default>` means the repo's default branch. Look it up with `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`; never assume `main`.

## Precedence

If the repository root contains any of the following, they override this file where they conflict. Read them before acting:

- `CONTRIBUTING.md`
- `.github/CODEOWNERS`
- `.github/pull_request_template.md`
- Branch protection rules (`gh api repos/:owner/:repo/branches/<default>/protection`)
- A commit template at `git config commit.template`

State the conflict in the next reply so the human sees which rule won.

A human instruction in the current session can authorize one hard-stop action (see below) for the exact action named. It does not carry over to later actions or sessions.

## Branching

Trunk-based. One default branch, always deployable. Short-lived feature branches off it. No `develop`, no release branches.

- Start from a clean working tree. If there are uncommitted changes you did not make, stop and ask.
- Branch from the default branch after `git fetch origin && git pull --rebase origin <default>`.
- Name: `<type>/<short-description>`. Lowercase, hyphen-separated, under 40 chars.
- Types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`.
- Examples: `feat/add-login`, `fix/cart-redirect`, `chore/upgrade-deps`.

## Commits

- One commit is one coherent change. Do not bundle unrelated edits.
- Stage files by name. Do not use `git add -A` or `git add .`.
- Before each commit, check the staged diff for secrets (`.env` files, keys, tokens).
- Title: imperative mood, under 72 chars. Body optional, wrapped at 72.
- Good: `Add rate limiting to login endpoint`
- Bad: `fixed stuff`, `WIP`, `updates`, `.`
- If a pre-commit hook fails, fix the cause. Never bypass it.
- Do not amend or force-push commits already on a shared branch without explicit human confirmation.

## Branch size and lifetime

- Measure with `git diff --shortstat origin/<default>...HEAD`, excluding lockfiles and generated files.
- Target: under 1000 lines of diff per PR.
- Hard stop at 1500 lines: stop and ask the human whether to split before opening the PR.
- Lifetime target: under 48 hours of active work.
- If the default branch has moved:
  - Before the first push: rebase onto `origin/<default>`.
  - After the first push: merge `origin/<default>` into the branch. Do not rebase a pushed branch.

## Workflow

1. Create the branch (see Branching).
2. Commit as you go. Make a small, coherent commit each time a piece works. No `WIP` commits, and no single giant commit at the end.
3. Push and open a draft PR early: `git push -u origin <branch>`, then `gh pr create --draft --base <default>` (head is the current branch).
4. Commit any follow-up changes on the same branch, then `git push`. Unpushed commits are not part of the PR, and CI does not see them. Run `git status -sb` and confirm the branch is not ahead of origin.
5. Run the local checks listed under Pull requests and fix any failures. Then `gh pr ready`, wait for CI to go green, and report to the human. The human merges.

## Pull requests

Before `gh pr ready`, run locally and fix any failures:

1. Lint
2. Type-check (if applicable)
3. Tests
4. Build

PR title: imperative, under 72 chars. Squash-merge makes it the commit message on the default branch, so write it for that audience.

PR description (use the repo's template if one exists; otherwise this structure):

- **What**: one-sentence summary.
- **Why**: the reason; linked issue if any.
- **How to test**: specific steps a reviewer can run.
- **Screenshots**: for any UI change.
- **Risk**: what could break; areas not touched that might still be affected.
- **Files changed**: count from `git diff --shortstat origin/<default>...HEAD`.
- **Features worked on**: a bullet list of each feature or change included in the PR.

## Merging (human action)

The agent never merges. A human does, using squash-merge only. Never merge-commit, never rebase-merge into the default branch.

The agent's job is to report whether the PR meets these conditions:

- Approvals per CODEOWNERS or branch protection; otherwise at least one non-author human approval.
- All CI checks green.
- No unresolved review comments.

After the human merges, delete the branch locally and on the remote, if the human asks or the repo does not auto-delete.

## Hard stops (ask the human, do not proceed)

Stop and ask before:

- Pushing directly to the default branch.
- Force-pushing to any shared branch, including `--force-with-lease`.
- `git reset --hard` on a branch with unpushed commits.
- `git clean -fd` or any command that discards uncommitted work you did not create.
- Deleting an unmerged branch, local or remote.
- Merging any PR, including one you opened.
- `git rebase` on a branch another contributor has committed to, or one already pushed.
- Any git command with `--force`, `-f`, `--no-verify`.
- Any `gh pr merge --admin` or equivalent that bypasses protection rules.
- Any operation that rewrites shared history.

## When things go wrong

- **Merge conflict**: stop. List the conflicting files, show the conflicting hunks, ask the human how to resolve. Do not guess.
- **CI failure**: read the output, attempt one targeted fix, re-run. If it fails again, stop and report the error output.
- **Accidental commit to the default branch locally**: do not push. Create a branch at the current commit, reset the default branch to `origin/<default>`, open a PR from the new branch.
- **Pushed a secret or credential**: stop immediately and report to the human. Do not push further, delete the branch, or attempt history rewrites. The human must rotate the credential, because rewriting history does not undo the leak.

## Not covered

- Release tagging and versioning.
- Deployment pipelines beyond "CI must pass".
- Monorepo tooling (workspaces, nx, turborepo).
- Hotfix procedures for production incidents.
- Submodules.

For any of these, ask the human rather than inventing a workflow.

## Repo-level enforcement

The rules above are advisory text. Mechanical enforcement lives in the repo configuration (branch protection, pre-commit hooks, required CI checks). That is a one-time human setup task, not something the agent executes on each git operation.
