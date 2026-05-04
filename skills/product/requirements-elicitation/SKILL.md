---
name: requirements-elicitation
description: Guide users through structured product requirements formulation from Business Requirements → User Requirements → Functional Requirements. Use when a user wants to define, describe, or document product requirements for a feature, module, or flow. Works standalone or in combination with notion-requirements-setup to write results directly to Notion.
---

# Requirements Elicitation Skill

This skill guides users through a top-down requirements process: Business Requirements (BR) → User Requirements (UR) → Functional Requirements (FR). It works as a Socratic dialogue — ask questions, let the user formulate, then refine together before writing to Notion.

## Core Philosophy

- **Never write requirements for the user** — ask questions, let them formulate, then help refine
- **Go top-down** — BR first, then UR, then FR. Never mix levels
- **One level at a time** — finish all BR before moving to UR
- **Validate before writing to Notion** — confirm formulations before creating records

## Levels Explained

| Level | Question it answers | Format |
|---|---|---|
| BR (Business Requirement) | Why does the platform need this? | "The platform must [verb] [object], because [business justification]." |
| UR (User Requirement) | What does the user want to accomplish? | "As a [role], I want to [action], so that [outcome]." |
| FR (Functional Requirement) | What must the system do? | "The system must [behavior] when [condition]." + Acceptance Criteria in Given/When/Then |

## Stage 0 — Setup

Before starting, determine:

1. **What feature/flow are we working on?** (e.g. "authentication page", "checkout flow", "onboarding")
2. **Is Notion connected?** If yes, ask for the parent page URL and use `notion-requirements-setup` to create the structure if it doesn't exist yet.
3. **What are the user roles in this product?** (e.g. Creator, Member, Admin, Learner). These will be used throughout UR formulation.

If Notion is not connected, work in conversation only and offer to export results as a markdown file at the end.

## Stage 1 — Business Requirements

### Goal
Identify why this feature/flow exists from a business perspective. No UI, no users, no implementation — only business value and constraints.

### How to guide

Start by asking the user to think about the business context:

1. **Who "owns" this feature from a business perspective?** (e.g. the platform, a specific tenant type, a paying customer segment)
2. **What business metric does this feature directly affect?** (conversion, retention, support load, revenue, data ownership)
3. **Are there business constraints that must be respected from the start?** (multi-tenancy, white-label, compliance, data sovereignty)

Then ask the user to formulate 2–4 BR in this format:
> *The platform must [verb] [object], because [business justification].*

### Reviewing BR formulations

A good BR:
- Uses an obligation verb: must, shall, should
- Names a concrete capability or guarantee — not a user feeling ("easy to use" is not a BR)
- Has a clear "because" that references business value, not implementation
- Does NOT mention UI, buttons, or technical implementation

A bad BR: "The platform should provide a simple login, because it affects conversion."
- "Simple" is a user perception, not a business guarantee
- Rewrite: "The platform must minimise friction in the authentication flow, because failed logins translate directly to lost revenue for course creators."

When the user writes a BR that is too vague or mixes levels, point it out and help them rewrite — don't rewrite it for them.

### Identifying Open Questions

After each BR, ask: "Is there anything about this requirement that is still unresolved or needs a decision before we can design it?"

Mark open questions explicitly. They block moving down to UR for that BR.

### Saving BR to Notion

Once all BR are agreed, create records in the Business Requirements database using `notion-create-pages`:
- `Name`: short title
- `Description`: full statement
- `Rationale`: the "because" part
- `Open questions`: any unresolved items
- `Feature`: the relevant module/feature tag
- `Status`: Draft

Save page URLs for use in UR relations.

## Stage 2 — User Requirements

### Goal
For each BR, identify who needs what and why — from the user's perspective. Focus on goals, not solutions.

### How to guide

For each BR, ask:
1. **Who are the users involved in this BR?** List all roles — some BR have multiple user types with different goals.
2. **Are there different scenarios?** (e.g. first-time vs. returning user, happy path vs. error state)
3. **For each user + scenario combination:** ask the user to write a User Story:

> *As a [role], I want to [action], so that [outcome].*

### Common patterns to watch for

**Multiple users per BR:** A single BR often spawns 2–4 UR across different roles or scenarios. This is expected and correct.

**Scenario splitting:** If one User Story contains "or" or describes two different triggers, split it into two separate UR.

**Unified flows:** If the product treats registration and login as a single flow (no separate "sign up" button), the UR should reflect this — don't write separate stories for register vs. login if the system handles them as one.

**Role naming:** Use the product's actual role names (Member, Creator, Admin) — not generic "user" or "customer".

