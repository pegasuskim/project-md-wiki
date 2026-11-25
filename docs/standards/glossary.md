# Glossary

This glossary applies to the whole repository and to everywhere else that Omni is built and sold. People and code use each term only in the meaning given here.

In code, name types and models in PascalCase, DB enum values in UPPER_SNAKE, and API values in snake_case. Name a file, a declaration, or a test after what it is, not after where it came from, such as the repository or the version that the code was moved from.

## Omni

Omni is a ReviewOps Agent. On behalf of a Tenant, Omni collects information from people, analyzes it, and produces a result. Omni runs this process by deciding its own next actions.

## Account

The login identity created at signup. Signup creates only an Account. An Account can be an Admin in more than one Workspace.

## Tenant

The unit that contracts for Omni and is billed for it. One organization can have more than one Tenant. A Tenant can exist before it has a contract. A Tenant always has exactly one Workspace.

## Tenant Status

How a Tenant is using Omni. The value is one of the following.

- `trial`: The Tenant is trying Omni with limited features.
- `service`: The Tenant uses Omni as a paying customer.
- `closed`: The Tenant has ended its use of Omni.

## Workspace

The whole work environment in Omni that Admins manage. Projects belong to a Workspace. A Workspace is not the unit of contract or billing; that is the Tenant.

## Project

A group inside a Workspace that holds Workflows and their Cases. Each Workflow belongs to one Project.

## Admin

A person in a Workspace who uses Omni. An Account becomes an Admin of a Workspace in one of three ways: Omni creates the Workspace for it, it accepts an Invitation to the Workspace, or an Admin of the Workspace or a System Admin adds it to the Workspace. An Admin creates Projects and Workflows, registers Cases, and checks Analysis results. Admin is the highest role in a Workspace, and all Admins have the same permissions. A Workspace has no Owner role.

## Invitation

An Admin's offer, sent by email, for a person to join the Admin's Workspace as an Admin.

## System Admin

A person at ArgosIdentity who manages all Tenants.

## Applicant

A person outside the Workspace who applies to the Tenant for the review of a Case and provides the Case's information. One person can apply for the review of many Cases, including Cases of different Workflows.

## Workflow

A type of review that a Workspace runs (for example, "Supplier registration review"). A Workflow holds a Policy, a Playbook, an Output Schema, an AI Model, an Information Checklist, Sending Settings, and a default period for the Case Deadline.

## Policy

Review rules that an Admin writes in natural language in a Workflow. A Policy states what to evaluate and by what standard, what result the Admin wants, and in what order the Agent works. Omni interprets the Policy and creates a Playbook from it.

## Playbook

Instructions that Omni makes from the Policy and that define how to analyze a Case's information. Each Workflow has one Playbook. A Playbook consists of ordered Steps.

## Step

An ordered unit that makes up a Playbook. Each Step defines what to evaluate and by what standard. The Analysis Agent evaluates the Case's information for each Step and produces a Step Result.

## Output Schema

The JSON structure of Analysis results, defined for each Workflow. Analysis results follow this structure exactly. Each value carries the reasoning behind it and the Step it is based on.

## Tool

A capability that the Analysis Agent decides on its own to call while performing a Step. Examples: AML check, business status lookup, account holder lookup, and text comparison. Whether something is a Tool depends on who decides to call it, not on what it is built with.

- Is a Tool: something the Analysis Agent picks and calls when it needs it. Internal MCPs, CLIs, third-party APIs, and external Agents all count.
- Is not a Tool: something the system always runs at a fixed stage (for example, Preprocessing), and a handoff in which one Omni Agent passes work to another.

## AI Model

The LLM used for Analysis. Each Workflow has one AI Model.

## Information Checklist

The list of information to collect for a Case. The Information Checklist is not the Playbook, which defines how to analyze the information.

## Checklist Entry

One piece of information that the Information Checklist asks for. A Checklist Entry is either required or optional. A Checklist Entry is uploaded for a Case once the Case holds an Item that belongs to that Checklist Entry. Each Item belongs to at most one Checklist Entry, and each Item that the Applicant uploads through the Intake belongs to exactly one.

## Sending Settings

The rules for the emails that Omni sends to the Applicant of a Case. Each Case with an Information Request has its own set of these rules.

## Case

A unit for reviewing one set of information with one Workflow. A Case has one Applicant or none, and Omni also reviews a Case that has none. Reviewing the same information with another Workflow makes a separate Case. A Case holds its Items and its Analyses. An Analysis can run again on the same Case, so one Case can have many Analyses.

