# Architecture

## Layers

Put the code of `src/`, other than `main.ts`, in the layer that holds its kind of code. The `nest` preset of `@omni/eslint-config` checks which layer imports which.

| Layer | Holds | Doesn't hold |
| --- | --- | --- |
| `bootstrap` | The code that starts the app, the code at its boundary with its callers, which receives a call from outside the process, checks it and answers it, such as a controller, a request schema, a guard, an interceptor, a filter or an adapter that presents a usecase in the protocol of a caller, and the environment variables that the app as a whole reads, such as its port | A business rule |
| `core` | One folder per module, each a single responsibility of the app's domain: its rules, its models, its usecases, and the ports through which it uses the outside | Code that reaches beyond the thread that runs it, such as to a database, an external API, the file system, a process stream or a worker thread |
| `infrastructure` | The implementations of the ports of `core`, in a folder named after the technology or the external service that they use, with the environment variables that only they read, such as an API key, and the Nest modules that bind them, and in `infrastructure/config/` the environment variables that adapters may read as well as `bootstrap`, such as the deployment that the app runs in | A business rule |
| `support` | Code that means the same in every module, such as canonical JSON or the code that connects the app's logs to NestJS | A term of one module |

- Put the code of `bootstrap` that only one protocol of the callers uses, such as its routes, its request schemas, the shape of its answers and its adapters, in a folder named after that protocol, such as `bootstrap/http/` for routes of plain HTTP and `bootstrap/mcp/` for an MCP endpoint, so that a change of one protocol stays in its folder. Put the code that every protocol can use, such as the authentication of a caller or the exception filter, in a folder named after what it provides, such as `bootstrap/auth/` or `bootstrap/exceptions/`.
- Name each folder of `src/` after the one thing that its code provides, such as `json` or `logging`, never with a name that fits any code, such as `common` or `utils`, because such a folder collects code that belongs elsewhere.
- Move code that knows no term of a module into `support` only when a second module needs it. Until then, keep it in the module that uses it.
- Keep a type or a rule of a module's domain in that module, whichever modules import it.
- Keep a usecase free of the protocol of its callers: it takes its input, and returns its result or throws its error. Turn them in `bootstrap` into what a caller sends and reads, such as a name, a schema, a status, a code or the shape of an answer, so that a change of how a caller reads a result changes only `bootstrap`, and a change of the result itself changes `core`.
- Keep in `core` a rule that decides what a caller may do, such as which caller may run which usecase, and apply it in `bootstrap`, such as by serving each caller only what the rule lets it run, so that a call of anything else is answered as a call of something that doesn't exist and the caller learns nothing of what others may run.
- Give the code of `support` what one module decides, such as the path of its health check, from the code that uses it, rather than holding the decision in `support`.

## Modules of core

Lay out a module of `core/<module>/` as below, and create a folder only when a file goes in it.

| Path | Holds |
| --- | --- |
| `domain/<concept>.ts` | A type that the module owns, with the types only it uses |
| `domain/errors/` | The errors that the module throws, each with a message that the app wrote, which says what failed and which a caller may read, and with the data that an answer to it needs as fields. `bootstrap` gives an error its code, the shape of its answer and any text that tells a caller what to do about it |
| `domain/policies/` | One rule each, as a pure function named `is*` when it answers or `assert*` when it throws |
| `domain/services/` | Pure functions that compute from the module's types |
| `application/usecases/` | One `@Injectable` class per usecase, with one public method, `execute()` |
| `application/contracts/` | The input and output types of a usecase, one file per usecase |
| `application/ports/` | One port per file: an interface and a `Symbol` token of the same name |
| `application/constants/` | Injection tokens and fixed operational values of the module |
| `<module>.module.ts` | The Nest module that provides the usecases and imports the infrastructure modules that bind its ports |

- Reach anything beyond the thread that runs the code through a port that the module using it declares, so that a test replaces it without touching the network.
- Name a port after its role. Name its implementation after the technology or the external service that it uses, followed by the name of the port.
- Make a usecase call ports and domain code, not another usecase, so that each usecase reads as the whole of what it does.

## Data of a module

