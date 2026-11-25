# Writing docs

These rules apply to every document under the root `docs/`, and to the root `AGENTS.md`.

## What a document holds

Put a sentence in a document only when each of these holds for it. When one doesn't, put the sentence where that one says:

- It is an instruction, the reason for one, or a definition or a fact. A sentence that tells what the code does goes nowhere, because the code shows it. Any other sentence, such as a requirement, a plan, or how a decision was reached, goes where the work is tracked, outside the repository.
- No check in the repository fails a change that breaks it, and it isn't about one place in the code. A rule that a type, a test, a lint rule, a CI check, or a setting fails a change for stays in that check. A rule or a reason about one place in the code goes in a comment there. When a check that fails exactly the changes that break a rule, no fewer and no more, would cost little to keep, write the check instead of the sentence. A rule that nothing in the code holds stays in its document even when the files already follow it, because the files show what the code does now, not what it must do.
- Any rule in it applies to work that a reader can do in the repository as it is. A rule for other work, such as a deploy to an environment that doesn't exist, goes in the Issue of that work. The PR that makes the work possible adds the rule to a document.
- It stays true after the work that adds it ends, so it doesn't say that something came from another repository of ours, such as the repository that the code was moved from. Any other sentence goes where the work is tracked.
- It applies to every app that its document covers, so that an app added later follows it without a change to the document. A rule or a fact that only one app needs, such as one about the app's domain or about an interface that only the app has, goes in the code of that app, as a check or in the declaration comment of the code that it is about, where the next reader of that code finds it.

In a change to a document, check the other sentences of each section that the change touches against these conditions too. Move a sentence that fails one where that one says.

## Which document holds it

- Keep every document in the root `docs/`, in the folder that the index names for its rules.
- Write a rule, its condition and its examples without the name of an app or a term of one app's domain. Name in the condition what the work touches instead, such as an environment variable or a class that implements a port.
- Put every rule and fact that one kind of work follows in the one document whose row in the index matches that work, so that its reader reads one document for it.
- Refer to a document other than the index and the root `AGENTS.md` in any way, even by describing it, only from the index or the root `AGENTS.md`, so that which documents a task needs is kept in one place.
- Put a rule that the work on both frameworks follows in `docs/standards/`, unless its subject has a document in each framework's folder. Write such a rule in each of those documents instead. Change every copy of it in the same change.
- Keep the procedure that every task follows in the root `AGENTS.md` only, because an agent reads it wherever in the repository it starts.

## Keeping the documents

Follow these rules in any change to the layout of the documents or to an index.

- Give `docs/` one index, its `README.md`, with a "When to read" table whose rows name the work that the reader is doing. State what goes where in `docs/` in the index.
- Name a document after the subject that its rules share, so that its name alone tells the reader what it covers and doesn't read as an Omni term or as another document. Rename or split a document when its rules outgrow its name.
- Update the index in the same change as the layout of its documents.

## How a rule reads

Write each rule as the outcome to keep, in a sentence that the reader can apply to cases it doesn't name. Name a case only when the outcome leaves it unclear. Write a prohibition only when it states the outcome more exactly.

Give the reason for a rule only when a reader has broken it, or when breaking it raises no error and the usual way of doing the work, such as a tool's default, breaks it. When a reader misreads a rule, rewrite the rule instead of adding a reason. Reword a sentence only when a reader got it wrong or it no longer holds.

- Write a rule as an instruction and a fact as a statement.
- Define the cases that a rule covers by what they share, in a sentence that allows one reading, rather than by listing them or by naming the one kind that exists today, such as HTTP for the requests that an app receives, so that a case added later falls under the rule. Name a case after `such as` only when a reader would misread the definition without it. Never let the cases that you name stand in for the definition.
- Write what would be a nested list under an item as sentences of the item, or give it a section of its own.
- Keep a rule and its condition in one item. State what several rules share, such as a condition or an outcome, once above them.
- Write in English, in the active voice, with at most one instruction in each sentence, so that each sentence states who must do what. A reason or a fact can share the sentence with its instruction.
- Write an Omni term capitalized and only in its defined meaning. Write the same word in lowercase when you mean something else, such as a GitHub Actions workflow.
- Refer to a section by its name instead of numbering the sections. Adding or moving a section changes the numbers after it, so a reference by number, wherever it is, then points to another section without an error.
