# Monorepo

## Layout

- Put an app that is deployed on its own in `apps/<plane>/<name>`, a package that the apps share in `libs/<name>`, and a shared tool configuration in `libs/config/<name>`.
- Name an app after its role, ending in `-server` for a Node server and in `-web` for a web app.
- Declare each image that an app ships in `omni.images` of its `package.json`, in the form that `.github/scripts/images.ts` describes.
- Give each app a port that no other app uses, the same everywhere the app sets it, such as its code, its scripts, its `.env.example`, its `Dockerfile`, and the `checkEnv` of its image, so that every app runs on one machine at once.
- Build one image for each app. Start each process of the app from that image, with a different command for each process. Build another image only for work that uses files that the app doesn't need at run time. A deploy tracks an image by its name, so renaming an image makes the next deploy treat it as new.
- Give each package in `nx.tags` the `type:app` or `type:lib` tag and a `scope:` tag for the plane whose code it is: `scope:app-plane` for what works with the Workflows and Cases of a Workspace, in every installation of Omni, `scope:control-plane` for what ArgosIdentity runs once to manage all Tenants and their installations, such as the System Admin Console, and `scope:shared` for code that both planes use. The import rules of `@omni/eslint-config` let a plane import only its own packages and the shared ones, so each plane builds without the other's code.
- Make a package's tsconfig extend a config of `@omni/typescript-config`.
- Make every package that has JavaScript or TypeScript code an ES module, with `"type": "module"` in its `package.json`.

## Images

- In the build stage, install only the app and the packages it depends on, and put in the image only what the app needs at run time.
- Give an image that needs environment variables to start a `checkEnv` in its `omni.images` entry, with made-up values. The image check of CI, `.github/scripts/verify-image.ts`, starts the image with them, and they are public in the repository, so a real secret never goes there.
- When an image does work that starts a process or loads a library, and neither the image's start nor its HEALTHCHECK starts that process or loads that library, check that work in a `RUN` step of the Dockerfile or with a `checkScript` in the image's `omni.images` entry. The image check otherwise passes an image that lacks what only that work needs.

CI builds and checks an app's images, and a deploy picks them, when a change reaches the inputs of the app's `image` target, so keep those inputs to what the images are built from:

- Add an npm package that writes build output that the image holds, such as SWC in a NestJS app or the compiler of a workspace package that the app runs, and the npm `dependencies` of such a workspace package, such as `pino` of `@omni/logging`, to `nx.namedInputs.imagePackages` in the app's `package.json`, besides the packages of its own `dependencies`. A package that only `typecheck`, `lint`, or `test` uses stays out, because no image runs it, even when an image holds it.
- Add a workspace package that every app's build reads, such as `@omni/typescript-config`, to the `projects` of the `image` inputs in `nx.json`.
- Add a workspace package that only some apps build or run, such as `@omni/logging`, to the images of those apps only: set `nx.targets.image.inputs` in each one's `package.json` to the inputs of `nx.json` followed by `{ "input": "imageFiles", "projects": [...] }`.

## Imports

- Use `@/` as the alias for an app's source root, and set it in both `paths` of the app's tsconfig and `resolve.alias` of its Vitest config.
- Give what a package of `libs/` exports, its names and its types alike, the role that it plays for the apps rather than the technology that implements it, such as the format that a logger writes or the library that it writes with, so that a change of technology changes nothing that the apps import.

## Tests

- Run the tests of every package, apps included, with Vitest.

## Dependencies

- Put a tool that more than one package uses in the `catalog` of `pnpm-workspace.yaml`, and depend on a tool of the catalog with `"catalog:"`, so that every package uses the one version that the catalog sets.
- Add a tool to the catalog, or raise its version, only to a stable release that supports the Node major version in `.nvmrc` and the versions of the other cataloged tools.
- In `allowBuilds` of `pnpm-workspace.yaml`, set a dependency that has an install script to `true` when the script must run, such as for a native binding or a Prisma engine, and to `false` when it isn't needed.

## Tasks and caching

Keep a cached result standing in only for a run whose inputs didn't change:

- Nx makes a target of each script in a package's `package.json`. Set how a target runs, such as its inputs, outputs, and `dependsOn`, in `targetDefaults` of `nx.json`, and give a package an `nx` field in its `package.json` only for what differs in that package. A field there replaces the same field of `targetDefaults` rather than adding to it, so repeat in it the defaults that still apply, such as the `inputs`.
- Put a file that changes the result of every task in `sharedGlobals` of `nx.json`.
- Add each environment variable that changes a task's result to the task's `inputs` as `{ "env": "<name>" }`. Nx passes every variable to the task but hashes only the ones that the inputs name.
- When a test of an app uses the output of the `build` of another app whose package name is `<other>`, add `{ "projects": ["<other>"], "target": "build" }` to `dependsOn` of `ci:check` and of each script that runs the test, in `nx.targets` of the app's `package.json`, so that the output exists when the test starts. An output that an earlier run left on your machine hides a missing entry. Add `{ "input": "default", "projects": ["<other>"] }` to `inputs` of `ci:check` and of each of those scripts that Nx caches, so that after a change to `<other>` CI picks the app to check and Nx doesn't replay a cached result.
- Set `"cache": false` on a task whose result depends on something that its inputs can't name, such as a database, a network service or the current time.
- Find the changed projects with `nx affected -t <target>`, or list them with `nx show projects --affected -t <target>`, with the base branch in `--base`, and leave out `--projects`, filtering the list afterwards when you need only some projects. With `--projects`, or without `-t`, Nx takes a project as changed when any file in it changed, instead of when a change reaches the inputs of the target.