- Give a module whose usecases keep rows in a database one port for all of it, named `<Module>UnitOfWork`, such as `ProjectUnitOfWork`, which holds a store for each kind of row and `transaction()`. A usecase then reads as the whole of what it changes, and can't change two kinds of rows outside one transaction by accident. A module that only ever writes one statement has no transaction to keep, so give it a port for that write alone.
- Put a lock of the database on the object that `transaction()` gives its work, and not on the stores, so that no code can take a lock that ends with its own statement.
- Reach nothing but the database in the work of a transaction, whose stores and locks it uses and whose rules it checks, because `transaction()` runs the work again after a conflict with another transaction, such as a deadlock. Call a service, such as to send a mail, after the transaction, and do what takes long, such as hashing, before it, because a transaction holds a connection and its locks.
- Check again within a transaction what a check before it decided, such as a permission, when the transaction waits for a lock, because it can have changed in the meantime.

## Database of an app

- Let only one app write each table. Change the rows of another app's table only by calling that app. The rules that check those rows then run in one place.
- Keep the tables of an app in a database of its own. Let no database user of another app connect to that database. A schema in a shared database doesn't keep that user out. The user that an app connects as holds `pg_read_all_data` and `pg_write_all_data`. These roles reach every table in each database that the user can connect to.
- When two apps call each other, let one of them send only events and reads. An event is a fact that already happened in the sender's own data. Let the other app send only commands and reads. A command asks the receiver to change the receiver's own data.
- Name an event between apps in the past tense and a command between apps in the imperative. Put the name where the receiver takes the call, such as in the path of a route. No check fails a call in the wrong direction. The name is the only place that shows such a call.

## Workers

A worker is a process of the app that serves no HTTP. Workers come in two kinds, told apart by what starts their work: a job on a queue starts the work of a queue worker, which keeps running and takes jobs off its queues, and a fixed interval starts the work of a periodic worker.

- Reach the engine that runs jobs through the `WorkflowEngine` port of `core/execution`, so that a usecase puts a job on a queue, sends it an event or cancels it without knowing the engine. Import the engine only in `bootstrap/orchestration/` and `infrastructure/dbos/`, which the `eslint.config.mjs` of the app checks, so that another engine replaces those two folders alone. Keep a feature of the engine, such as its limit per partition, in the adapter, and a term of the app, such as the limit per Tenant, in `core`.
- Give each worker an entry, `src/main.<name>-worker.ts`, with a root module in `bootstrap/workers/`, so that a deployment runs each worker with its own command from one image.

### Queue workers

- Give work that has to scale and fail on its own a queue worker of its own, which takes the queues of that work, so that a slow job doesn't slow an answer to a request or the jobs of other work.
- Make the root module of each queue worker import `WorkerModule.forRoot` with its queues.
- Register a workflow with `registerJobWorkflow`, in `onModuleInit` of a provider of `bootstrap/orchestration/` that the root module of the worker imports, because the worker starts the engine after that. Registered this way, each run of a job logs its end once.
- Keep in a workflow only the order of its steps, each a usecase run through `runJobStep` with a fixed `name`, because a usecase holds the rules, the engine replays the workflow when a worker dies, and the engine tells steps apart on a replay by that name, which is empty for an anonymous function.
- Give each workflow the limits of its queue from `JOB_LIMITS`, so that a job gone wrong fails rather than runs and spends without end. `registerJobWorkflow` gives the workflow the budget of the job, whose time counts from the first start of the job, also on a worker that takes it over, and it checks the time when the workflow ends, as `runJobStep` does before each step. A step that is running keeps to its own time limit. Write the end of a job in the tables of its module with this in mind: a job whose last step ran out the time fails after that step. Write the failure of a job through `runJobEndStep`, because `runJobStep` would refuse to run it once the time ran out. A job that nothing ends, such as one that the reclaimer gave up or one that was cancelled, writes no end of its own, because no code of it runs any more, so end such a job in its row, with one statement that names the limit of its queue, when a caller reads it past that limit or a new request for the same subject comes. Start a job with one statement that holds the same limit, and write the result of its last step only for a job that its row says still runs, so that a job that was only slow can't change anything after a caller read it as failed.
- Make each step safe to run twice: a step that was running when its worker died runs again on the next worker, while the steps that ended stay ended. A failure of a step comes back on the next worker as a plain error with its fields, so tell it by its `name` and its fields rather than with `instanceof`.
- Keep the state of a job that a caller or a report reads in the tables of the module, and treat the tables of the engine as the engine's own, so that another engine leaves the data of the app as it is. Only the adapter reads or writes them, and `WorkflowReclaimer`, which writes them to change a job in one transaction, has to follow the SDK when its version changes.
- Don't put a step that waits with no end that the app sets, such as for a person, in a queue with a limit, because a job that waits holds its place among the running jobs. A job that checks on work that it handed to a service may wait in its queue within the time limit of the job, because its place then also limits how much work of the queue that service holds.
- When a change alters the order or the names of the steps of a workflow, keep the jobs that are running working through `DBOS.patch`, because every deploy shares one application version, so that a new worker takes over the jobs that an old one left. A job that a change breaks fails on its step, because the engine checks the name of each step that it replays.

