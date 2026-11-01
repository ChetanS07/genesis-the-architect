# Project Folder Structure

**Feature-first beats type-first.** Group files by the business capability they serve (auth, billing, dashboard), not by file type (components, hooks, utils). Type-first layouts give every new file a global bucket to dump into. Feature-first forces "what does this belong to?" before "what kind of file is it?"

Applies to applications. Libraries and CLIs follow only the naming rules. Repo-specific choices live in optional `docs/standards/project-structure.md`; without it, the defaults here apply.

## Where a new file goes

Use the first match:

1. Wires URLs or starts a process: `app/`, `routes/` or an entry point.
2. Defines a table or row type: `entities/`.
3. Request or response shape used by two or more apps: `packages/contracts/`.
4. Infrastructure (DB client, env, logger, ports, adapters, middleware): `shared/`, from day one.
5. Supports tests: `testing/`.
6. Used by one feature: that feature's folder.
7. Domain-free helper that a second feature needs: promote to `shared/`. Never promote on first use.
8. Behaviour owned by another feature: import it through that feature's public surface.

## Layout

```
<repo>/
├── apps/{web,api}/src/      # one per deployable; migrations stay outside src/
├── packages/contracts/src/  # shapes used by 2+ apps; everything else is duplicated on purpose
├── e2e/                     # tests across deployables
├── docs/{adr/,standards/}
├── GLOSSARY.md              # source of feature names
└── eslint.boundaries.mjs    # copied from the plugin, version pinned

src/                         # inside each app
├── app/ or routes/          # composition: URLs and wiring only, kept thin
├── server.ts, app.ts        # composition: entry points
├── features/<capability>/
├── entities/                # only if this app owns a database schema
├── shared/{components,hooks,types,lib,middleware,ports,adapters}/
└── testing/                 # fixtures, fakes, test app factories
```

Single-app projects drop `apps/` and `packages/`; `src/` sits at the repo root. Existing repos with another root layout keep it and record it in `project-structure.md`.

## Import direction

```
composition  →  features  →  entities  →  shared  →  contracts
```

- A layer imports only from layers to its right.
- Features reach each other only through the public surface, with no cycles.
- `testing/` can import any layer. Production code cannot import it, except a test-built entry point.