## Case Deadline

The date by which Omni asks the Applicant of a Case with an Information Request to reach Submission Complete.

## Information Request

Omni's request to the Applicant of a Case to submit information for the Case's Checklist Entries through the Intake.

## Submission Status

How far the Applicant has come in submitting a Case's information. A Case without an Information Request has no Submission Status. The value is one of the following.

- `requested`: Omni has made the Information Request, and the Applicant hasn't entered the Intake yet.
- `submitting`: The Applicant has entered the Intake, and the Case is waiting for Submission Complete.
- `submitted`: The Applicant has reached Submission Complete.

Submission Status is not Analysis Status.

## Submission Complete

The point at which every required Checklist Entry is uploaded for a Case and the Applicant has finished submitting information for that Case. Only a Case with an Information Request reaches Submission Complete.

## Item

One file, text, or JSON that Omni receives for a Case. An Item is material that the Analysis Agent uses when it performs a Step.

## Preprocessing

The stage that converts an Item into Markdown that the Analysis Agent can read. Preprocessing extracts the text, including text in images, identifies the document type, and creates a summary.

## Analysis

One run in which the Analysis Agent analyzes a Case's information according to the Playbook, together with its result. An Analysis can be run again on the same Case.

## Analysis Status

How far one Analysis has progressed. The value is one of the following.

- `pending`: The Analysis hasn't started yet.
- `processing`: The Analysis is running.
- `completed`: The Analysis has finished with a result.
- `failed`: The Analysis has ended without a result.

## Step Result

The result of one Step. The value is one of `passed`, `needs_review`, or `failed`. `needs_review` means a person needs to look at the result. It does not mean the review of the Case.

## Verdict

The conclusion of an Analysis. Omni computes the Verdict from Step Results by rules, without Admin confirmation. If any Step Result is `failed`, the Verdict is `rejected`. Otherwise, if any Step Result is `needs_review`, the Verdict is `pending_review`. If all Step Results are `passed`, the Verdict is `verified`. If the Analysis has not finished or has failed, the Verdict has no value (`null`).

## Analysis Score

A score from 0 to 100 that shows how completely an Analysis carried out its Playbook and filled its Output Schema. Analysis Score is not accuracy or AI confidence.

## Report

A PDF that contains the result of one Analysis.

## Dashboard

The Omni screen that Admins use. An Admin sees only their own Workspace's data. The Admin works with Workflows and Cases here.

## Omni Chat

The Dashboard screen where an Admin gives Omni work through Conversations.

## Conversation

The messages and documents exchanged between one Account and Omni in Omni Chat. A Conversation is not the Intake Conversation.

## Scope

The part of a Workspace that a message in Omni Chat applies to: all Projects, one Project, or one Workflow.

## Change Approval

The approval that Omni asks for in a Conversation before Omni changes or deletes something that already exists. The Admin in that Conversation gives it.

## System Admin Console

The Omni screen where System Admins manage all Tenants. The System Admin Console is not the Dashboard.

## Intake

The screen where an Applicant enters through an Access Link and submits one Case's information by talking with the Communication Agent. Only the Applicant of that Case can use it. The Intake is not the Dashboard.

## Intake Conversation

The conversation between the Applicant and the Communication Agent in the Intake of one Case. It holds both the Applicant's messages and the Communication Agent's replies. Each Case has at most one Intake Conversation.

## Access Link

The link to the Intake that opens one Case. Omni sends it to the Applicant in the Request Email.

## Access Code

The code that Omni sends to the Applicant's email address and that the Applicant enters after opening an Access Link.

## Request Email

The email in which Omni sends an Information Request to the Applicant of a Case. It lists the Checklist Entries, states the Case Deadline, and holds both the Access Link and an Access Code. The Communication Agent doesn't write it. The Request Email is not the Information Request. Sending the Request Email again doesn't make a new Information Request.

## Agent

A component that decides its next action on its own, runs it, checks the result, and repeats (an agentic loop). A single model call is not an Agent.

## Analysis Agent

The Agent that analyzes a Case's information according to the Playbook and produces a result.

## Communication Agent

The Agent that talks with an Applicant in the Intake to collect a Case's information. It doesn't judge the information that it collects. The Analysis Agent does.

## Forwarded Question

A question from an Applicant that the Communication Agent can't answer from the Information Checklist and passes to an Admin. It appears on the Dashboard.
