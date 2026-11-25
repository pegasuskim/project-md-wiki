# Environment variables

## Environment variable or constant

- Make a value an environment variable only when it is a secret, such as an API key or a token, or when each deployment of the app chooses it, such as the URL of a service that each deployment has its own of. Derive a value that follows from which deployment the app runs in alone, such as whether a feature runs in `production`, from `APP_ENV` instead.
- Keep a value that is neither a constant in the code that uses it, such as a time limit or the URL of a public API. A variable for such a value is a key that every deployment must set and can set wrong.

## Secrets

- Wherever ArgosIdentity runs the app, keep the original of each secret that the app reads from its environment in GCP, never in the repository.
- Make the app read a secret only from its environment, never by calling GCP, so that its tests, CI and the image check run without GCP credentials.

## Reading a variable

- Declare the variables that code needs in a zod schema in the file of that code, such as its `*.config.ts`.
- Take the value of a variable in the provider of the code that needs it from `ConfigService`, which holds what `validateEnv` returned when the app started, so that each value is checked and parsed once. Only code that runs before `ConfigModule` can serve the environment, as the two files of "The deployment" do, reads `process.env` instead.
- Check the shape of a value in its schema, such as a range or a format, so that a wrong value stops the app when it starts rather than failing the first call that uses it.
- Import that schema into `envSchema` of `bootstrap/config/` and spread it there, rather than declaring the keys again, so that the check of every variable when the app starts uses the same schema.

## The deployment

- Name the deployment that the app runs in with `APP_ENV` alone, one of `local`, `development` and `production`, and derive from it every other value that names the deployment, such as `NODE_ENV`, the env files of local development and the environment of a log line, rather than adding a variable for it. A second variable that names the deployment can disagree with `APP_ENV` without an error.
- Derive `NODE_ENV` from `APP_ENV`, as `ConfigModule` resolves it from the environment and the env files, in `bootstrap/config/node-env.bootstrap.ts`, which `main.ts` imports before any other module, rather than setting it in the image, because some libraries read it when they load.
- Pick the env files of local development in `bootstrap/config/env-files.ts`, by the `APP_ENV` of the environment: `.env.<APP_ENV>` and `.env`, or `.env.local` and `.env` when `APP_ENV` is unset.

## Required values

- Give each app a test of its `validateEnv` that fails for each key of `envSchema` left out or set empty, so that no key has a default or is optional. A default or an optional key lets a deployment that lacks the value start, and then run with a value that no one chose or fail at the first call that needs it.
- Give each app a test that the keys of its `.env.example`, of the environment that its tests set and of the `checkEnv` of its image are the keys of `envSchema`, so that a new variable reaches each of them.
- Give a variable in `.env.example` the value of local development when it isn't a secret and every developer uses the same value, such as the port of the app, and leave every other value blank, so that a copy of the file needs only the values that each developer has to supply.