Lint enforces this (`eslint-plugin-boundaries` via `eslint.boundaries.mjs`, or the stack's equivalent). A rule that is not linted will drift.

## Feature layers

One folder holds one kind of thing. Layers come from this closed list:

| Folder        | Holds                                           |
| ------------- | ----------------------------------------------- |
| `components/` | UI                                              |
| `hooks/`      | React hooks (only where the stack has them)     |
| `actions/`    | Server actions (only where the stack has them)  |
| `handlers/`   | HTTP handlers                                   |
| `services/`   | Orchestration and side effects                  |
| `logic/`      | Pure rules, no I/O                              |
| `data/`       | Queries against this app's own database         |
| `clients/`    | Calls to external APIs and third-party services |
| `content/`    | Static copy                                     |
| `config/`     | Feature settings                                |
| `types/`      | Types                                           |

Any other folder inside a feature is a sub-feature.

**`services/` vs `logic/`:** if the file touches I/O, directly or through what it calls (database, API, mail, clock, randomness), it is a service (or `data/` or `clients/` if that is all it does). If it only maps values in to values out, it is `logic/`. `logic/` never imports `data/`, `clients/`, `services/` or `shared/adapters/`.

Layer folders appear on demand: a feature with one layer stays flat, with two or more each gets a folder. No empty folders or `.gitkeep` ahead of the code.

```
features/job-postings/{components,data,handlers,services}/
```

**Sub-features.** Split a feature when it holds two capabilities that do not import each other, or group features that share a prefix and depend on each other. Sub-features import each other directly; everything outside the parent uses the parent's public surface.

```
features/billing/{invoices,subscriptions}/{components,data}/
```

## Public surface and barrels

Each feature has one public surface, the only part other features and composition layers may import. Lint rejects imports into anything else. Use one mode per app; the profile sets the default and an ADR can override it.

**Explicit paths** (default for `next-app` and the web app of `fullstack-monorepo`). No barrel files: they slow dev compilation, weaken tree-shaking and can leak server code into client bundles. Other features may import only from `components/`, `hooks/`, `services/` and `types/`. Anything needed from an internal layer is exposed through a function in `services/`. Server-only files use the `server-only` package in Next.js.

**Barrels** (default for `api-only` and profiles without a client bundle). `features/<name>/index.ts` is the public surface. Server-only code in a client-bundled app goes in an optional `server.ts` behind `server-only`. These are the only re-export files: none inside layers, sub-features, `shared/` or `entities/`.

## Entities

- Table definitions and row types only. No queries, rules or extra validation.
- One file per table, named for the table (`jobPostings.ts`). No grouping barrel.
- Relations are declared in the table that holds the foreign key; reverse relations live in the querying feature's `data/`.
- Queries over these tables belong to the feature that runs them.
- Migrations are generated from `entities/`.

**Leaving this model.** A shared `entities/` fits a small team on one database. If two or more people regularly conflict there, or features ship schema changes on different cadences, move to per-feature schemas in `features/<name>/data/schema/`. Features reference each other's tables by id only, and cross-feature joins move to a composition-level query. Record an ADR and apply it to the whole app.

## Naming and tests

- Folders: kebab-case. React components and classes: PascalCase (`ProfileCard.tsx`). Everything else: camelCase (`getUser.ts`). Role suffixes are fine where a role repeats (`billing.routes.ts`).
- Name files for what they do (`formatCurrency.ts`). `utils.ts`, `helpers.ts`, `common.ts`, `misc.ts` and `index2.ts` mean the purpose is not understood yet.
- Features use the glossary term on every tier. Content features with no capability behind them can be named for their page (`home`, `legal`).
- Tests are co-located (`foo.ts` beside `foo.test.ts`). Route integration tests sit beside the route; cross-deployable tests go in `e2e/`. A mirrored `__tests__/` tree is fine when tooling requires it.
- Conventions bend to match an existing codebase. Once set, they are locked for that repo.

## Profiles

| Part                                   | `next-app`          | `fullstack-monorepo`                  | `api-only`          | `content-site`    |
| -------------------------------------- | ------------------- | ------------------------------------- | ------------------- | ----------------- |
| `apps/` and `packages/`                | No                  | Yes                                   | No                  | No                |
| `app/`                                 | Yes                 | In web                                | No                  | Yes               |
| `routes/`                              | No                  | In API                                | Yes                 | No                |
| `entities/`                            | If it owns a schema | In the app that owns it               | If it owns a schema | No                |
| `shared/ports/` and `shared/adapters/` | If needed           | In API                                | If needed           | No                |
| Barrel mode                            | Explicit paths      | Explicit paths in web, barrels in API | Barrels             | Explicit paths    |
| Feature naming                         | Capabilities        | Capabilities                          | Capabilities        | Pages are allowed |

## What holds and what bends

**Always holds:** feature-first; one layer per folder; shared only after the second use; no bucket file names; one lint-enforced public surface per feature; one-way imports with no feature cycles.

**Holds when it applies:** `entities/` in any app that owns a schema (until an ADR moves it); `packages/contracts/` when two or more deployables share shapes.

**Bends, then locks per repo:** root layout; case conventions; barrel mode and schema model (per app); test placement; component file case.

## Red flags

- A second layer is joining a folder, or a DB query and an external API call share one.
- A pure function is landing in `services/`, or I/O in `logic/`.
- A bucket file name, or a feature that cannot be described as one noun phrase.
- An import reaches into another feature's internals, or two features import each other.
- A folder outside the layer list that is not a sub-feature.
- A table definition in a feature, or a query or rule in `entities/`.
- `shared/` or `entities/` importing from a feature, or production code importing `testing/`.
- An empty layer folder or `.gitkeep` "for later".

**Done when** every file sits in a feature, `entities/`, `shared/<layer>/`, `testing/`, `packages/`, a composition layer or a repo-level folder, no folder mixes layers, and the boundaries lint passes.

Example trees are in `folder-structure-scaffolds.md` beside this file. Read it when scaffolding a new project or when the repo's shape is unclear.
