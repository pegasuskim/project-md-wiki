# Logging

These rules apply to the NestJS apps, which log through the `Logger` of `@omni/logging`.

## Setting up an app

- Make the app's one `Logger` in `src/support/logging/logger.ts`, before Nest starts.
- Give the `Logger` to `NestFactory.create` through the app's `NestLoggerAdapter`, with `abortOnError: false`, so that Nest's own lines and a failure to start go through it. Nest otherwise exits by itself on a failure to start, before `main.ts` can log it.
- Call `app.enableShutdownHooks()` in `main.ts`. Without it, Nest runs no shutdown hook on `SIGTERM`, and the shutdown goes unlogged.
- Log `applicationStarted` in `main.ts` once the app listens.
- Log a failure to start once, with `applicationStartupFailed`, in the `catch` of `main.ts`, before the process exits with 1.
- Name the service in `serviceName` after the app's image.
- Name the dataset `omni.<folder>`, with `<folder>` the name of the app's folder and `_` for `-`.
- Derive the level and the environment of the service in `bootstrap` from `APP_ENV`, the level being `info` in `production` and `debug` elsewhere, and take the version from `SERVICE_VERSION`, which `envSchema` checks with `loggingEnvSchema` when the app starts.

## Wiring of the logs into NestJS

Put the code that connects `@omni/logging` to NestJS in `src/support/logging/` of the app, rather than in a shared package, so that each app moves to another NestJS version on its own. Keep the app's exception filter in `src/bootstrap/exceptions/` instead, because it also turns an error into the answer to the caller.

- Make the app's `LoggingModule` global.
- Import `LoggingModule` once, in the app module, through `LoggingModule.forRootAsync`.
- Give `LoggingModule.forRootAsync` what the app decides about its logs, such as the paths of its health checks and the settings of the logger, so that the code in `support` imports no other layer and reads no environment variable.
- Log the shutdown from `LoggingModule`, when Nest starts to shut the app down and when it has finished.
- Run each request with its ids, through `runWithRequestIds`, from when the app receives it, so that every line of the request carries them. Take the ids of an HTTP request from `requestIdsOf`.
- Log each request once, when it ends. For an HTTP request, call `trackRequest` when its handler starts, with the paths of health checks that `LoggingModule.forRootAsync` was given in `excludePaths`. Call the `handlerEnded` that `trackRequest` returns when the handler ends.
- Keep both lines of a handler that throws after the answer went out, `http.request.completed` and `exception.handled`, because the line of the answer can't be taken back, and leaving out the exception would hide a bug that no caller can report.
- Log an exception that ends a request once, in the app's exception filter. For an HTTP request, log it through `logExceptionHandled` rather than `exceptionHandled` of the logger, because `logExceptionHandled` also marks the request, without which `trackRequest` logs the request a second time.
- Register the exception filter in the app module with `APP_FILTER` and `useClass`, so that a test that builds the app from that module gets the filter, and Nest gives a filter that extends `BaseExceptionFilter` its HTTP adapter.
- In a filter that extends `BaseExceptionFilter`, override `handleUnknownError` to answer as Nest does but without logging, because Nest's version logs the error a second time through Nest's logger.

## Where each event is logged

Write a line only for an event of this table, in the place that the table names, so that a failure isn't counted twice and no layer has to guess whether another one logged it. Add a row for an event that no row of the table covers in the change that first logs it.

| Event | Logged by | Level |
| --- | --- | --- |
| A request that a handler answered, `http.request.completed`, with its duration | `LoggingModule` | `info` |
| A request whose caller closed the connection before the answer, `http.request.aborted` | `LoggingModule` | `warn` |
| A request that an exception ended, `exception.handled` | The app's exception filter, or, for a request that a library answers itself with a 5xx, the route that hands the request to the library, with the error that the library reports, through `logExceptionHandled` before the answer is written | `error` for a 5xx, `warn` for a 4xx |
| A failure of the work of a request that its answer reports without an error status, with its cause | The code of `bootstrap` that turns the failure into the answer | `warn` when the code can tell that the input of the caller caused it, and `error` otherwise, also when a service refuses a credential of the app |
| A failure of one item of the work of a request that answered, which the work recovered from and reports in its answer | The same code, on a line for each item | As for a failure of the work |
| A failure in the work of a request that the work went on past and that the answer doesn't report, such as data that a model couldn't make, which the answer shows in another form, or a job that the request couldn't stop, with what failed in `event.action`, other than a failure to remove a file, which the row of files covers | The code of `bootstrap` that runs the usecase, such as the handler of the route, also for an answer without a body, from the failures that the usecase returned | `warn` |
| A failure to charge for work that a request finished, `credit.charge.failed`, which the answer doesn't report and which nothing charges later, with what it charges for in `labels` | The code of `bootstrap` that runs the charge after the work | `error` |
| A failure to charge for work that a job finished because what it charges by has no price, `credit.charge.unpriced`, which nothing else shows, with what it charges for and what has no price in `labels` | The workflow of the job, within the step that charges, after every statement of the step that can fail, so that neither a replay of the job nor a retry of the step writes it again | `error` |
| A failure to remove a file that the work of a request or a job no longer needs, which neither an exception nor the answer reports and nothing removes later, such as one that a request that failed had uploaded | The adapter that removes it, on a line for each file, or for each folder of files that it removes at once, without what can name a person, such as the name of the file | `error` |
| An exception that a library would answer itself without an error status, such as a bug in the handler that the SDK of a protocol calls | The code of `bootstrap` that catches it before the library does | `error` |
| Startup and a failure to start | `main.ts`, and the failure to start of a worker in the entry of the worker | `info`, `fatal` for a failure to start |
| A check at the start of the app of settings that it can run without, which the start doesn't wait for, when it finds what an operator has to fix or when it fails, once at each start | The code of `bootstrap` that runs the check | `error` |
| A worker that started, `worker.started`, and one that stopped taking jobs, `worker.draining` | The runtime of the worker, or for a periodic worker, which takes no jobs, the code of `bootstrap` that runs its work | `info` |
| Shutdown | `LoggingModule` | `info` |
| The end of a run of a job, `job.completed` or `job.failed`, with its duration, or `job.stopped`, for a job that was cancelled or went to another worker | `registerJobWorkflow` | `info`, `error` for `job.failed` |
| A failure in a step of a job that the job went on past, `job.step.recovered`, such as a summary that it goes on without or an answer that it asks for again, with the step in `labels` | The workflow of the job, within the step, so that a replay of the job doesn't write it again, from what the usecase of the step returned | `warn` |
| A job of a dead worker that went back to its queue, `job.reclaimed`, or that was given up after killing its workers, `job.reclaim.exhausted` | The reclaimer of the worker | `warn`, `error` when it was given up |
| A failure of the heartbeat of a worker, `worker.heartbeat.failed`, or of its reclaiming, `job.reclaim.failed`, once for each outage | The runtime of the worker | `error` |
| A row that work repeating at a fixed interval changed, such as `project.trial.closed`, with the ID of the row in `labels` | The code of `bootstrap` that runs the work, on a line for each row | `info` |
| A failure of work repeating at a fixed interval, such as `project.trial_expiry.failed`, once for each run that it fails in | The code of `bootstrap` that runs the work | `error` |
| Nest's own lines, `application.framework.log` | The app's `NestLoggerAdapter` | `debug`, or Nest's level when it is `warn` or above |
| The warnings and errors of DBOS, `application.framework.log`, other than those of a step or a job, which the line of the job records, and its progress, which no one acts on | The app's `DbosLogger` | The level of DBOS |

