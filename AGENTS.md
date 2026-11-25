# project name

These rules apply to every task in the repository, from its start to its finish.

## Start

1. Decide which packages and files the change touches before you read the documents.
2. Find the documents for the task in `docs/README.md` at the repository root and read each one whose row matches the task, even when no one named it.

## While working

- When the work comes to match another row of the index, read that row's documents before you go on.
- Read a document before you change it, even when its row in the index doesn't match the task.
- Check a document or a comment against the code before you act on it. When they disagree on what the task touches, decide which one is right and make the other match it before you go on. When you can't tell, change neither and report the mismatch.

## Finish

- Before you report the task as done, run the checks that CI runs on the change and make them pass: every step of `.github/workflows/ci.yml` that applies to the change, not only `pnpm ci:check`.
- When you can't make a check pass, don't get around it, such as by committing with `--no-verify` or removing a failing test. A check that you get around stops catching the problem for every later change.
- Fix the debt that the change introduces in the same change. When you can't make a check pass or can't finish the task, stop and report what is left to whoever gave you the task, rather than leaving it in the code.
- When the change adds or changes a rule, update every document that holds the rule in the same change.
- When the change adds or changes a document, give the changed documents to two agents that haven't seen your work. Ask the first to plan the task of the Linear Issue from those documents. Ask the second to find each sentence in them that breaks a rule of `docs/standards/writing-docs.md`. Fix each place where the plan goes wrong and each sentence that the second agent finds. An author misses those sentences in their own change even after reading `writing-docs.md`, and the first agent can plan correctly from a document that has them.
- Don't merge a PR whose change touches `docs/standards/glossary-sample.md`, even when you may merge other PRs. Report the open PR to whoever gave you the task, so that a person reviews and merges it. A term's definition decides how code, documents, and the product use the term, so a wrong one spreads past the change.
