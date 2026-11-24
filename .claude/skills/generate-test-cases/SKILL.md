---
name: generate-test-cases
description: Generate high-coverage test cases or test scenarios from a feature description, Jira user story, task, or code. Auto-detects input level — User Story → Test Scenarios (E2E, Happy path, Edge case); Task → Test Cases (Integration testing) — mapped to the company's 5-layer testing pyramid (QA_WayOfWork_2025), with full AC traceability. Use when the user asks to create manual test cases, functional test specs, test scenarios, or QA test coverage documentation.
argument-hint: feature description, Jira user story, task, or code
allowed-tools: Read, Grep, Glob
---

# Test Case & Scenario Generation (2-Level)

## Purpose
Transform feature descriptions, Jira user stories, tasks, or raw code into clear, atomic, executable, and reviewable test artifacts. Applies the correct level of detail based on the input source — broad scenarios for stories, specific cases for tasks.

## Input
Target: $ARGUMENTS

If no input was provided, ask the user for the feature description, user story, task, or code to analyze.

---

## Step 0: Input Type Detection (Mandatory — Do This First)

Determine whether the input is a **User Story** or a **Task** before proceeding. This determines which path to follow.

**Signals of a User Story:**
- Contains "As a [role], I want...", "User story", "Story"
- AC describes business outcomes or user-facing behavior
- Jira issue type = Story or Epic
- No specific API endpoint, service name, or implementation detail
- Scope spans multiple services or a full user journey

**Signals of a Task:**
- Contains "Task", "Sub-task", or references specific implementation
- AC describes technical behavior, API contracts, or data validation
- Jira issue type = Task or Sub-task
- References specific services, endpoints, or internal logic
- Scope is a single service or feature slice

**Decision:**
- Input is a **User Story** → follow **Story Path** below
- Input is a **Task** → follow **Task Path** below
- Input is ambiguous → ask: "Input นี้เป็น User Story หรือ Task ครับ?"

---

## Requirement Analysis (Mandatory — Both Paths)

Run this right after the Analyze Phase, before Coverage Rules, in both Story and Task paths. Findings must drive case creation — an unresolved finding still gets flagged in the final output, never silently dropped.

1. **Ambiguous or incomplete requirements** — anything in the input readable more than one way, or missing the detail needed to write a deterministic test
2. **Edge cases not explicitly covered** — conditions the requirement implies but never states
3. **Contradictions or missing business rules** — including conflicts against any other story/task visible in context (e.g. two stories covering overlapping scope with different rules)
4. **Dependencies between features** — where one feature's failure or change affects another feature's expected behavior

Format each finding as:
- `AMBIGUOUS: [requirement] → [what needs clarification]`
- `MISSING: [edge case] → [why it matters for testing]`
- `CONTRADICTION: [requirement A] ↔ [requirement B] → [which one governs, or flag as unresolved]`
- `DEPENDENCY: [feature A] ↔ [feature B] → [risk if one fails]`

**Every finding must resolve to one of:**
- A test case created to cover it (tagged `— qa-insight` or `— gap` per AC Traceability below), or
- An explicit `⚠️ NOT COVERED — requires clarification` flag when it can't yet become a case (e.g. an unresolved CONTRADICTION)

Present this as its own section in the output, **before** the test case/scenario table.

---

## Story Path — Test Scenario Generation

**Output goal:** Broad test scenarios covering E2E, Happy path, and Edge cases — user-facing perspective, not deep technical steps.

### Story Analyze Phase
**Core rule: Design scenarios from behavior first. Map to ACs after — not before.**

1. Parse the user story and identify the **actor**, **goal**, and **business outcome**
2. Identify user journeys: what does the actor do from start to finish?
3. Identify **edge conditions** in the business rules (not implementation detail)
4. Note integration points at the boundary level (e.g., "calls payment service") — not internal service logic
5. Extract and ID every acceptance criterion — used for traceability mapping only, not to drive scenario creation
6. Run **Requirement Analysis** (see section above) before moving to Coverage Rules

### Story Coverage Rules
- **Happy Path**: Core journeys where the actor achieves the goal
- **Negative / Rejection Scenarios**: At least 30% of scenarios must test the business rules that block or reject the user
- **Edge Cases**: Boundary conditions visible to the user or business (max qty, expired coupon, zero items, etc.)
- **E2E journeys**: When the scenario spans multiple steps the user takes (e.g., add to cart → checkout → confirm)
- **Do NOT generate** deep technical steps, specific API calls, or internal service behavior — those belong in Task-level test cases

