---
name: notion-requirements-setup
description: Set up a three-tier requirements database structure in Notion (Business Requirements, User Requirements, Functional Requirements) with relations, ID prefixes, and inline views on a single page. Use when a user wants to track product requirements in Notion, set up a new project's requirements structure, or replicate a BR→UR→FR hierarchy.
---

# Notion Requirements Setup Skill

This skill creates a complete three-tier requirements structure in Notion: three linked databases (BR, UR, FR) displayed as inline views on a single page.

## When to Use

Trigger when the user:
- Wants to set up product requirements tracking in Notion
- Mentions "BR", "UR", "FR", "business requirements", "user stories", "functional requirements"
- Wants to replicate a requirements structure for a new project
- Has an existing "Requirements" or "Продукт" page and wants to add structure to it

## What You Need From the User

Before starting, ask for:
1. **Parent page URL** — where should the three databases live? (e.g. a "Requirements" page or "Product" section)
2. **Feature prefix** (optional) — e.g. "AUTH", "BILLING". Used for the Feature select field options. Can be added later.

If the user provides a Notion URL, use `notion-fetch` to get the page ID.

## Step 1 — Create the Three Databases

Create all three databases as children of the parent page. Use `notion-create-database` for each.

### Business Requirements (BR)

```
title: "Business Requirements"
parent: { page_id: <parent_page_id> }
schema: CREATE TABLE (
  "Name" TITLE,
  "ID" UNIQUE_ID PREFIX 'BR',
  "Feature" SELECT('Auth':blue, 'Billing':green, 'Automations':orange, 'Workspace':purple, 'Other':gray),
  "Status" SELECT('Draft':gray, 'Approved':green, 'Deprecated':red),
  "Description" RICH_TEXT,
  "Rationale" RICH_TEXT,
  "Open questions" RICH_TEXT
)
```

**Save the returned `data_source_id` as `BR_DS_ID`.**

### User Requirements (UR)

```
title: "User Requirements"
parent: { page_id: <parent_page_id> }
schema: CREATE TABLE (
  "Name" TITLE,
  "ID" UNIQUE_ID PREFIX 'UR',
  "Business Requirement" RELATION('<BR_DS_ID>'),
  "Feature" SELECT('Auth':blue, 'Billing':green, 'Automations':orange, 'Workspace':purple, 'Other':gray),
  "Status" SELECT('Draft':gray, 'Approved':green, 'Deprecated':red),
  "User role" SELECT('Member':blue, 'Creator':purple, 'Admin':orange),
  "Description" RICH_TEXT
)
```

**Save the returned `data_source_id` as `UR_DS_ID`.**

### Functional Requirements (FR)

```
title: "Functional Requirements"
parent: { page_id: <parent_page_id> }
schema: CREATE TABLE (
  "Name" TITLE,
  "ID" UNIQUE_ID PREFIX 'FR',
  "User Requirement" RELATION('<UR_DS_ID>'),
  "Feature" SELECT('Auth':blue, 'Billing':green, 'Automations':orange, 'Workspace':purple, 'Other':gray),
  "Status" SELECT('Draft':gray, 'Approved':green, 'Deprecated':red),
  "Priority" SELECT('Must':red, 'Should':orange, 'Could':gray),
  "Description" RICH_TEXT,
  "Acceptance criteria" RICH_TEXT
)
```

## Step 2 — Create Inline Views on the Parent Page

Use `notion-create-view` three times with `parent_page_id` (not `database_id`) so the tables appear inline on the parent page.

```
# BR view
parent_page_id: <parent_page_id>
data_source_id: <BR_DS_ID>
name: "Business Requirements"
type: table
configure: SHOW "ID", "Feature", "Status", "Description", "Rationale", "Open questions"

# UR view
parent_page_id: <parent_page_id>
data_source_id: <UR_DS_ID>
name: "User Requirements"
type: table
configure: SHOW "ID", "Business Requirement", "User role", "Feature", "Status", "Description"

# FR view
parent_page_id: <parent_page_id>
data_source_id: <FR_DS_ID>
name: "Functional Requirements"
type: table
configure: SHOW "ID", "User Requirement", "Feature", "Priority", "Status", "Description", "Acceptance criteria"
```

## Step 3 — Explain Page Templates (Manual Step)

Notion API does not support creating page templates programmatically. Tell the user to create them manually — it takes ~5 minutes for all three databases.

Instructions to give the user:

> **For each database (BR, UR, FR):**
> 1. Open the database
> 2. Click the ▾ arrow next to the **New** button (top right)
> 3. Select **+ New template**
> 4. Name it and paste the template content below

**BR Template — "New Business Requirement":**
```
## Statement
The platform must [verb] [object], because [business justification].

## Rationale
Why does this matter to the business? What happens if we don't do it?

## Open questions
- [ ] 
```

**UR Template — "New User Story":**
```
## User story
As a [role], I want to [action], so that [outcome].

## Acceptance criteria
- [ ] Given [context] / When [action] / Then [result]
- [ ] 

## Notes
```

**FR Template — "New Functional Requirement":**
```
## Description
The system must [behavior] when [condition].

## Acceptance criteria
Given: 
When: 
Then: 

## Edge cases
- [ ] 

## Out of scope
- 
```

## Step 4 — Confirm and Hand Off

After creating all three databases and views, confirm to the user:
- ✅ Three databases created: BR, UR, FR
- ✅ Inline views added to the parent page
- ⚠️ Page templates need to be created manually (5 min)

Share the parent page URL and tell them they can start adding requirements immediately using the `requirements-elicitation` skill.

## Notes on Relations

- UR has a `Relation → BR` field. When creating UR records programmatically, pass the BR page URL as the relation value.
- FR has a `Relation → UR` field. When creating FR records programmatically, pass the UR page URL as the relation value.
- There is no direct FR → BR relation by design. The chain is FR → UR → BR (two clicks to trace back to the business reason).
- When creating pages via API, the `data_source_id` for the parent must be the collection ID, not the database ID.

## Troubleshooting

**Relation field not linking correctly:** Ensure you're passing the full Notion page URL (e.g. `https://www.notion.so/abc123...`) as the relation value, not just the UUID.

**Inline views not appearing:** The `parent_page_id` parameter must be a regular page ID, not a database ID. Verify you're using the parent requirements page ID.

**Feature options mismatch:** The Feature select options are suggestions. Update them to match the actual modules of the project before starting elicitation.
