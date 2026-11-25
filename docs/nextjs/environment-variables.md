# Environment variables

- Read a value that differs between environments, such as the address of a backend, when a request runs, and never in `next.config.ts`. `next build` evaluates that file once and writes its `rewrites()`, `redirects()` and `headers()` into the build output, so the image keeps the value of the environment that built it, whatever its container sets later.
- Make each package script that starts the app take its port from `PORT`, with the app's own port as the fallback. Set `PORT` in the environment, not in an `.env` file, because the script reads it before Next.js loads `.env`.
- Wherever ArgosIdentity runs the app, keep the original of each secret that the app reads from its environment in GCP, never in the repository.
- Make the app read a secret only from its environment, never by calling GCP, so that its tests, CI and the image check run without GCP credentials.