### Story Output Format

**Before the table:** Present the Requirement Analysis findings (see section above), then list extracted AC IDs and their text.

| Scenario ID | Scenario Name | AC Ref | Type | Description | Priority |
| :--- | :--- | :--- | :--- | :--- | :--- |
| TS-F-001 | [Short name] | AC-1 | Happy path | What the user does and what they expect to see | High/Med/Low |
| TS-E-001 | [Short name] | AC-2 | Edge case | Boundary or unusual condition | Med |
| TS-ERR-001 | [Short name] | — implicit | Negative | Business rule rejection or blocked state | High |

**Type values:** `Happy path`, `Negative`, `Edge case`, `E2E`

**After the table:**
- **AC traceability matrix** — one row per AC: `AC-1 | TS-F-001, TS-ERR-001 | covered`  
  Mark each AC: `covered`, `happy-path only`, or `⚠️ NOT COVERED`
- **Pyramid layer note** — Story scenarios map primarily to: E2E, Exploratory, and the Integration layer for cross-service journeys. Note which pyramid layers the scenario set covers.

---

## Task Path — Test Case Generation

**Output goal:** Specific, executable test cases with detailed steps — focused on Integration testing, API contracts, and technical acceptance criteria.

### Task Analyze Phase
**Core rule: Design test cases from technical behavior first. Map to ACs after — not before.**

1. Identify core flows **and enumerate every integration point the task crosses** (service-to-service, API gateway, 3rd-party/payment, event broker, database) — this list drives mandatory negative coverage
2. Identify input fields, state transitions, and validation rules
3. **Implementation Mining** *(optional — only when real code or a PR diff is available)*: Read the implementation to surface conditions the task omits. See "Implementation Mining" section below. Skip when input is only a description.
4. Extract and ID every acceptance criterion — used for traceability mapping only, not to drive case creation
5. Run **Requirement Analysis** (see section above) before moving to Coverage Rules

### Task Coverage Rules
- **Happy Paths**: Verify the task's core technical flows succeed
- **Negative Scenarios**: At least 40% of cases must target validation errors, invalid inputs, and unauthorized actions
- **Boundary Value Analysis (BVA)**: Test exact thresholds, null, empty strings, and caps
- **Atomic Steps**: Every test step has exactly one observable, measurable checkpoint
- **Integration Point Negatives**: Every integration point must have negative cases — see "Mandatory Integration Point Negatives" section below

### Task Output Format

**Before the table:** Present the Requirement Analysis findings (see section above), then list extracted AC IDs and their text.

| Test ID | Title | AC Ref | Preconditions | Test Steps | Expected Result | Priority | Pyramid Layer |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TC-F-001 | [Short descr] | AC-1 | Prerequisites & test data | 1. Step one\n2. Step two | Exact verifiable state | High/Med/Low | Unit / Component (BE) / Component (FE) / Integration / E2E / Exploratory |

**ID prefixes:** `TC-F` (Functional), `TC-E` (Edge), `TC-ERR` (Error Handling)

**After the table:**
- **AC traceability matrix** — one row per AC: `AC-1 | TC-F-001, TC-ERR-004 | covered`  
  Mark each: `covered`, `happy-path only`, or `⚠️ NOT COVERED`. List `— gap` cases as requirements defects for the BA.
- **Integration point coverage map** — list every integration point and which failure modes (timeout / 5xx / broker unreachable / DB drop) each one got a case for. Flag uncovered failure modes as gaps.
- **Pyramid summary** — count per layer (e.g., "Unit: 4, Component (BE): 3, Integration: 4, E2E: 1"). Flag if E2E > ~10% of total (ice-cream-cone risk).

---

## AC Traceability (Applies to Both Paths)

**Mapping is mandatory — coverage is not.** Every test case/scenario must be mapped to an AC if one exists. But cases/scenarios that have no AC behind them are **valid and expected** — QA's value is seeing risks, dependencies, and edge conditions that the story/task author didn't focus on. Never drop a good test case just because there's no AC for it.

