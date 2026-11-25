# Deploy

## What a deploy picks

Keep a deploy to an environment picking only the images that a change since each image was last deployed to that environment can alter:

- Move an image's tag only after the deploy of that image succeeds, so that the next deploy picks again an image whose deploy failed or was skipped, even when the other images of its app shipped.
- Give each image one tag in each environment, with a prefix that names the step that ends the deploy of that image, such as the push to the registry or the start on the resource that runs the image. A deploy that picks from the tags of an earlier step skips the images that passed that step but whose deploy never finished.

## What a deploy changes

- Let a deploy change only the image of each container that it ships to and, in prod, the labels `commit` and `source` of that container's resource.
- Keep each GCP resource that a deploy ships to, with every setting of it other than what a deploy changes, such as the environment variables, the ports and the secrets of a Cloud Run service, in the Terraform of the repository `argos-identity/argos-gcp-infra`. The next apply of that Terraform puts back, without an error, any of those settings that was changed outside it.
- Give each value that differs between environments to the resource when its container starts, as an environment variable or a secret of Secret Manager. Keep the value out of the image, because the prod deploy of a commit of `main` builds again, from that commit, the image that dev ran.
- Apply the change to `argos-gcp-infra` that an app change needs in an environment before a deploy ships the app change to that environment. A deploy runs the new image with the settings that the resource has at that moment. For dev, apply the change to `argos-gcp-infra` before the app change reaches `main`, because a push to `main` starts the dev deploy.

## Deploying to prod

- Build each prod image from the commit that the prod deploy ships.
- Give the prod deploy no access to the dev project, so that nothing that can change dev can change what prod runs.
- Run the prod deploy for a commit of `main` after dev deployed that commit, and before dev deploys a later change to any image. The prod deploy ships only the images that dev shipped for that commit, as their `deploy/dev/` or `push/dev/` git tags show. It skips the other images without failing.
- To undo an image in prod, use `rollback.yml` to ship, without a build, the image that the prod registry holds for a commit that prod ran it from.
- Roll back only to a commit whose code works with the current schema of the prod database, because a rollback undoes no migration.
- After a rollback, run the prod deploy of `main` only for a commit that fixes the image. The rollback moves the tag of the image back, so the next prod deploy of `main` builds the image again from its commit.
- Undo a migration by shipping a new migration through dev.
- To see what a prod resource runs, read its labels `commit` and `source` with `gcloud run services list`, `gcloud run worker-pools list` or `gcloud run jobs list` and `--format='table(metadata.name, metadata.labels.commit, metadata.labels.source)'`, in the project that `deploy-prod.yml` deploys to. The labels name the commit and the source, `main` or `hotfix`, of the images that the newest run of `deploy-prod.yml` or `rollback.yml` gave the resource, even a failed run, while the `deploy/prod/<image>` git tag points at the newest deploy of each image that succeeded.

## Shipping a fix to prod alone

To ship a fix to prod without the changes that `main` got after the commits that prod runs, take these steps:

1. Merge the fix to `main`.
2. Push a new branch `hotfix/<name>` at the commit of the `deploy/prod/` git tags that has the others in its history.
3. On a work branch from `hotfix/<name>`, apply the fix with `git cherry-pick -x`.
4. Merge the work branch to `hotfix/<name>` through a PR.
5. Run `deploy-prod.yml` on `main` with `hotfix/<name>` in its input `ref`.

When the cherry-pick conflicts, write the fix for the code of prod on the work branch instead. End the message of that commit with `(cherry picked from commit <full SHA of the fix on main>)`. Without that line, the next prod deploy of `main` fails. When you can't write the fix for the code of prod, ship it through `main` and dev instead.