### Acceptance Criteria at UR level

UR do not need full Given/When/Then — that's for FR. But each UR should have a clear "definition of done" statement: how do we know this story is satisfied?

### Saving UR to Notion

Once all UR for a BR are agreed, create records in the User Requirements database:
- `Name`: short title
- `Description`: full User Story
- `Business Requirement`: relation to the parent BR page URL
- `User role`: the role from the story
- `Feature`: same as parent BR
- `Status`: Draft

Save page URLs for use in FR relations.

## Stage 3 — Functional Requirements

### Goal
For each UR, define the specific system behaviors, rules, and edge cases. This is the most granular level — what engineers will implement.

### How to guide

For each UR, ask:
1. **What are the distinct methods or paths to fulfill this story?** (e.g. email+OTP, Google OAuth, Passkey are three separate FR for one UR about authentication)
2. **What is the happy path?**
3. **What are the error states and edge cases?**
4. **What must NOT happen?** (security, privacy, brand exposure rules)

For each FR, ask the user to describe:
- **Description:** "The system must [behavior] when [condition]."
- **Priority:** Must / Should / Could (MoSCoW lite)
- **Acceptance Criteria:** Given/When/Then scenarios

### Acceptance Criteria format

```
Given: [precondition / state of the world]
When: [user action or system event]
Then: [expected system behavior]
```

Each FR should have 2–6 Given/When/Then scenarios covering:
- Happy path (main success scenario)
- Error state (invalid input, expired token, network failure)
- Edge case (empty state, concurrent requests, boundary values)
- Security/privacy constraint (if relevant)

### Splitting FR correctly

Split into separate FR when:
- Different methods (email vs. OAuth vs. Passkey)
- Different triggers (user-initiated vs. system-triggered)
- Different error handling strategies
- Different priority levels (Must vs. Could)

Keep in one FR when:
- Same method, multiple scenarios (different outcomes of the same action)
- Related error states for the same flow

### Priority guidelines

| Priority | Meaning |
|---|---|
| Must | Without this, the feature cannot ship. Core functionality. |
| Should | Important but not blocking. Ship in first iteration if possible. |
| Could | Nice to have. Defer if time-constrained. |

### Saving FR to Notion

Once all FR for a UR are agreed, create records in the Functional Requirements database:
- `Name`: short title
- `Description`: system behavior statement
- `User Requirement`: relation to the parent UR page URL
- `Priority`: Must / Should / Could
- `Acceptance criteria`: full Given/When/Then text
- `Feature`: same as parent UR/BR
- `Status`: Draft

## Handling Revisions Mid-Process

Users often realize something needs to change as they go deeper. This is healthy.

**If a UR reveals a gap in BR:** Pause, go back to BR level, update or add the missing BR, then continue.

**If an FR reveals a gap in UR:** Same — pause, add or update the UR, then write the FR.

**If a decision changes an already-written requirement:** Update the existing record in Notion using `notion-update-page`. Never leave stale requirements.

## Traceability Check

At the end of any session, offer to do a traceability review:
- Every FR should link to a UR
- Every UR should link to a BR
- No BR should be "orphaned" (no UR derived from it)
- No FR should be "orphaned" (no UR parent)

## Exporting Without Notion

If Notion is not available, at the end of a session offer to export requirements as a markdown file:

```markdown
## Business Requirements

### BR-1: [Title]
**Statement:** ...
**Rationale:** ...
**Open questions:** ...

## User Requirements

### UR-1: [Title]
**Story:** As a [role], I want to [action], so that [outcome].
**Linked BR:** BR-1
**Role:** ...

## Functional Requirements

### FR-1: [Title]
**Description:** ...
**Priority:** Must
**Linked UR:** UR-1
**Acceptance Criteria:**
Given: ...
When: ...
Then: ...
```

## Tips for Effective Elicitation

**Socratic over prescriptive:** Ask "what do you think should happen?" before offering options. Users often know the answer — they just haven't articulated it yet.

**Name the level:** Always tell the user which level you're working on. "We're still at the business level — let's not think about buttons yet."

**Surface hidden assumptions:** When users say "obviously" or "of course", that's usually an assumption worth making explicit as a requirement.

**One decision at a time:** If a question has multiple sub-decisions, resolve them one at a time. Don't let open questions pile up.

**Open questions are first-class:** An unresolved open question at BR level should block UR formulation for that BR. Don't paper over it.

**Non-goals are requirements too:** Explicitly noting what is out of scope prevents scope creep. After defining each BR, ask: "Is there anything that explicitly should NOT be included?"
