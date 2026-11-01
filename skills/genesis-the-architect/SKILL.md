---
name: genesis-the-architect
description: Install the genesis-the-architect dev standards into this repo's docs/standards/ and register them in CLAUDE.md.
disable-model-invocation: true
---

Install the plugin's standards into the current repo, then register them in `CLAUDE.md` so future sessions read the right file at the right moment. Run once per repo; run again only to pull newer standards.

Reference files sit in `references/` beside this file:

| File | Holds | Pointed at from CLAUDE.md |
| --- | --- | --- |
| `branching-and-pr.md` | branches, commits, PRs, merging, hard stops | yes |
| `folder-structure.md` | feature-first placement rules | yes |
| `folder-structure-scaffolds.md` | example trees per stack | no, `folder-structure.md` points to it |
| `coding-standards.md` | approach, core rules, TypeScript, agent rules | yes |
| `repo-setup.md` | one-time GitHub and hook setup, for humans | no, a human checklist |

`design-system.md`, `system-design.md` and `tech-stack-and-low-level-design.md` have no source file. Step 3 writes the first from what the user supplies; step 4 brings in the other two from the user's repo.

## 1. Confirm the repo

The working directory holds a `.git` directory or a clear project root (`package.json`, `pyproject.toml`, `go.mod`). If neither is visible, ask the user for the repo path and stop until it is confirmed.

Done when the repo root is known.

## 2. Copy the references

Create `<repo>/docs/standards/` if missing. Then, for each of the five files above, compare `references/<file>` with `<repo>/docs/standards/<file>`:

- Missing in the repo: copy it. Report "created".
- Byte-identical: leave it. Report "unchanged".
- Different: collect it for the batch question below.

Ask once for all differing files together: "`<names>` differ from the plugin's current versions. For each: overwrite, keep the repo version, or show diff?" Apply each answer per file. A diverged file may be a deliberate customization, so the repo version stays until the user says otherwise.

Done when every one of the five files is created, unchanged, overwritten or kept, with the outcome noted for the report.

## 3. Ask about the design system

Ask once: "Do you have a design system to register? Share URLs (Figma, hosted docs, Claude Design artifact links) or saved Claude Design names. Reply `skip` if none."

- `skip`, or an empty reply: write no `design-system.md`. Step 5 omits its bullet.
- URLs or names: write `<repo>/docs/standards/design-system.md` from the template below, with the user's entries verbatim under Sources. Leave the URLs unfetched and the tokens unwritten; the sources hold them.

```markdown
# Design System References

User-supplied design system sources for this repo. Treat each entry as the source of truth for visual language, components, tokens, and interaction patterns.

## Sources

- <entry>

## How to use

Before implementing or modifying any UI, read the sources above and align colors, typography, spacing, and components to what they define. If a source is a Claude Design artifact, open it and extract tokens as needed.
```

Done when the user has answered and `design-system.md` exists exactly when they supplied entries.

## 4. Bring in the design files

Two files come from the user's own architecture work: `system-design.md` and `tech-stack-and-low-level-design.md`. Treat each file separately, with the same procedure. For each, look in this order and take the first match:

1. `<repo>/<file>` at the project root: move it to `<repo>/docs/standards/<file>` (`git mv` when the file is tracked). If a file already sits at the destination, ask whether to replace it or keep it, and leave both in place until the user answers.
2. `<repo>/docs/standards/<file>` already there, as after an earlier run: leave it.
3. Neither exists: the file is missing.

Ask once for all missing files together: "Do you have your `<names>` with you?" Take the answer per file:

- Yes: take the file the user attaches or points to, place it at `<repo>/docs/standards/<file>`, and leave the original where it was if it lives outside the repo.
- No, or no reply: skip that file. Step 5 omits its bullet.

File contents stay exactly as the user wrote them.

Done when each of the two files is in `docs/standards/` exactly when it was found or supplied, and neither is left at the repo root.

## 5. Register in CLAUDE.md

Write the block below into `<repo>/CLAUDE.md`, creating the file if missing. Drop the design-system bullet when step 3 was skipped, and each design-file bullet whose file step 4 skipped.

```markdown
## Standards

This repo uses the `genesis-the-architect` plugin. Reference files live under `docs/standards/`. Read the matching file **before** doing the work it covers.

- **Git branching & PRs** → `docs/standards/branching-and-pr.md`
  Read before: creating a branch, committing, opening or updating a PR.
- **Folder structure** → `docs/standards/folder-structure.md`
  Read before: creating, moving or renaming a file or folder, scaffolding a module.
- **Coding standards** → `docs/standards/coding-standards.md`
  Read before: writing or editing code.
- **System design** → `docs/standards/system-design.md`
  Read before: adding a component or service, changing how components interact, or planning work that spans several of them.
- **Tech stack & low-level design** → `docs/standards/tech-stack-and-low-level-design.md`
  Read before: choosing or swapping a library, framework or vendor, or designing a component's internals.
- **Design system** → `docs/standards/design-system.md`
  Read before: building or changing UI.

If a reference file does not exist, skip its rule rather than inventing content.
```

Placement:

- No `## Standards` heading yet: append the block at the end.
- A `## Standards` section exists, including one from the earlier `cs-dev-standards` plugin: replace it, from its heading to the next `##` heading or end of file.

Every other line of `CLAUDE.md` stays where it is. Paths stay repo-relative, because the plugin's install location differs across machines.

Done when `CLAUDE.md` holds exactly one `## Standards` section matching the block and a diff shows no change outside it.

## 6. Report

Tell the user in a few lines:

- Which files were created, unchanged, overwritten or kept.
- Whether the design-system pointer went in, and for each design file whether it was moved from the root, supplied by the user, kept in place, or skipped, with its pointer.
- That the files in `docs/standards/` are theirs to edit, and a re-run asks before replacing any edited file.
- That `repo-setup.md` is a human checklist for branch protection, hooks and CI, which the agent does not run.

Re-run only when the user asks ("re-run genesis-the-architect", "pull latest standards").