**Extracting and IDing ACs:**
- If the story/task already numbers its criteria, reuse those IDs verbatim (`AC-1`, `AC-3.2`)
- If unnumbered prose, assign `AC-1`, `AC-2`, … in the order they appear, then restate them before the output table
- Split compound criteria into `AC-2a`, `AC-2b` when they assert independently-verifiable behaviors
- Never invent an AC to justify a case

**Mapping rules:**
- Every case/scenario's **AC Ref** column must be filled — either an AC ID or one of the markers below. **Hard gate:** a case/scenario whose AC Ref is empty and carries no marker is a Validate Phase failure — do not present output until it's filled in or marked.
- A case/scenario with no governing AC must be marked with **why it still belongs**:
  - `— implicit` — standard quality bar the story never spelled out (BVA on an obvious field, auth check, integration negative)
  - `— code-only` — surfaced by Implementation Mining, absent from the story. Add `⚠️ code-only path, not in story`
  - `— qa-insight` — QA's own analysis: deeper dependency, cross-service risk, or edge case the story/task didn't consider. **This is expected and valuable — do not suppress these cases.**
  - `— needs-qa-review` — an AC exists, but it's written at a higher level (story/E2E, user-facing) than this case's layer naturally reaches (e.g. a Unit/Component case under an E2E-worded AC). The generator can't confidently confirm the link itself — a human QA reviewer must confirm the mapping before sign-off. Do **not** treat this as a gap.
  - `— gap` — clearly should have been an AC but isn't. Flag for BA as a requirements defect
- An AC with **zero** cases is a hard failure of the Validate Phase — write the case or state plainly why it's untestable

**AC Traceability Matrix (after the table):**

Two-direction check:
1. **AC → Cases**: one row per AC — `AC-1 | TC-F-001, TC-ERR-004 | covered`  
   Mark each: `covered`, `happy-path only`, or `⚠️ NOT COVERED`
2. **Cases beyond AC**: list all cases marked `— qa-insight`, `— implicit`, `— code-only`, or `— gap` in a separate section so the reader can see what QA added beyond the written requirements

```
### Beyond-AC Cases
| Test ID | Marker | Reason |
| :--- | :--- | :--- |
| TC-ERR-003 | — qa-insight | Payment service retry not mentioned in AC — race condition risk if network flaps mid-transaction |
| TC-E-007 | — implicit | BVA: qty = 0 boundary not stated but obviously relevant |
```

### Needs QA Review

List every case tagged `— needs-qa-review` in its own table, separate from Beyond-AC Cases — these have a real AC, but the layer-to-AC mapping needs a human to confirm it before sign-off, not QA-added insight to review for value.

```
### Needs QA Review
| Test ID | AC Ref | Why mapping needs confirmation |
| :--- | :--- | :--- |
| TC-F-003 | AC-2 — needs-qa-review | AC-2 describes the full checkout journey (E2E-level); this case only verifies one Component (BE) step of it — confirm the link still holds |
```

---

## Confidence Marker (Applies to Both Paths)

When the exact system behavior behind an Expected Result isn't knowable from the input — no code, no spec, no confirmed contract for a real implementation detail, not a style choice — don't write it as settled fact, and don't bury the uncertainty inside the wording.

- Write the best-guess Expected Result, then append inline: `⚠️ NEEDS CONFIRMATION: <specific thing to verify>` (e.g. exact error message text, status code, an API contract field)
- Applies to both Story and Task paths — uncertainty isn't exclusive to technical detail
- Never silently guess and present the guess as a confirmed expected result

---

## Mandatory Integration Point Negatives (Task Path)

**Non-negotiable.** First enumerate every integration point, then prove each has at least one negative case per applicable failure mode below.

| Failure mode | Simulate | Expected Result must assert | Layer |
| :--- | :--- | :--- | :--- |
| **Connection Timeout** | downstream exceeds read timeout | bounded timeout, defined error returned, retry is idempotent (no duplicate record/charge) | Integration |
| **Downstream 5xx Outage** | downstream returns 500/502/503 repeatedly | error surfaced (not swallowed), circuit breaker opens after threshold, fallback runs | Integration |
| **Event Broker Unreachable** | Kafka broker down / publish rejected | message routed to DLQ with replay context; primary business flow still completes; no silent loss | Integration |
| **Database Connection Drop** | connection killed mid-transaction | transaction rolls back fully; no partial rows; failure reported; retry leaves exactly one consistent record | Integration |

