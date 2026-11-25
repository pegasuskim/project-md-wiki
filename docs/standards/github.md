# GitHub

## Branches

- Keep `main` and the `hotfix/<name>` branches as the only long-lived branches.
- Don't merge a `hotfix/<name>` branch to `main`, because `main` already has each fix that the branch holds.
- Make each change on a work branch from the branch that its PR merges to, with any name that doesn't start with `hotfix/`.

## Pull request

- Write the PR title and body by filling in the template in "PR template", which holds the rest of the rules for them.
- Start the title with `[<Issue ID>]`, using the ID of the Issue that the PR works on, so that Linear links the PR to that Issue.
- Put a magic word before an Issue ID in the body only for an Issue that the PR works on: `Fixes` when the PR finishes the Issue, which moves the Issue to Done when the PR merges, and `Part of` when it doesn't, which only links them.
- Write the title and body as the commit message on `main`, because the squash commit of the PR takes the title as its title and the body as its body.
- When you change what an open PR's title or body says, write both again by the template for the PR's whole diff instead of editing them, because an edited body keeps sentences that no longer hold. Keep from the old body only what still holds and the diff can't show, such as a decision and its reason or a check done by hand on code that no later commit changed.
- When the PR changes a document under `docs/`, give each changed part of the document a review comment on its lines that says what problem led to the change, because a reviewer judges each change to these documents on its own, where it is in the diff.

## Merge

The ruleset of `main` and the repository settings decide how a change reaches `main`:

- A change reaches `main` only through a PR, and the PR merges only by squash merge, only after the `quality` check passes, and only when it is up to date with `main`.

## Settings

- Change a repository setting or the ruleset only after the user confirms it. When the work turns out to need such a change while you work unattended, note it for the end-of-work report instead of stopping to ask.

## PR template

