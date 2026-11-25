# Quality gates

## Commands

Run the scripts of the root `package.json` from the repository root, and run pnpm through Corepack (`corepack enable` once) so that the version in `packageManager` runs.

## Package scripts

Give every app the scripts below under these names, because Nx skips a project that has no target of the name without an error.

| Script | What it does |
| --- | --- |
| `build` | Builds the app. Only in an app whose code needs a build to run |
| `start` | Runs the built app. Only in an app that has a `build` script |
| `start:dev` | Starts the app for local development, and restarts it on changes. Only in an app that local development runs without its image |
| `typecheck` | Runs the type checker without emitting files |
| `lint` | Runs ESLint, and Steiger in a Next.js app |
| `lint:fix` | Runs ESLint and fixes what it can |
| `test` | Runs the tests that need no database or network service. Only in an app that has such tests |
| `test:watch` | Runs the tests again on every change. Only in an app that has a `test` script |
| `test:build` | Runs the tests of the built app. Only in an app that has such tests |
| `ci:check` | Runs every check of the app that needs no database or network service, such as `typecheck`, `lint`, `test`, `build`, and then `test:build` |
| `generate` | Generates code that the other scripts need, such as `prisma generate`. Only in an app that generates code |

- Put every check of a package that needs no database or network service in its `ci:check`, in the order that the package needs, because CI runs `ci:check` for each changed package. A check in `dependsOn` of the `ci:check` target is in `ci:check` too, and runs before every step of the script.
- Give a script that does the job of one of these its name. An existing name for that job can stay only as an alias.
- Run the tests that need a database or a network service under a script of their own, in a CI job that provides what they need.
- Give a shared package that has TypeScript code a `typecheck` script, and run it in the package's `ci:check`, because CI checks a shared package only through its `ci:check`.
- Leave Prettier out of a package's `lint`, because the root `pnpm format:check` checks the formatting of the whole repository.

## Lint and formatting

Give each package that has code an `eslint.config.mjs` that extends a preset of `@omni/eslint-config`, and put a rule where it applies.

| Where | Applies to | What goes there |
| --- | --- | --- |
| `base.ts` | Every package that has code | Rules that every package follows, whatever its framework |
| `node.ts` | Every package that isn't a Next.js app | Rules for code that runs on Node without Next.js |
| `nest.ts` | Every NestJS app, extending `node.ts` | Which layer of `src/` may import which, that it imports statically, and what the code of `src/` may use, such as `process` |
| `next.ts` | Every Next.js app | React, accessibility, and Next.js rules, where an app keeps its code and how it lays out its slices, which files `app/` may hold, what those files and the root files of Next.js import and declare, where code may reach a backend or read a server environment variable, the modules of `src/shared/api/`, whether code may cache what a backend answers, and the names of files and folders |
| `eslint.config.mjs` of an app | That app | Rules that only that app follows, such as an import that only it bans |

- Use only a plugin that supports the ESLint version in the catalog. When no such plugin has the check you need, write it as a rule of our own in `libs/config/eslint-config/rules/`, typed with `TSESLint.RuleModule` of `@typescript-eslint/utils`, so that `typecheck` catches a wrong rule shape or an ESLint API that no longer exists before an app's `lint` runs.
- Give each Next.js app a `steiger.config.mjs` that re-exports `@omni/steiger-config`, and run Steiger on its `src/` in its `lint` script.
- Keep an app's ESLint config a `.mjs` file, rather than installing `jiti` so that ESLint can load an `eslint.config.ts`.
- Point your editor's ESLint at the Node major version in `.nvmrc`, such as with `eslint.runtime` of the VS Code ESLint extension, and restart its ESLint server. An editor can run ESLint on a Node of its own, and an older Node, such as Node 20, fails on the `.ts` presets with `ERR_UNKNOWN_FILE_EXTENSION`.
- Keep one Prettier config, the root `.prettierrc`, and put a setting for one app in its `overrides`.

## Code that Node runs without a build

Node runs some TypeScript code as it is, without a build, such as `@omni/eslint-config` and the scripts in `.github/scripts/`, and fails on what it can't run only when the code runs. Follow these rules in such code:

- Use no decorators, which a tsconfig doesn't reject.
- Import another file of the same code with its `.ts` extension, because Node doesn't map `.js` to `.ts`.

## Git hooks

- Give each app a `lint-staged` field in its `package.json` that runs its `eslint --fix` before Prettier, and run Prettier there with `--ignore-path ../../../.prettierignore`, the root ignore file seen from `apps/<plane>/<name>`. Prettier run from an app reads no ignore file of the root, so it formats the files that the root leaves alone, such as Markdown files.

## CI

- Give a setup that only one app needs, such as a database for its tests, to that app only, so that the other apps don't wait for it.
- Pin each action to a commit SHA, with its version in a comment.
- Add a check that belongs to no project, such as one of the whole repository or of the CI scripts, to the `ci:repository` script of the root `package.json`, not as a step of its own in `ci.yml`. The `repository` job of CI and the root `ci:check` both run that script, so that a local `ci:check` runs the same checks as CI.
- Put a check of the documents under `docs/` or of the root `AGENTS.md` in `.github/scripts/test/docs.spec.ts`. `ci:repository` runs that test with the other tests. A change to only those files runs only that test, through the `ci:docs` script, so a check of them elsewhere doesn't run on such a change.

## CI scripts

- Put a script that CI runs in `.github/scripts/`, written in TypeScript, and add its dependencies to the root `package.json`, because these scripts aren't a package.
- Put a test of a script under `.github/scripts/test/`, at the same path as the script, named `*.spec.ts`, and write it for Node's test runner (`node:test`), so that the scripts need no other dependency. Put a test of a repository setting there too: in the test of the script that relies on the setting, or, when no script does, in a file named after what it checks.