Rules:
- Tag as `TC-ERR-*` and count toward the ≥40% negative mandate
- Name the **specific** integration point in the title (e.g., `[Checkout → Payment]`, `[Kafka: order.paid]`)
- Expected Result must state the **observable recovery behavior** — not "handles error gracefully"
- Preconditions must describe **how** failure is injected (mock, toxiproxy, stopped container)
- If a mechanism doesn't exist yet (no DLQ, no circuit breaker), still generate the case and flag: `⚠️ no DLQ configured — coverage gap, confirm with Dev`

---

## Implementation Mining (Task Path — Optional)

Run only when the input includes actual source code or a PR diff. If not, skip entirely — never fabricate code details.

| Mine | Look for | Becomes | Layer |
| :--- | :--- | :--- | :--- |
| **Branches** | if/else, switch, guard clause, early return | one case per path | Unit / Component |
| **Validations** | min/max, regex, required, type checks | boundary + negative cases | Unit / Component |
| **Error paths** | throw, try/catch, error status codes, fallbacks | TC-ERR cases | Component (BE) / Integration |
| **Boundaries** | numeric thresholds, off-by-one, null/empty | BVA cases | Unit |
| **State / permission** | auth checks, role gates, feature flags | unauthorized / negative cases | Integration |

Rules:
- Reference the function/file (and line if known) in Preconditions so the case is verifiable against the code
- Mark mined cases that the story never mentioned: `⚠️ code-only path, not in story`
- Do not reproduce source code beyond what describes the test

---

## Testing Pyramid Mapping

Reference: **QA_WayOfWork_2025 — Pyramid Paradigm**. Every test artifact must be tagged at the cheapest layer that fully verifies the behavior.

- **Unit**: Single function in isolation. Mocked dependencies. Pure logic, BVA, null/empty checks.
- **Component (BE)**: Single API endpoint against its contract. Real DB, not mocks.
- **Component (FE)**: Single UI widget/form in isolation. Rendering, validation states, no real backend.
- **Integration (Subsystem)**: Real service-to-service interaction, no mocks. Cross-service data flow, contracts, race conditions.
- **E2E**: Complete user journey through the real stack. Reserve for critical business/revenue paths only.
- **Exploratory**: Requires human judgment — visual/UX review, ambiguous copy, ad-hoc edges.

**Pyramid level by input type:**

| Input | Primary layers | Notes |
|-------|---------------|-------|
| User Story | Integration, E2E, Exploratory | Scenarios describe user journeys — naturally higher layers |
| Task | Unit, Component (BE/FE), Integration | Cases describe technical behavior — naturally lower-middle layers |

**Decision heuristic (E2E vs Integration):** *"If swapping the UI for a curl command wouldn't change what the test proves, it belongs at Integration — not E2E."*

**Target shape:** ~70% Unit+Component, Integration next, **E2E ≤ 10%**, Exploratory for human-only cases.

**Integration ceiling:** Integration count must not exceed Unit+Component combined. If it does, the suite needs rebalancing — see Validate Phase.

---

## Validate Phase (Both Paths)

Before presenting output, self-review:
1. Every case/scenario has an AC Ref filled — either an AC ID or a marker (`— implicit / — qa-insight / — code-only / — needs-qa-review / — gap`). **Hard gate: an empty, unmarked AC Ref blocks output — fix it before presenting anything.**
2. No case was created *solely* to fill an AC slot — every case must be motivated by a real behavior or risk
3. Any AC with zero cases is flagged as `⚠️ NOT COVERED` in the traceability matrix — **do not invent a case to fill the slot; flag it instead**
4. Cases marked `— qa-insight` are **kept and highlighted** — they represent QA's deeper analysis, not noise
5. Negative coverage ≥ 30% (Story) or ≥ 40% (Task)
6. Integration points (Task path) each have at least one negative case
7. **Pyramid shape — fix, don't just flag:** E2E ≤ ~10% of total AND Integration ≤ Unit+Component combined. If either is violated, split/rewrite the offending broad case(s) into the correct lower layer(s) before presenting output — re-tagging the same case alone is not a fix.
8. Every Expected Result stating an unverified technical detail carries an inline `⚠️ NEEDS CONFIRMATION` marker — none are presented as confirmed fact without support
9. Every Requirement Analysis finding resolved to a case, a `— gap` / `⚠️ NOT COVERED` flag, or an explicit unresolved note — none silently dropped
10. No duplicate cases
