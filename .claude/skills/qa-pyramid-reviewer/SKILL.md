---
name: qa-pyramid-reviewer
description: Review an existing set of QA test cases against Big C's 5-layer "QA Way of Work 2025" testing pyramid — classify each case to its correct layer, flag mis-tagged cases and anti-patterns (ice-cream cone, hourglass, over-promoted E2E, non-atomic steps, vague expected results, coverage gaps), and recommend the missing tests. Use when the user wants to review, audit, critique, or health-check test cases (from conversation, a file, a Jira issue's sub-tasks, or a JQL result) — NOT to generate new ones from scratch.
argument-hint: Jira key (e.g. SNP-12345), a JQL query, a file path, or paste the test cases
allowed-tools: Read, mcp__bigc-jira__jira_get_issue, mcp__bigc-jira__jira_search, mcp__claude_ai_Atlassian_Rovo__getJiraIssue, mcp__claude_ai_Atlassian_Rovo__searchJiraIssuesUsingJql
---

# QA Pyramid Reviewer

## Purpose
A structured review skill for **existing** test cases. It does four things:
1. **Classify** every test case to the pyramid layer where it is actually best (cheapest, fastest) verified.
2. **Flag mis-tags** — where the case is tagged on one layer but belongs on another.
3. **Detect anti-patterns** — ice-cream cone, hourglass, over-promoted E2E, non-atomic steps, unverifiable expected results, duplicates, missing preconditions/data, wrong ownership.
4. **Recommend missing tests** — concrete gaps with a proposed layer and priority.

This skill does **not** generate a full suite from a raw feature. For that, use `generate-test-cases`. This skill critiques what already exists.

## Input
Target: $ARGUMENTS

Resolve the source of test cases in this order:

