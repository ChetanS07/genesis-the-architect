# Folder Structure: Reference Scaffolds

Reference trees applying the rules in `folder-structure.md`, one per profile. Not prescriptive: the shape bends to the stack, the rules do not. Read only the section matching the repo's profile.

Every tree shows a project at full size. Start smaller: folders appear when the code needs them, and no empty folders or `.gitkeep` files are created ahead of it.

Libraries and CLIs have no scaffold. They follow only the naming rules.

## `next-app`

One Next.js deployable that owns its database. Explicit-paths mode: no barrel files.

```
<repo>/
├── src/
│   ├── app/                          # composition: routing only, kept thin
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx        # imports features/auth/components
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/
│   │   │   └── billing/page.tsx      # imports features/billing/*/components
│   │   ├── api/webhooks/stripe/route.ts  # calls features/billing/invoices/services
│   │   └── layout.tsx
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   ├── components/           # LoginForm.tsx, RegisterForm.tsx
│   │   │   ├── actions/              # loginAction.ts, logoutAction.ts
│   │   │   ├── data/                 # getSession.ts, createUser.ts
│   │   │   └── types/                # authTypes.ts
│   │   │
│   │   └── billing/
│   │       ├── invoices/             # sub-feature
│   │       │   ├── components/
│   │       │   ├── actions/
│   │       │   ├── data/
│   │       │   └── services/         # recordPayment.ts (server-only)
│   │       └── subscriptions/        # sub-feature
│   │           ├── components/
│   │           ├── data/
│   │           └── clients/          # stripeSubscriptions.ts
│   │
│   ├── entities/                     # users.ts, invoices.ts (one file per table)
│   │
│   ├── shared/
│   │   ├── components/               # Button.tsx, Input.tsx: used by 2+ features
│   │   ├── hooks/                    # useDebounce.ts, useMediaQuery.ts
│   │   ├── lib/                      # db.ts, env.ts, logger.ts
│   │   └── types/
│   │
│   └── testing/                      # fixtures and fakes
│
├── e2e/
└── docs/standards/
```

Key moves: `app/` imports only a feature's public layers (`components/`, `hooks/`, `services/`, `types/`), and business logic never lives in a `page.tsx` or `route.ts`. `actions/` and `data/` stay internal; anything another feature needs is exposed through `services/`. Queries stay in `features/<name>/data/`, while the tables they use sit in `entities/`. External API calls go in `clients/`, not `data/`. Server-only files in a public layer use the `server-only` package.

## `fullstack-monorepo`

A Next.js frontend (explicit paths) and an Express API (barrels) that share contracts.

```
<repo>/
├── apps/
│   ├── web/
│   │   └── src/
│   │       ├── app/                  # composition: pages call the API
│   │       ├── features/
│   │       │   ├── auth/
│   │       │   │   ├── components/
│   │       │   │   ├── hooks/
│   │       │   │   └── clients/      # authApi.ts: calls to the API app
│   │       │   └── billing/
│   │       ├── shared/
│   │       └── testing/
│   │
│   └── api/
│       ├── drizzle/                  # generated migrations stay outside src/
│       └── src/
│           ├── app.ts                # composition: builds the app
│           ├── server.ts             # composition: starts the process
│           ├── routes/               # composition: maps URLs to feature handlers
│           │   ├── auth.routes.ts
│           │   ├── auth.routes.test.ts
│           │   └── billing.routes.ts
│           ├── features/
│           │   ├── auth/
│           │   │   ├── handlers/     # loginHandler.ts, registerHandler.ts
│           │   │   ├── services/     # loginUser.ts, resetPassword.ts
│           │   │   ├── logic/        # canResetPassword.ts
│           │   │   ├── data/         # findUserByEmail.ts
│           │   │   └── index.ts      # public surface: exports the handlers
│           │   └── billing/
│           ├── entities/             # users.ts, invoices.ts
│           ├── shared/
│           │   ├── adapters/         # resendMailSender.ts
│           │   ├── lib/              # db.ts, env.ts, logger.ts
│           │   ├── middleware/       # errorHandler.ts, rateLimit.ts
│           │   └── ports/            # mailSender.ts
│           └── testing/              # createTestApp.ts, fakeMailSender.ts
│
├── packages/
│   └── contracts/                    # shapes shared between web and api
│       ├── package.json              # "exports" maps ./auth and ./billing
│       └── src/
│           ├── auth.ts
│           └── billing.ts
│
├── e2e/                              # tests across deployables
├── docs/{adr,standards}/
├── GLOSSARY.md
└── eslint.boundaries.mjs
```