- Record a failure once, on the line of the event that it ends, such as a failure that the answer reports or `exception.handled`. The code that throws it, and the code that catches it and throws another, write no line: they pass the original error as the `cause` of the one they throw, with the call that failed, so that the line of the event keeps its cause.
- Leave an exception that no code turns into a result, such as a bug, to the exception filter. Where a library answers every error of a handler itself, such as the SDK of a protocol, catch the error in `bootstrap` before the library does, log it there, and answer without its message, because the library would send its message to the caller and log nothing.
- Write the cause of a failure on the line of the event that it ends: the service on the far end of a call that failed in `calledService`, and the code of the failure, such as a network code or `INVALID_RESPONSE`, in `errorCode`.
- Write no line of its own for a call to a service that succeeds, or that ends because the time of the work that started it ran out. A time limit of the call's own, such as a timeout of the adapter, is a failure of the call.
- Throw from a usecase the failure that ends its work, and return with its output each failure of an item that its work recovered from.
- Log a failure that the answer reports without an error status, and the failures of items, in the code of `bootstrap` that turns them into the answer, such as the interceptor of a route or the handler that the SDK of a protocol calls, because that code knows which answer the failure became.
- Leave to the library of a protocol a request that it rejects by its own rules, such as one that names a Tool that it doesn't serve or sends arguments that break the schema that it was given, and write no line of it: the caller reads the rejection in the answer, and the line of the request records only that the request was answered.
- When the caller closes the connection before the answer, abort the work through its signal, such as the one that the library of a protocol gives the handler of a call, and write no line of its failure: the code that catches failures to log them checks that the signal aborted first, because `http.request.aborted` records the request, and a line of the failure that the abort caused would count it twice.
- Log a job as a request is logged: one line when a run ends, and none for each step that goes as it should, so that the lines of a job don't grow with its steps. Leave out the message of `job.failed`, as with a request that ends in an error, and write the failure of a step that ends the job on no line of its own, because `job.failed` records it, with `calledService` and `errorCode` when the failure holds them, such as a limit of the job or a failed call of a model.
- Tell of a failure that a process tries again on its own once for each outage, as `RecurringTask` does, rather than on each try, because a database that is down would otherwise write a line for each worker every few seconds.
- Use `error` only for a failure that an operator has to look into, and `warn` for one that the caller or the input caused or that the work went on past.

## Ids of a request

- `requestId` names one HTTP request, and `traceId` the chain of calls across Argos services. Any caller can choose both, so use them only to join log lines, never as a key that tells one caller or one piece of work from another, such as for authentication or idempotency.
- Send neither id to a service that Argos doesn't run.

## What a line holds

- Name an event in `event.action` as dot-separated lowercase words that end in what happened, such as `http.request.completed` and `application.started`.
- Give an event `event.outcome` only when it can succeed or fail.
- Name the caller of a request on the line that logs how it ended, through `identifyCaller`, in the code that learns who the caller is, such as a guard: a person by their id in `user.id`, which ECS keeps for the user who performs the action, and in `labels` what ECS has no field for, such as `caller_type` and the `project_id` of an API key. Never name a caller by an email. An operator then reads from the method, the path and the status who did what on any route, without a line for each action.
- Name a key of `labels` in lowercase with `_` between its words, as ECS does, such as `upstream_status`.
- Put a value in the field of the line that defines it when one exists, and a value that no field defines in `labels` when it is a word or an ID to filter by, such as a status or the ID of a record. Leave out any other value that no field defines, such as a measured number, which `labels` would keep as text.
- Keep out of every field, `message` included, any value that can hold personal data or a secret, such as the body of a request or an answer, the input or output of a usecase, or a token, a key or a password.
- Leave out the message of an error that can quote such a value, with `includeMessage: false`.
