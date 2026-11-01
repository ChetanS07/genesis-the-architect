# Coding Standards

Applies to new and changed code. Don't rewrite unrelated code to comply.
If a rule needs breaking, say why in a comment.

## Approach

### 1. Think before coding

Don't assume. Don't hide confusion. Surface tradeoffs.

- State assumptions explicitly. If uncertain, ask rather than guess.
- Present multiple interpretations when ambiguity exists — don't pick one
  silently.
- Push back when a simpler approach exists.
- Stop when confused. Name what's unclear and ask.

### 2. Simplicity first

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it.

Test: would a senior engineer call this overcomplicated? If yes, simplify.

### 3. Surgical changes

Touch only what you must. Clean up only your own mess.

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- Notice unrelated dead code? Mention it, don't delete it.
- Remove imports, variables, and functions that your changes made unused.
  Leave pre-existing dead code alone unless asked.

Test: every changed line should trace directly to the user's request.

## Core (any language)

1. One module, one responsibility.
2. Name things for what they do. File naming: see folder-structure.md.
3. Keep functions short. Past about 50 lines, extract unless it reads
   worse. Tests, JSX and data tables are exempt.
4. Comments explain why, not what.
5. Validate at trust boundaries: API input, env vars, external files,
   cross-process messages. Unvalidated data never goes past the boundary.
6. Errors are typed and carry a stable code. Never throw strings.
7. Runtime output goes through a leveled logger. CLI stdout and throwaway
   scripts are exempt.

## Testing

- New logic gets a test. A bug fix gets a regression test.
- Test behaviour through the public interface.
- Test files are exempt from the error and function-length rules.

## TypeScript

- Strict mode. No `any` without
  `// eslint-disable-next-line @typescript-eslint/no-explicit-any -- <reason>`.
- Validate boundaries with Zod.
- Errors extend `Error` with a `.code: string` field.
- Named exports only, except where a framework or tool requires a default.
- Every promise is awaited, returned, or explicitly ignored with `void`.
- Prettier and ESLint configs live at the repo root.
- Import order: builtins, external, internal.

## Enforcement

Lint and typecheck must pass before commit. If tooling can check a rule,
tooling owns it; don't re-check it by hand.

## Agents

1. Never edit a file marked `@human-edited: do not regenerate`; stop and ask.
2. Put the task ID in a commit trailer: `Task: <id>`.
