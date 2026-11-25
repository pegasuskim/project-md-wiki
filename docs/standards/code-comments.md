# Code comments

This document applies to the comments in TypeScript code everywhere in the repository, whoever writes them, people and coding agents alike.

## Write a comment

### Decide what to write

Say what the code does, and how, with the code itself: a clearer name, an explaining variable, a smaller function, a more precise type that makes the mistake impossible, a runtime check that fails as soon as the mistake happens, or a test that shows the cases it handles. Leave it out of comments, because a comment that restates the code goes wrong without an error when the code changes.

Write a comment only for what the code can't say and the next reader would otherwise change wrongly, and only when it is one of these. Declaration comments have their own rules below.

- **Why the code exists or takes this form**, including the reason for a chosen value, when the code doesn't make it obvious.
- **An external constraint**: the spec that the code follows, the third-party behavior that it works around, or the source of third-party code copied in, with the notice that its license requires.
- **A warning that the code can't enforce**: what breaks, in one line, when the code changes.

Leave out a reason that a reader who knows the language sees from the code, such as using a `Map` for fast lookups.

A comment holds only what stays true for as long as the code exists. The commit message and the PR hold the history of the change and the context of the task, and the work that is left lives where the work is tracked, so a comment holds no code that was taken out, no task to do later, and no reference to the work, such as an Issue ID or the number of an item in a PRD, nor to where the code came from in another repository of ours, such as the repository or the version that it was moved from, which changes or goes away while this code stays.

### Make the comment readable without context

The next reader may be an agent that opened only this part of the file and knows nothing about your task.

- Make each comment understandable on its own, with the reason in the comment itself and clear about which code it is about. A link or a pointer to other code can support the reason but can't replace it.
- Write in English.
- Keep the comment short, and state facts, such as what is true and what breaks, without hedges.

### Write a declaration comment

- Write a declaration comment (`/** */`) only when a clearer name or type can't show how to use the declaration, whether or not it is exported.
- Write the contract that the types can't express: the errors it throws, side effects, concurrency conditions, and other constraints. Describe the contract and not the implementation, which can change while the contract stays the same.
- Write the contract once, on the declaration that callers use, such as an interface rather than its implementation. On an override, write only how its contract differs.
- Write `@param` and `@returns` only when they add something that the name and the type don't, such as a unit, an allowed range, or what `null` means.
- Put the declaration comment above the decorators, because tools don't read a comment between a decorator and the declaration as documentation.

## Work with existing comments

- When a comment and the code disagree, a comment that describes what the code does is usually the wrong one, and the code can be the bug when the comment states what the code must do.
- When a change touches a comment, such as by moving it with its code, make all of it follow these rules, and keep the information in it that is true and that these rules allow.
- When you change or copy code, update or delete its comments in the same change, including a warning that your change makes untrue.