```markdown
<!--
Goal:
- This body exists so that a human reviewer can decide, with the least reading, whether to approve and on what conditions.
- Put in the open body what the reviewer needs to decide. Put in Details what they need to check that decision. Leave out anything that serves neither.
- The rules below apply this goal to the usual PR. When a rule doesn't fit your PR, follow the goal instead.
- Keep these in the open body, even to make the body shorter. Don't cut them, and don't fold them into Details:
  - a decision that a reviewer could disagree with, and the alternative it was compared with
  - the how-you-know tag of each claim that rests on a fact
  - each change to a sensitive area, whether or not a test checks it
  - a risk left after merge, a failed check, or a check that nobody ran, if it changes whether or how to approve

Repository rules:
- If the repository's docs require information, sections, or a title format, include them. Where to put that information and how much to write still follow the goal: put in the open body only the part the reviewer needs to decide, and put the rest in Details. Add a required section after Out of scope.
- If the docs define only part of a rule, follow them for that part and this template for the rest.

Title: <type>(<scope>): <changed behavior>
- Keep the title to 70 characters or fewer, not counting what the repository's docs add, such as a version or an Issue ID.
- Describe the behavior change, not the work you did: name the situation and what now happens in it.
- Use one of these types: feat, fix, refactor, perf, docs, test, chore.
- For the scope, use the service or module that changes. Leave it out if the change has no single scope.

Language:
- Write the title subject and the body in Korean.
- Keep these in English, as written: the type and scope, code identifiers, file paths, commands, glossary terms, the how-you-know tags such as (test), and the error messages and log lines that you quote.

Reader:
- The reader is a teammate who knows the product and the languages and frameworks of this repository, but has never seen this part of the code.
- Say what happens in plain words. Add code names only as pointers, in parentheses.
  If a word names how the code is built, such as route, handler, middleware, or component, write what that code checks, changes, or shows instead.
- Don't use terms from your own workflow, such as spec IDs or task names.

Claims and evidence:
- In the open body, write each claim that the reviewer needs, in one sentence, and how you know it.
- Keep in the open body a number that changes the decision, such as a limit, a size, or an exception to it. Write the number itself, not only that something grows, shrinks, or is limited.
- Never put a decision that a reviewer could disagree with only in Details. Details backs what the open body says and adds nothing new.
- Don't repeat what the diff and the Checks tab already show, such as checks that passed, unless the reviewer needs it to decide. A failure or a check that nobody ran that changes the decision goes in Review points, even if the Checks tab shows it.
- Write only what actually happened. If you don't know a reason, a test name, an impact, or a number, don't invent it.
- Write each command so that it runs from the repository root, with the directory or package it needs.
- Don't ask the reviewer to check or decide, such as "확인이 필요하다". If you couldn't decide, say what the options are and which one you chose.

Shape:
- Use two levels of bullets at most. Use bold only for item titles.
- Say each fact once, in the one place where it serves the reviewer. Don't say it again in another form.
  On a PR that changes a document under `docs/`, say the problem that led to each changed part only in the review comment that "Pull request" asks for on it.
- Delete a section or a Details subsection that has nothing to report. Review points always stays, because it always ends with the sensitive areas item.
- Before you finish, read the whole body from top to bottom as that teammate, and sort each sentence by the Goal, whatever its source or form: keep in the open body what they need to decide, move to Details what only checks or reproduces it, and cut the rest.
  Then check that everything in the Goal's keep list is in the open body, that each 막음, 감수, sensitive change, and claim that rests on a fact has its tag right after it, and that Details holds no decision that the open body doesn't show. Don't tag a choice or a value judgment that no check could confirm. Then delete every comment, including this one.
-->

## What this PR does

<!--
What changes?
- Say what this PR makes possible or different for a user or a caller, in as few sentences as that takes. Leave out code identifiers.
- Choose the form that makes the change clearest, and use one form for each fact:
  - sentences
  - a before-and-after line for each behavior change: "<situation>: <before> → <after>"
  - a list of parts, one line each, without file names, in an order where each part can be understood from the ones before it
  - a small table
  - a mermaid diagram, only when the order or concurrency of a flow is the point, with 7 nodes or fewer
- If the repository has several units that are used or deployed on their own, name the ones this PR changes, including the ones it changes through shared code, dependencies, or settings. If what gets redeployed or released differs from those, say so in one line.
- Add screenshots if the UI changes.
- Add these lines only when they help the reviewer decide what to read:
  - "동작은 같은 변경: <change>, <change>". If one of them is a choice that a reviewer could make differently, make it a Review point instead.
  - "<생성 | 이름 변경 | 이동>: <files> (<command or how>)"
-->

## Review points

<!--
What should the reviewer judge before approving, and how do you know?
- Put the item a reviewer is most likely to disagree with first. Always end with the sensitive areas item.
- Start each item with a bold title, a colon, and one or two sentences. Use sub-bullets only for a before-and-after line and for 막음 or 감수.
- Decisions that a reviewer would accept or reject together can share one item. Name each decision in it, with its alternative and reason.
- An item is one of these:
  - A choice that a reviewer could reasonably make differently. Name what you compared it with and why you chose it: "<대안> 대신 <고른 것>. <이유>."
    If you didn't seriously consider an alternative, name the obvious other way.
    A choice that follows the product, the linked Issue, or an obvious convention doesn't count. Don't write it anywhere.
    If the choice changes existing behavior and What this PR does doesn't show the before and after, add one before-and-after line.
    For a PR that adds or changes a rule, each rule decision that a reviewer could make differently must be in an item, alone or grouped with others.
  - A risk that could still go wrong after merge, even if all tests pass. Look for two requests or jobs at the same time, code outside the diff that uses the same data, deploy order and older clients, size, and what a user sees.
    End with one sub-bullet: "막음: <why it doesn't happen>" or "감수: <why it is acceptable now>", in one sentence.
  - A change to a sensitive area that no test checks. Say why it is safe.
  - A check that failed or that nobody ran, and why.
- Put how you know right after the claim it covers: each 막음 and 감수, each sensitive change, and each choice whose reason is a fact (a count, a measurement, how the code behaves).
  A tag covers only the claim right before it. If one sentence holds claims that you checked in different ways, tag each claim or split the sentence.
  The tags:
  - (test)
  - (test, broken on purpose): the test fails when you break the behavior on purpose
  - (checked by hand) or (checked by script)
  - (code only): you read the code, but nothing ran it
  Put the test name, the command, or the output in Details, not here.
- The last item is always "**건드린 민감 영역**: <what changed> (<tag>), <what changed> (<tag>)". Write what changed, not only the name of the area. If the PR changes no sensitive area, write "없음".
  A change that no test checks also needs its own item above.
  Sensitive areas: authentication and permissions, sessions and tokens, personal data, data that another service reads or syncs and other contracts between services, data deletion and migrations, public APIs and webhook, event, or log formats, environment variables and secrets, billing and pricing, the product's core decisions and the thresholds behind them, and any area that the repository's docs mark as sensitive.
- Leave out worries that apply to any change, such as "it fails if the database is down".
- If you have more than 5 items besides the sensitive areas item, split the PR, or add an item for the choice to keep it as one PR. Don't move items to Details to stay under it.
-->

- **<choice>**: <대안> 대신 <고른 것>. <이유>.
- **<risk>**: <what could go wrong>
  - 막음: <why it doesn't happen> (test)
- **건드린 민감 영역**: 없음

## Out of scope

<!--
What did you leave out on purpose?
- List what a reviewer might expect but you left out on purpose, one line each.
- Add the follow-up Issue if there is one.
-->

<!--
Details: how do you know?
Keep it folded. Write one line per item, and put long output in a nested <details>.
Write each piece of evidence once. If it backs more than one claim, write it under the first and point to it from the others.
- Evidence: for each item in Review points, copy its bold title, and list the evidence under it: test names, numbers and dates, code locations, and commands.
  For a choice, add the other alternatives you dropped and why.
  For a PR that adds or changes a rule, the rest of each rule decision's rationale goes under its item: what you researched, the alternatives you dropped, and measurements.
- Verification: the checks that back What this PR does and that Evidence and the Checks tab don't already show. Write "<passed | failed | not checked (reason)> <what you checked> — <command or method>".
  Add each existing test whose expected values you changed, and why.
  What to check depends on the type:
  - feat: each acceptance criterion in the linked Issue, or the behavior that this PR says it adds if there is no Issue.
  - fix: the bug reproduces before the fix and not after.
  - refactor: the tests that lock in the behavior that must not change pass before and after.
  - perf: the measurement method, the target, and the values before and after.
  - docs: the commands and examples in the docs run, and their links open.
  - test: the new tests fail when you break the tested behavior on purpose.
  - chore: for upgraded dependencies, the changelog entries that affect this code and how you checked each one.
-->

<details>
<summary>Details</summary>

### Evidence

- **<point>**
  -

### Verification

-

</details>
```
