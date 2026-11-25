# docs

This `docs/` holds every document of the repository other than the root `AGENTS.md`. `standards/` holds the rules that every task in the repository follows when its row in the index matches, such as the checks that a change must pass, whatever the framework of the code that it touches. `nestjs/` holds the rules that every app built on NestJS follows, and `nextjs/` the rules that every app built on Next.js follows. Put a document that meets [writing-docs.md](standards/writing-docs.md) but isn't such a rule directly under `docs/`, in no directory.

## When to read

| What you're doing | Document |
| --- | --- |
| Running a command from the repository root, or changing a package script, a GitHub Actions workflow, a script that a workflow runs, or anything that checks a change | [standards/quality-gates.md](standards/quality-gates.md) |
| Adding or changing an app, its image, or a package, or anything it imports or depends on, or a test that starts another app, or adding an environment variable that an app needs to start | [standards/nx-monorepo.md](standards/nx-monorepo.md) |
| Working with a branch, a PR, or a GitHub setting | [standards/github.md](standards/github.md) |
| Changing or running a deploy, or adding a setting that an app needs where it runs, such as an environment variable, a port or a secret | [standards/deploy.md](standards/deploy.md) |
| Reading or changing the rows of a database of a deployed environment | [standards/deployed-databases-sample.md](standards/deployed-databases-sample.md) |
| Starting work that will end in a PR, or working with Linear | [standards/linear.md](standards/linear.md) |
| Naming something, or using an Omni term, in code or in a document | [standards/glossary.md](standards/glossary.md) |
| Writing a code comment | [standards/code-comments.md](standards/code-comments.md) |
| Writing or changing TypeScript code | [standards/typescript.md](standards/typescript.md) |
| Any change to the repository's documents | [standards/writing-docs.md](standards/writing-docs.md) |
| Writing, changing or moving code in the `src/` of a NestJS app, or adding a table or a database to it | [nestjs/architecture.md](nestjs/architecture.md) |
| Any work on the tests of a NestJS app | [nestjs/testing.md](nestjs/testing.md) |
| Writing code that logs in a NestJS app, or adding a NestJS app | [nestjs/logging.md](nestjs/logging.md) |
| Adding, changing or reading an environment variable of a NestJS app, or adding a NestJS app | [nestjs/environment-variables.md](nestjs/environment-variables.md) |
| Writing, changing or moving code in a Next.js app | [nextjs/architecture.md](nextjs/architecture.md) |
| Any work on the tests of a Next.js app | [nextjs/testing.md](nextjs/testing.md) |
| Adding or changing an environment variable that a Next.js app reads | [nextjs/environment-variables.md](nextjs/environment-variables.md) |
| Writing or changing what a Next.js app shows or how it looks | [nextjs/design-system.md](nextjs/design-system.md) |
