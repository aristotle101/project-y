---
inclusion: manual
---

# Push Test Cases to Jira

## Purpose
Take test cases (either already generated in conversation or freshly generated from a Jira story) and create each one as a **Test case** issue in Jira, linked to the parent story.

## Step 1 — Resolve Parent Issue

Look for a Jira issue key (pattern `[A-Z]+-\d+`) in the user's message.

- Use `jira_get_issue` to fetch its `summary`, `issuetype`, and `project_key`.
- Confirm to the user: "สร้าง test case ใน **[KEY] [summary]** ใช่ไหม?" before proceeding.

If no Jira key is found, ask:
> "ต้องการสร้าง test case ใน Jira issue ไหน? (ระบุ issue key เช่น SNP-12345)"

## Step 2 — Obtain Test Cases

**If test cases already exist in this conversation** (a table with columns: Test ID, Title, Preconditions, Test Steps, Expected Result, Priority, Pyramid Layer):
- Use those test cases directly. Do NOT re-generate.
- Inform the user: "ใช้ test cases ที่ generate ไว้แล้ว ([N] cases)"

**If no test cases exist in conversation:**
- Fetch the parent issue's description via `jira_get_issue` (fields: `summary,description`).
- Generate test cases inline: happy paths, negative scenarios (≥40%), boundary value analysis, atomic steps.
- Tag each case with Pyramid Layer: Unit / Component (BE) / Component (FE) / Integration / E2E / Exploratory.
- ID format: `TC-F-###` (Functional), `TC-E-###` (Edge), `TC-ERR-###` (Error Handling).
- Present the table to the user and ask for confirmation before creating in Jira.

## Step 3 — Ask for Components

**Always ask** the user before setting Components:
> "ต้องการใส่ Components field ไหม? ถ้าใช่ระบุมาด้วย"

Never fill Components automatically.

## Step 4 — Map and Create Issues

For each test case row, create a Jira issue with:

| Jira field | Value |
|---|---|
| `project_key` | Derived from parent issue key (e.g. `SNP-12345` → `SNP`) |
| `summary` | `[{Test ID}] {Title}` |
| `issue_type` | `Test case` |
| `description` | See template below |
| `additional_fields` | See field mapping below |

### Description template

```
**🧪 Test Data**

```
(ข้อมูลทดสอบ ถ้ามี)
```

**🧭 Prerequisite**

{Preconditions — as bullet list}

**📋 Test Steps**

{Test Steps — preserve numbered list}

**✅ Expected Result**

{Expected Result — as bullet list}
```

### additional_fields mapping

```json
{
  "parent": "{parent_key}",
  "priority": {"name": "{mapped_priority}"},
  "customfield_10683": {"value": "{testing_pyramid_value}"},
  "customfield_10127": {"value": "{testing_type_value}"}
}
```

**Priority mapping:**
- `High` → `High`
- `Med` or `Medium` → `Medium`
- `Low` → `Low`

**Testing Pyramid (customfield_10683) mapping:**
| Pyramid Layer | Jira value |
|---|---|
| Unit | `Unit Tests` |
| Component (FE) | `Component Tests` |
| Component (BE) | `Component Tests` |
| Integration | `Integration Tests` |
| E2E | `E2E Tests` |
| Exploratory | `Exploratory Tests` |

**Testing Type (customfield_10127) mapping:**
| Testing Pyramid | Testing Type |
|---|---|
| Unit Tests | `Automate` |
| Component Tests | `Automate` |
| Integration Tests | `Automate` |
| E2E Tests | `Automate` |
| Exploratory Tests | `Manual` |

**Technical Stack Components (customfield_10649):**
- If parent summary contains `[APP]` → add `"customfield_10649": {"value": "MOBILE"}` to additional_fields
- Otherwise → omit this field

**Ignored fields:** `customfield_10650`, `customfield_10036`, `customfield_10050` — do not include.

If the user provided Components in Step 3, add `"components": [{"name": "{component}"}]` to additional_fields.

## Step 5 — Create and Report

- Create issues one by one (not batch) to allow per-item error handling.
- After each creation, note the returned issue key.
- After all creations, present a summary table:

| Test ID | Title | Jira Issue | Status |
|---|---|---|---|
| TC-F-001 | ... | SNP-12350 | ✓ Created |
| TC-ERR-001 | ... | — | ✗ Failed: {reason} |

- If any creation fails, show the error and ask if the user wants to retry.
- Present total count: "สร้างสำเร็จ {N}/{total} test cases ใน {parent_key}"

## Rules
- Always confirm parent issue before creating anything.
- Never create issues in a project the parent issue does not belong to.
- Do not create duplicate test cases — if the parent already has test cases, warn: "Parent issue already has test cases. สร้างต่อได้ไหม?"
- Never fill fields not listed above (no assignee, labels, fixVersions, etc.).
