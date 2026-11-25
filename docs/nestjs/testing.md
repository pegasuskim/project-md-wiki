# Testing

## Where tests go

- Import the code under test with `@/`, in `vi.mock` too. A `vi.mock` path that no longer matches the module leaves the real module in place without an error, and a relative path stops matching when either file moves.
- Put a test under `test/`, at the same path as the code it tests, named `*.spec.ts`. The test of `src/<path>/<name>.ts` is `test/<path>/<name>.spec.ts`, and the contract test of a port, described in "Ports and fakes", is `test/<path>/<name>.contract.spec.ts`, without the `.port` of the port's file name. The fake of that port is `test/fakes/<name>.fake.ts`.
- Keep everything that only tests use under `test/`: setup files, fixtures, fakes, and the end-to-end tests in `test/e2e/` that call the app as its callers do.
- Put a test that needs a database under `test/database/`, followed by the path that it would otherwise have under `test/`, so that the `test` script, which runs without a database, leaves it out. A contract test goes there when one of its drivers needs the database, and an end-to-end test goes to `test/database/e2e/` when its cases need the rows of a real database.

## Levels

- Cover a change with a test at each level of the table below that the change reaches, so that a mistake in wiring or in an adapter fails a test as surely as a mistake in a rule does.

| Level | What it checks | What it replaces |
| --- | --- | --- |
| Unit | A domain rule, a pure function, or a usecase, alone | Each port, with a stub written in the test or with the fake of the port |
| Integration | Code of the app together with what it uses, all of it real, such as an adapter with what it talks to, or a module of `core` with its wiring. Code of `bootstrap` at the boundary with the callers runs in an app that the test builds from that code and from entry points that only the test declares | Only what is outside the process, such as `fetch` |
| End-to-end | The app as its callers reach it, such as over HTTP: which entry point answers a call, the checks of the call, and the answer to an error | Only the ports that reach outside the process, with the fakes of "Ports and fakes" through `overrideProvider` |

- In an integration or end-to-end test, also replace through `overrideProvider` a provider that holds a setting or registered entries, such as a time limit or a list of registered handlers, when the test needs another value. Keep the usecases, the rules, and the wiring real.
- Test an adapter that talks through the client of an SDK rather than `fetch`, such as an AWS SDK client, with the real client pointed at a server that the test starts in its own process, such as with `node:http`, and that stands in for the service. A fake of the method through which the client sends a request would pass a command that the service refuses or that the client sends differently. Give the module that client through `overrideProvider` of the client's provider. When the service is another app of the repository, start the build of that app instead of a stand-in, so that a change to that app that breaks its callers fails their tests. Give that app a free port, made-up values for every variable of its environment and a stand-in for each service that a case makes it call. Start it in a directory without env files and without the environment of the test, so that no real value reaches it.
- Test a usecase that reaches its rows through a unit of work against a real database, with the module of the usecase wired as the app wires it, rather than with a fake of the unit of work, because what such a usecase promises depends on its transactions and its locks, which no fake has.
- Show that a call takes a lock by holding the lock first, waiting until the call blocks on it, and changing in the same transaction what the call must then see, rather than by calls at the same time, which pass without the lock whenever they happen not to overlap.
- Test code that starts a worker thread with the real worker entry of `src/`, rather than a fake of the code that starts it, so that a test covers what the worker does, how it ends, and what it posts back.

## Ports and fakes

An end-to-end test that replaces a port passes only as long as what replaces it behaves as the adapter does, so keep the two together:

- Keep one fake of each port that an end-to-end test replaces, in `test/fakes/`, and use it in every end-to-end test that replaces the port, seeded with what the case needs, rather than a stub written in the test. Spy on the fake with `vi.spyOn` to check the calls that it gets.
- Give such a port a contract test that runs the same cases against the adapter and against the fake, in one file, so that neither can be left out. The cases own their names and expected values, and a driver for each of the two owns how to set up each case.
- Make a driver that can't set up a case fail it rather than skip it, because a skipped case removes its check without an error.
- Cover in the contract only what the declaration of the port promises its callers, such as the answer, the errors that it declares, and how it ends once its signal aborts. Test what the adapter sends and receives in the adapter's own test.
- Make the fake throw the errors that the port declares, never an error of its own.
- Delete a fake once no end-to-end test uses it, with its driver in the contract test.

## Test data

- Build the input of a case that differs from a valid input in a few fields from a function that returns the valid input and takes those fields as overrides, rather than from a copy of the valid input, so that a case shows what it changes.
- Fix every value that an expectation depends on and that changes between runs, such as the current time with `vi.setSystemTime` or an id or a random value with a mock of what makes it.
- Keep a fixture in the test file that uses it, and move it to `test/fixtures/` when a second file needs it.
- Write an expected value by hand, rather than computing it with the code under test, which would move the expectation along with a bug.

## Environment

- When a test that builds a module needs the environment that its providers read, set it with one helper in `test/helpers/` of the app, such as `stubTestEnv`, so that a new required variable is added to the tests in one place.
- When a test builds, without the app module, a module whose providers take settings from `ConfigService`, give it the app's `ConfigModule` with `validateEnv` through one helper in `test/helpers/`, after setting the environment, so that the module gets the values that the app would check when it starts.
- Set the environment before a test imports a module that checks the environment when it is imported, such as `app.module.ts`, by importing that module with `await import()` after `vi.stubEnv`. To import it again with other values in the same file, call `vi.resetModules()` first and import again every injection token that the test uses, because the fresh copy defines new tokens.

## End-to-end

- Start one app per end-to-end suite over HTTP. Call `app.listen(0)` after `app.init()`. Close the app in `afterAll`. A server that isn't listening makes supertest open and close a port for each request, and suites that run together then race for ports.

## Processes of a worker

- Test what a worker promises when its process dies or is told to stop with real processes on a real database, which run the code of `src/` as it is, with the work held where the test tells it to, so that a signal arrives at a known step. Start a queue worker through `startProbeWorker`, with a workflow of `test/fixtures/` that waits where the test tells it to.

## Built app

- Test what only the process of the built app shows, such as a file that the app loads by its path at run time or a line that `main.ts` writes, against the built app in `test/build/`, which the `test:build` script runs after `build` in `ci:check`. A test of the source passes with such a file missing from the build.

## A test that can fail

- When a test passes on its first run, break once the behavior it guards and see it fail, because a test that can't fail guards nothing.
- When a test passes and fails on the same code, find and fix what makes it change, such as the clock, a random value, state that tests share, or cleanup that races the next test, rather than retrying or skipping it. Leave `retry` unset in the Vitest config. A retry hides a failure that a caller of the app meets too.