Key moves: both apps use the same `features/` and `shared/` layout, with barrel mode set per app. Only the API has `entities/`, because only the API owns the schema; the web app's calls to the API live in `clients/`. Each capability keeps its glossary name in both apps. `packages/contracts/` is the only cross-app shared code and is imported by file through `exports`, not through a barrel. Everything else is duplicated per app on purpose.

## `api-only`

One Express API that owns its database, in barrel mode. Add `packages/` only when a second deployable needs the contracts.

```
<repo>/
├── drizzle/                          # generated migrations stay outside src/
├── src/
│   ├── app.ts                        # composition: builds the app
│   ├── server.ts                     # composition: starts the process
│   ├── routes/                       # composition: maps URLs to feature handlers
│   │   ├── billing.routes.ts
│   │   └── billing.routes.test.ts
│   ├── features/
│   │   └── billing/
│   │       ├── handlers/
│   │       ├── services/
│   │       ├── logic/
│   │       ├── data/
│   │       └── index.ts              # public surface
│   ├── entities/                     # invoices.ts
│   ├── shared/
│   │   ├── adapters/                 # stripePaymentGateway.ts
│   │   ├── lib/                      # db.ts, env.ts, logger.ts
│   │   ├── middleware/
│   │   └── ports/                    # paymentGateway.ts
│   └── testing/                      # createTestApp.ts, fakePaymentGateway.ts
└── docs/standards/
```

Key moves: routes stay thin and call feature handlers through `index.ts`. `shared/ports/` and `shared/adapters/` appear only when the app has an external service worth swapping in tests.

## `content-site`

A marketing or content site with no database of its own. Explicit-paths mode.

```
<repo>/
├── src/
│   ├── app/                          # composition: routing only, kept thin
│   │   ├── about/page.tsx
│   │   ├── legal/page.tsx
│   │   └── page.tsx
│   ├── features/
│   │   ├── home/                     # named for its page
│   │   │   ├── components/
│   │   │   └── content/              # faqs.ts, processSteps.ts
│   │   ├── legal/                    # named for its page
│   │   │   ├── components/
│   │   │   └── content/
│   │   └── contact-requests/         # a capability, named for what it does
│   │       ├── components/
│   │       └── clients/              # submits to an external form service
│   └── shared/
│       └── components/
└── docs/standards/
```

Key moves: pages with no capability behind them can name their own feature. Anything with behaviour, such as a form that submits, is named for the capability even when it appears on only one page. With no database of its own, outside calls go in `clients/`, never `data/`.

## Other stacks

Apply the same shape, mapping the stack's layers to the closed list in `folder-structure.md`:

- **Vite SPA**: explicit paths, no routing folder. The router config is the composition layer and imports only public layers.
- **Remix / React Router 7**: `routes/` replaces `app/`, explicit paths, same thin-routing principle.
- **Python API (FastAPI, Django)**: `src/features/<name>/` with `handlers`, `services`, `data` (queries) and `types` (schemas), and database models in `src/entities/`. A small feature can use one module per layer. The public surface is the feature package's `__init__.py`. Enforce imports with `import-linter`.
- **Go service**: `internal/features/<name>/` with `handler.go`, `service.go` and `data.go`, one file per layer while small, folders when it grows. Database models go in `internal/entities/`. The public surface is the feature package's exported identifiers. Enforce imports with `depguard`.

In every case: name the features, give each its own folder, keep one layer per folder, put the data model below features, and promote feature code to `shared/` only on the second use.
