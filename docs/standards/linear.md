# Linear

## Team and text

- Create Issues in the `Squad Omni` team, whose Issue IDs start with `OMNI-`.
- Write titles, descriptions, and comments in Korean, or in English when the user asks.
- Write a number in Linear text without `#` before it, such as "row 34", because Linear turns `#34` into a link to pull request 34.
- In a Linear text and in each of its sections and list items, put the main point first and the specifics after it.
- Include only the words that a teammate needs. Leave out what Linear shows next to the text.
- Write two or more parallel points as bullets, one point to a bullet.

## Work phases

Linear is the team's shared record of who is doing what and how far along it is. Status only needs to be roughly right. If you're not sure which status fits, leave it and explain why in a comment.

### Before starting

The user is present. Show drafts and wait for confirmation.

- If the work has no Issue, find a matching open Issue or create one as described in "Issues and Sub-issues". Reuse a match unless the user says otherwise.
- Read the Issue's comments and linked Issues.
- Settle the acceptance criteria.
- Agree how far to take the work, usually up to an open PR.
- If the user already has 3 or more other In Progress Issues, mention it.

### While working

You work unattended, so never stop to ask.

- Move the Issue to In Progress when you start, and to In Review when the work reaches the agreed point. Move it back from In Review to In Progress when the work starts again. Never move it back from Done.
- If the work can't continue until another Issue is finished or someone replies, move the Issue to Blocked and post a Progress comment that fills in Blocking. If another Issue blocks it, also add a Blocked by relation to that Issue. Move it back to In Progress when the work continues.
- Check off action items as you finish them. At the end of each checked item, link the page that shows its result, so that a teammate sees the result from the item alone. When no page shows the result, write how to check it instead.
- When the plan changes, strike through what no longer holds instead of deleting it. Keep done action items checked. Add the new plan as new text and action items. Write code in struck text without backticks.
- Keep the comments described in "Comments".
- Note an out-of-scope problem that you find instead of fixing it, along with a due date at risk and any other change you would make in Linear.

### When finished

Send the user an end-of-work report with the result and your noted changes as drafts. Apply only what the user confirms.

- Note as a change for the report any change in Linear that "While working" doesn't let you make, such as a new Issue, a new priority or due date, or Done, Canceled or Duplicate. Leave Done to the GitHub integration for an Issue that an open PR names with `Fixes`, which moves the Issue to Done when the PR merges.
- When you propose Done, say whether the acceptance criteria are met, the deliverable is linked, and the review is approved.
- When you change a priority or due date or cancel an Issue, comment the reason.
- Mark a duplicate as a duplicate of the original instead of canceling it.

## Issues and Sub-issues

- An Issue is one deliverable, such as one PRD, one feature, or one bug fix. If the work has more than one deliverable, create one Issue for each.
- Create a Sub-issue only when an Issue is too large for one review. A Sub-issue is typically one PR.
- Name an Issue after one actual case, not after ongoing work such as operations, maintenance, or support, so that the Issue ends when the case is resolved.

### Fields

Fill in these fields for both Issues and Sub-issues.

| Field | How to fill it in |
| --- | --- |
| Title | State the action and its target specifically enough that the title alone tells what will be done. Start English titles with a verb, and end Korean titles with one. |
| Assignee | One person, the user by default. Leave it empty only in Backlog, and never for Urgent or High. |
| Priority | Required. Choose it as described in "Priority", and give a one-line reason. |
| Due date | Required for Urgent and High. Otherwise, set it only if the finish date is known. |
| Project and Milestone | Set them only if one fits. |
| Status | Todo if the Issue is ready to start. Otherwise, Backlog. |
| Description | Use the format below. |

```markdown
## Background
<the facts that a teammate needs to follow the problems and the work>

## Problems
- <a problem and what it causes>

## Action items
- [ ] <one action>

## Acceptance criteria
- <verifiable condition>

+++ Agent notes
<what only the person who does the work needs>
+++
```

Write the description so that a teammate who reads it once knows why the work is needed, what will be done, and how to tell that it is done:

- Write Background and Problems only if the work can have more than one reason or you are not sure that it has only one.
- Write Action items and Acceptance criteria in every description.
- If the facts you have do not show why the work is needed or how to tell that it is done, ask the person who requested the Issue before you write the description. Ask one short question at a time.
- Keep every fact that the reason for the work rests on.
- Under Background, state facts that a teammate can check, such as the current behavior, a number, or a linked source. If a fact that the reason for the work rests on cannot be checked, state who reported it.
- Under Problems, describe each problem as the current state, not as a missing solution. Write each problem as its own bullet, even when there is only one.
- When the description has Problems, write action items that together address every problem.
- Write each condition that the result must meet under Acceptance criteria.
- Put what only the person who does the work needs, such as which files change or how the code is laid out, in Agent notes, which Linear shows collapsed.

## Projects and Milestones

- Create a Project only for a time-bound effort, not for a team or ongoing work.
- Give a Project start and target dates and a goal in its summary. If these aren't known yet, create a research Issue first.
- Create a Milestone only for a long Project or a Project with an interim release, such as Beta or GA. Give a Milestone a target date.

## Priority

Set priority by when not doing this would impact the business.

- **Urgent**: Immediately. This includes outages, payment errors, and security incidents.
- **High**: Soon. This includes release-blocking bugs, work that blocks a teammate, and customer requests with a fixed deadline.
- **Medium**: Not immediately. This includes planned features, UX improvements, and internal tooling improvements.
- **Low**: Never. This includes non-urgent refactoring, documentation cleanup, and minor UI polish.

Apply these rules when you choose a priority:

- Feature work defaults to Medium. Raise it to High only if a delay would push back a Project or Milestone target date, or if the work blocks others.
- Raise a priority for its business impact, not for a requester's sense of urgency alone.

## Comments

Write each comment so that someone who never saw this conversation understands it.

### On an Issue assigned to the user or to nobody

Once work on the Issue has started, post a new comment at each of these points. Never edit an earlier comment. Before work on the Issue starts, change its description instead, because nobody has acted on the description yet. Post no comment for a step that changes nothing that a teammate knows about the Issue.

- When you make a decision or change the scope, post a separate comment for each decision or change:

  ```markdown
  **[Decision | Scope change]** <the decision or the change, in one line>
  - Why: <the reason>
  - Changes: <what changes for the Issue>
  - Link: <PR or commit>
  ```
- When you stop, at the agreed point or earlier:

  ```markdown
  **[Progress]** <the current state, in one line>
  - Done:
  - Left:
  - Blocking:
  ```

### On an Issue assigned to someone other than the user

Keep one comment of yours on the Issue. Edit that comment to correct or add to it, instead of posting another. The assignee then finds what you need in one place. Post or edit it only after the user confirms its text. While you work unattended, note the comment for the end-of-work report instead of stopping to ask. Leave to the assignee how to meet what you need. Write the comment by this template:

```markdown
**[Request]** <what you need from this Issue, in one line>
- Goal: <what your work will do>
- Needs: <each condition that must hold in this Issue for the Goal>
```