1. **Test cases already in this conversation** (a table with Test ID / Title / Steps / Expected Result / Pyramid Layer, or similar) → review those directly.
2. **`$ARGUMENTS` is a Jira key** (`[A-Z]+-\d+`) → use whichever Jira MCP "get issue" tool is available in this session for the parent, then fetch its sub-tasks/linked test cases. **Filter strictly to `issuetype = "Test case"`** — dependency/linked Tasks or Stories that merely reference the same parent are not test cases and must not be reviewed as such. If it is itself a Test case issue, review that one; if it is a story, review its child test cases.
3. **`$ARGUMENTS` is a JQL query** (contains `project =`, `sprint =`, `AND`, etc.) → use whichever Jira MCP "search" tool is available in this session (fields: `summary,description,status,customfield_10683,labels` — `customfield_10683` is this Jira site's "Test Type/Layer" field) and review the returned cases. Warn if results exceed one page and offer to page through. If `customfield_10683` comes back missing/null for all results, don't assume every case is untagged — this custom field ID is site-specific and may have changed; say so explicitly and ask the user to confirm the field before treating cases as untagged.
4. **`$ARGUMENTS` is a file path** → `Read` it.
5. **Nothing usable** → ask:
   > "อยากให้ review test case ชุดไหน? วาง table มา, บอก Jira key / JQL, หรือ path ไฟล์ก็ได้"

Language: write the review in the **same language as the test cases** (Thai stays Thai). Keep Test IDs and layer names as-is.

## Testing Pyramid Reference (source of truth)
Internal **QA_WayOfWork_2025 — Pyramid Paradigm**. Bottom = many/cheap tests, top = few/expensive tests.

- **Unit** *(Dev owns code+test)*: A single function/method/class in isolation. Mocked dependencies. Pure logic, calculations, BVA on isolated values, null/empty/threshold checks with no cross-service dependency. Runs in milliseconds.
- **Component (BE)** *(Dev owns code+test+assert)*: A single service/module/API endpoint against its contract. Real DB connections, not mocks. E.g. `POST /checkout` with valid/invalid payload → status + response shape.
- **Component (FE)** *(Dev owns code+test+assert)*: A single UI widget/page/form in isolation (Storybook/RTL-style). Rendering, validation states, disabled/enabled states, inline error display — without a real backend or full navigation.
- **Integration (Subsystem)** *(QA owns design+test+data)*: Real service-to-service interaction, no mocks — data flow, contracts, and consistency across two or more services (e.g., Checkout → Payment → DB; Gateway routing/auth). Most "business logic across systems" scenarios (discount rules, tax rules, race conditions, out-of-stock handling) belong here.
- **E2E** *(QA owns design+test+data)*: A complete user journey through the real stack as the customer experiences it (Home → Cart → Checkout, guest checkout, cross-app flows). Slow, flaky-prone, expensive — reserve for critical business/revenue paths only.
- **Exploratory** *(QA owns, manual only)*: Requires human judgment — visual/UX review, ambiguous copy, ad-hoc edge cases impractical or not worth automating.

**E2E-vs-Integration heuristic:** *"If swapping the UI for a curl command wouldn't change what the test proves, it belongs at Integration — not E2E."*

**Target shape:** ~70% Unit, Component close behind, Integration next, **E2E ≤ ~10%**, Exploratory only what genuinely needs a human. A case that fits multiple layers should live at the **lowest** layer that still fully verifies the behavior. **Integration ceiling:** Integration count must not exceed Unit+Component combined (see AP-12) — a suite that violates this needs cases rebalanced into lower layers, not just re-tagged.

## Review Workflow

### Step 1 — Normalize
Read every case into: Test ID, Title, Preconditions, Steps, Expected Result, Priority, Tagged Layer. If a field is missing, note it (it is itself a finding).

### Step 2 — Classify (actual layer)
For each case, decide the **correct** layer using the reference above and the curl heuristic. Compare to the **tagged** layer:
- Match → ✅
- Different → ⚠️ **Mis-tagged** (record Tagged → Correct + one-line reason).
- Tagged value isn't one of the 6 pyramid layers (e.g. an ad-hoc bucket like "Happy Case" / "Edge Case" from a legacy table) → don't drop it and don't force-fit it into a pyramid layer for the Tagged column. Count it as **Untagged** in the Distribution table instead, so suite totals still reconcile.

### Step 3 — Detect anti-patterns
Check each case and the suite as a whole against the catalog below. Assign 🔴 (must fix) or ⚠️ (should fix).

### Step 4 — Coverage gap analysis
- Every acceptance criterion / core flow has at least one case?
- Negative scenarios ≥ ~40% of cases? BVA present on real thresholds? Error-handling/graceful-degradation covered?
- Any layer suspiciously empty (e.g. business rules exist but zero Integration; UI states exist but zero Component-FE)?
- **Integration recheck (mandatory):** For every distinct integration point identified in the feature (each service-to-service call, API contract, or cross-service data flow), verify that the Integration layer contains **at least one negative case** (bad payload, auth failure, timeout, downstream error response, network failure) **and at least one edge case** (boundary value, empty/null response, duplicate event, race condition). If either is missing for any integration point → flag AP-11 🔴.
- **Integration ceiling check (mandatory):** After re-classification, compare the Integration count to Unit+Component combined. If Integration is higher → flag AP-12 🔴 and recommend which Integration cases should be split/rewritten down into Unit/Component instead — not just re-tagged.

### Step 5 — Recommend missing tests
Propose concrete cases for the gaps: `Title → proposed Layer → Priority → one-line why`. Use existing ID prefixes (`TC-F`, `TC-E`, `TC-ERR`).

## Anti-Pattern Catalog

| Code | Anti-pattern | How to spot it | Severity |
|---|---|---|---|
| AP-1 | **Ice-cream cone** | E2E + Exploratory dominate; Unit/Component thin. E2E > ~10%. | 🔴 |
| AP-2 | **Hourglass** | Healthy Unit + E2E but the Integration/Component middle is missing. | 🔴 |
| AP-3 | **Over-promoted E2E** | E2E case that passes the curl heuristic — no UI/session/gateway behavior is actually being proven. | 🔴 |
| AP-4 | **Mis-tagged layer** | Tagged layer ≠ actual best layer (e.g. pure calc tagged Integration; multi-service rule tagged Unit). | ⚠️ |
| AP-5 | **Non-atomic** | One case asserts several independent outcomes → can't tell what failed. | ⚠️ |
| AP-6 | **Unverifiable expected result** | "works correctly", "displays properly", no measurable/observable checkpoint. **Exception:** an Expected Result that states a concrete best-guess checkpoint and appends an explicit `⚠️ NEEDS CONFIRMATION: ...` marker is not AP-6 — that's an intentionally flagged uncertainty from `generate-test-cases`, not vague writing. | 🔴 |
| AP-7 | **Missing preconditions / test data** | Case can't be executed as written. | ⚠️ |
| AP-8 | **Redundant duplication** | Same behavior re-tested at multiple layers with no added risk coverage. | ⚠️ |
| AP-9 | **Wrong ownership** | Dev-layer test (Unit/Component) written as QA manual, or Exploratory being automated. | ⚠️ |
| AP-10 | **Happy-path-only** | No negative / boundary / error cases for a flow that clearly has failure modes. | 🔴 |
| AP-11 | **Integration point missing negative/edge** | An integration point (service-to-service call, API contract, data flow across two systems) has zero negative cases (bad payload, auth failure, timeout, downstream error) or zero edge cases (boundary values, empty/null responses, race conditions). Every distinct integration point must have ≥1 negative **and** ≥1 edge case. | 🔴 |
| AP-12 | **Integration-heavy / inverted pyramid** | After re-classification, the Integration layer count exceeds the Unit + Component (BE+FE) combined count. | 🔴 |

## Output Format

Present the review as **one report** in this order:

**1. Verdict line** — overall pyramid health in one sentence + shape label (Healthy / Ice-cream cone / Hourglass / Top-heavy).

**2. Distribution table** — tagged vs. recommended counts per layer:

| Layer | Tagged | After re-classify | Target |
|---|---|---|---|
| Unit | … | … | ~70% base |
| Component (BE) | … | … | high |
| Component (FE) | … | … | high |
| Integration | … | … | medium |
| E2E | … | … | ≤10% |
| Exploratory | … | … | minimal |
| Untagged | … | — | 0 |

`Untagged` only ever has a Tagged count — every case gets re-classified into a real layer, so its "After re-classify" cell is always `—`.

**3. Findings table** — one row per flagged case (skip clean cases). Severity comes straight from the Anti-Pattern Catalog; when a row cites more than one AP code, show the highest severity among them (🔴 beats ⚠️):

| Test ID | Severity | Finding | Anti-pattern | Tagged → Correct | Fix |
|---|---|---|---|---|---|
| TC-E2E-002 | 🔴 | Proves API contract only, no UI behavior | AP-3, AP-4 | E2E → Integration | Move to Integration; drop UI steps |

**4. Coverage gaps & recommended tests:**

| Proposed Title | Layer | Priority | Why it's missing |
|---|---|---|---|
| [checkout] reject negative quantity | Component (BE) | High | No negative case on qty validation |

**5. Summary** — counts: `N reviewed · X mis-tagged · Y anti-patterns · Z recommended`. `Y` = the number of **distinct AP codes** that appear at least once across the whole suite (not the number of finding rows, and not total occurrences — AP-4 showing up on three different cases still counts once toward Y). Lead with the single highest-impact action.

## Rules
- **Read-only.** Never edit Jira issues or files. Report only.
- Judge against the pyramid reference, not personal taste. Every 🔴/⚠️ must cite an AP code or a pyramid rule.
- Don't rewrite whole cases; give the minimal fix. If the user wants rewritten cases, hand off to `generate-test-cases`.
- When a case legitimately fits two layers, say so and recommend the lower one — don't force a single answer.
- Be direct. If the suite is healthy, say so plainly and stop; don't invent findings to fill the table.
