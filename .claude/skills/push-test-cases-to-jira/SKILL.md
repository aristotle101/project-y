---
name: push-test-cases-to-jira
description: Generate test cases from a Jira story or feature description and create them as Sub-task issues in Jira. Can work from test cases already generated in the current conversation, or re-generate from a Jira issue key. Use when the user wants to push, create, or save test cases to Jira.
argument-hint: <parent-jira-key> e.g. SNP-12345
allowed-tools: mcp__bigc-jira__jira_get_issue,mcp__bigc-jira__jira_create_issue
---

# Push Test Cases to Jira as Sub-tasks

## Purpose
Take test cases (either already generated in conversation or freshly generated from a Jira story) and create each one as a **Subtask** issue in Jira, linked to the parent story.

## Input
Arguments: $ARGUMENTS

## Step 1 — Resolve Parent Issue

If `$ARGUMENTS` contains a Jira issue key (pattern `[A-Z]+-\d+`):
- Use `jira_get_issue` to fetch its `summary`, `issuetype`, and `project_key`.
- Confirm to the user: "สร้าง sub-task ใน **[KEY] [summary]** ใช่ไหม?" before proceeding.

If no Jira key in arguments, ask the user:
> "ต้องการสร้าง sub-task ใน Jira issue ไหน? (ระบุ issue key เช่น SNP-12345)"

## Step 2 — Obtain Test Cases

**If test cases already exist in this conversation** (a table with columns: Test ID, Title, Preconditions, Test Steps, Expected Result, Priority, Pyramid Layer):
- Use those test cases directly. Do NOT re-generate.
- Inform the user: "ใช้ test cases ที่ generate ไว้แล้ว ([N] cases)"

**If no test cases exist in conversation:**
- Fetch the parent issue's description via `jira_get_issue` (fields: `summary,description`).
- Generate test cases inline using the generation rules below.

### Inline Test Case Generation Rules (used only if no existing test cases)
- Happy paths, negative scenarios (≥40%), boundary value analysis, atomic steps.
- Tag each case with Pyramid Layer: Unit / Component (BE) / Component (FE) / Integration / E2E / Exploratory.
- ID format: `TC-F-###` (Functional), `TC-E-###` (Edge), `TC-ERR-###` (Error Handling).
- Present the table to the user and ask for confirmation before creating in Jira.

## Step 3 — Map and Create Sub-tasks

For each test case row, create a Jira Subtask with:

| Jira field | Value |
|---|---|
| `project_key` | Derived from parent issue key (e.g. `SNP-12345` → `SNP`) |
| `summary` | `[{Test ID}] {Title}` |
| `issue_type` | `Subtask` |
| `description` | See template below |
| `additional_fields` | `{"parent": "{parent_key}", "priority": {"name": "{mapped_priority}"}}` |

**Description template (Markdown):**
```
**Preconditions:** {Preconditions}

**Test Steps:**
{Test Steps — preserve numbered list}

**Expected Result:** {Expected Result}

**Pyramid Layer:** {Pyramid Layer}
```

**Priority mapping:**
- `High` → `High`
- `Med` or `Medium` → `Medium`
- `Low` → `Low`

## Step 4 — Create in Batches and Report

- Create sub-tasks one by one (do NOT use batch — individual creation allows per-item error handling).
- After each creation, note the returned issue key.
- After all creations, present a summary table:

| Test ID | Title | Jira Sub-task | Status |
|---|---|---|---|
| TC-F-001 | ... | SNP-12350 | ✓ Created |
| TC-ERR-001 | ... | — | ✗ Failed: {reason} |

- If any creation fails, show the error and ask the user if they want to retry.

## Rules
- Always confirm parent issue before creating anything.
- Never create issues in a project the parent issue does not belong to.
- Do not create duplicate sub-tasks — if the user runs this twice on the same parent, warn: "Parent issue already has sub-tasks. สร้างต่อได้ไหม?"
- Present total count at the end: "สร้างสำเร็จ {N}/{total} sub-tasks ใน {parent_key}"