### Periodic workers

- Run work that repeats at a fixed interval, such as closing what has expired, in a periodic worker, whose root module imports `WorkerModule.withoutQueues()`, when each run is idempotent: a run that runs again, or at the same time as another, leaves the same rows, and the next run catches up on what a stopped run left, because such work needs no record, reclaim or retry of the engine.
- In each app, run the periodic tasks that have the same interval and the same time limit in one periodic worker, so that a task that joins them needs no entry and no deploy of its own.
- Make each start of a periodic worker run each of its tasks once and exit, so that no process stays up between runs. Something outside the app starts it at its interval.
- Exit with 0 when every task succeeded and 1 when a task failed, so that whatever starts the worker can tell which run failed, also when runs overlap. Keep each task independent of the others. Run every task even when one before it fails.
- Let the run in progress end on a signal to stop, so that it records what it did.

## Models

- Reach a model through the `ModelRegistry` port of `core/llm`. Import `ai` and `@ai-sdk/*` only in `infrastructure/ai-sdk/`, which the `eslint.config.mjs` of the app checks, because a call past that folder goes past the limit of calls, the limits of its job and the record of its tokens.
- Make each `generate` one call of the model, and keep a loop over calls, such as with Tools, in its caller, so that the limits apply to each call and the caller owns when the loop ends.
- Give `generate` the budget of the job that makes the call, or `null` for a call that no job makes, such as of a request. Let `JobLimitExceededError` through code that recovers from a failed call, such as a step that goes on without a note, because it ends the job.
- Read the limits of tokens and cost of a job as what stops its next call, and not as a ceiling on what it spends: calls that are out at the same time can't see what the others will use, so a job can pass a limit by what those calls use, and it fails when the first of them records that.
- Fail a call whose model can't be found, such as one of a provider that the app doesn't know or one that a failed read of the database leaves unknown, rather than call another model, because documents would then go to another service and the record of the call would name another model than the one that judged. Only a deployment with no active `LlmConfig` falls back on the model of its environment.
- Keep the text of a prompt in a file of `src/prompts/<set>/`, one folder for each kind of job, and not in code, so that the SHA-256 of a file names its text. The `prompt_hash` of a run names the whole folder, with the files that the run had no use for, and not what a job sent once it filled them, which isn't recorded because it holds the documents of a customer.
- Fill a prompt with `renderPrompt`, rather than with `replace`, because a value holds what a customer wrote, which `replace` would read as a pattern. It fills the placeholders that its caller names and leaves any other, which a prompt shows the model as part of a template, so give it every value that the prompt takes.
- Run `RecordRunUsecase` in each step of a job that calls a model, before the call, with what the job works on as `credit_transactions` refers to it, the set of its prompts and every Tool that it may offer, so that the view `run_summaries` shows in one row for each job what made its result. In a step it runs only when the job calls a model, so a deploy that changes a prompt during a job adds the new hash to that row only when the job goes on to send that text. A job that skips it leaves its result without a record, and no check fails.
- Read `estimated_cost_usd` of that view as what the calls of a job that a model answered cost at list prices, by which its limits stop it: a call that got no answer has no row. Read `credits_charged` as what the Tenant was charged for what the job worked on, which is the cost that a customer can check. Several jobs can work on one subject, and each of their rows shows all its credits, so don't add `credits_charged` up over them.

## Routes

- Write the version of the REST API in the path of each controller, such as `@Controller('v1/projects')`, rather than in a global prefix of `main.ts`, so that a test that builds the app from its module serves the same paths, and a path that has no version, such as `/health`, needs no list of exceptions.
- Check a body or a query by giving its decorator a strict zod schema, such as `@Body({ schema })`, so that the answer names a field that the route doesn't know and the API docs show the schema. A global pipe runs this check, so a route needs no pipe of its own for it, and a value whose decorator has no schema gets no such check. Leave a schema open only where the REST contract ignores unknown fields, such as a form that a route has always read by the names it knows, because its callers would start to fail.
- Serve what scales the queue workers at `GET /metrics`, outside the REST API, in the text format of Prometheus, with `@MetricsScraperOnly()`, because a scraper can send a fixed token and no signature or session.

## External services

- Parse the answer of an external service in its adapter, with a zod schema of that answer.
