# Deployed databases

## Keep out of the rows of prod

Never connect to the databases of `argos-omni-prod` yourself, even as a user that an app connects as, so that only the apps and their migrations read and change its rows.

## Read the rows

Read the rows of a database in an environment whose Cloud SQL instance allows the Data API with that API, which runs your SQL on the server as your own Google identity, rather than as a user that an app connects as. The users of an app may write rows, so a query run as one of them can change data by mistake. Only `argos-omni-dev` allows the Data API.

```sh
gcloud sql instances execute-sql omni --project=argos-omni-dev --database=<database> --row-limit=50 \
  --sql="SELECT ..."
```

- Pass to `--database` the database whose rows you read, from `gcloud sql databases list --instance=omni --project=argos-omni-dev`.
- Use Cloud SQL Studio in the Google Cloud console when you want an editor in the browser. It runs the same API as the same identity.
- Ask for only the rows and columns that you need, with a `WHERE` and a `LIMIT`. The query runs on the server that the apps use, so a scan of a large table slows them while it runs.
- Keep the rows that a query returns out of anything that you share when they hold personal data. Describe them instead, by a count or by the IDs of the rows, without their values.

## Change the rows

When your Google identity is a writer, change the rows of a database with the command of Read the rows. A writer may insert, update, and delete rows, and may change no table.

- Change only the rows that your task is about, and name them in the `WHERE` of each `UPDATE` and `DELETE`. A writer may change the rows of every database on the server, and the apps and other people use the same rows.
- Put rows that every environment needs in a migration instead, so that each environment gets them from a reviewed change.

## Get access

To read rows, add your Google identity to the list `database_readers` in the root `argos-omni-dev/cloud-sql/` of `argos-identity/argos-gcp-infra`. To also change rows, add it to `database_writers` in that root instead. Use the database only after the apply of that change finishes.

When a query fails with `permission denied for database` after that apply, run each migration job that `.github/scripts/cloud-run-resources.ts` names. Run each only when its last execution succeeded on the job as it is now. Such an execution prints the job's generation and `1`. Otherwise, leave that job to the next deploy of its image. A deploy changes the image of the job before it runs it, so a deploy that stopped in between leaves an image whose migration no execution applied, and running the job then applies it.

```sh
gcloud run jobs describe <migration job> --project=argos-omni-dev --region=us-central1 --format='value(metadata.generation)'
gcloud run jobs executions list --job=<migration job> --project=argos-omni-dev --region=us-central1 --limit=1 \
  --format='value(metadata.labels."run.googleapis.com/jobGeneration",status.succeededCount)'
gcloud run jobs execute <migration job> --project=argos-omni-dev --region=us-central1 --wait
```
