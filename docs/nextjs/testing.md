# Testing

## Where tests go

- Import the code under test with `@/`, in `vi.mock` too. A `vi.mock` path that no longer matches the module leaves the real module in place without an error, and a relative path stops matching when either file moves.
- Put a test in a `__tests__/` folder next to the code it tests, named `*.test.ts` or `*.test.tsx`. The test of `<folder>/<name>.tsx` is `<folder>/__tests__/<name>.test.tsx`. A folder whose name starts with `_` isn't a route, so `__tests__/` can sit inside `app/`.
- Name a browser end-to-end test `*.spec.ts`, so that Vitest doesn't run it.

## Test data

- Build the input of a case that differs from a valid input in a few fields from a function that returns the valid input and takes those fields as overrides, rather than from a copy of the valid input, so that a case shows what it changes.
- Fix every value that an expectation depends on and that changes between runs, such as the current time with `vi.setSystemTime` or an id or a random value with a mock of what makes it.
- Keep a fixture in the test file that uses it, and move it to a file of its own in the `__tests__/` folder when a second test there needs it.

## A test that can fail

- When a test passes and fails on the same code, find and fix what makes it change, such as the clock, a random value, state that tests share, or cleanup that races the next test, rather than retrying or skipping it. Leave `retry` unset in the Vitest config. A retry hides a failure that a user of the app meets too.
