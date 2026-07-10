---
inclusion: manual
---

# High-Coverage Manual & Functional Test Case Generation

## Purpose
Transform feature descriptions, Jira user stories, or raw code into clear, atomic, executable, and reviewable manual test cases. Prevent code logic gaps and enforce edge-case auditing.

## Operational Workflow
1. **Analyze Phase**: Parse input requirements, technical constraints, and user personas. Identify core flows.
2. **Generate Phase**: Build non-duplicated test specifications using the structural template below.
3. **Validate Phase**: Self-review for missing boundaries, ambiguous steps, or orphan requirements before presenting output.

## Rules and Coverage Mandates
- **Happy Paths**: Verify core functional user journeys.
- **Negative Scenarios**: At least 40% of generated cases must intentionally target validation errors, invalid inputs, and unauthorized actions.
- **Boundary Value Analysis (BVA)**: Test exact mathematical thresholds, null values, empty strings, and character caps.
- **Atomic Steps**: Every test action must have exactly one direct, observable, and measurable checkpoint.

## Testing Pyramid Mapping
Reference: internal **QA_WayOfWork_2025 — Pyramid Paradigm**. Every test case must be tagged with the layer of the 5-layer pyramid where it is best (cheapest, fastest) verified. Bottom = many/cheap tests, top = few/expensive tests.

- **Unit** *(Dev owns code+test)*: A single function/method/class in isolation. Mocked dependencies. Pure logic, calculations, BVA on isolated values, null/empty/threshold checks with no cross-service dependency. Runs in milliseconds.
- **Component (BE)** *(Dev owns code+test+assert)*: A single service/module/API endpoint against its contract. Real DB connections, not mocks. E.g. `POST /checkout` with valid/invalid payload → status + response shape.
- **Component (FE)** *(Dev owns code+test+assert)*: A single UI widget/page/form in isolation (Storybook/RTL-style). Rendering, validation states, disabled/enabled states, inline error display — without a real backend or full navigation.
- **Integration (Subsystem)** *(QA owns design+test+data)*: Real service-to-service interaction, no mocks — data flow, contracts, and consistency across two or more services (e.g., Checkout service → Payment service → DB; Gateway routing/auth). This is where most "business logic across systems" scenarios (discount rules, tax rules, race conditions, out-of-stock handling) belong.
- **E2E** *(QA owns design+test+data)*: A complete user journey through the real stack as the customer experiences it (Home → Cart → Checkout, guest checkout, cross-app flows like Opal config → storefront behavior). Slow, flaky-prone, expensive — reserve for critical business/revenue paths only.
- **Exploratory** *(QA owns, manual only — no dev involvement)*: Requires human judgment — visual/UX review, ambiguous copy, ad-hoc edge cases impractical or not worth automating.

**Decision heuristic (E2E vs. Integration):** *"If swapping the UI for a curl command wouldn't change what the test proves, it belongs at Integration/Subsystem — not E2E."* Only promote to E2E when UI/session/gateway behavior itself matters, it's a critical revenue/trust journey, or Integration would miss it (e.g., a real production incident could span that boundary).

**Target shape (avoid the ice-cream cone and the hourglass):** ~70% Unit, Component close behind, Integration next, **E2E capped at ~10%**, Exploratory covers only what genuinely needs a human. When a scenario could fit more than one layer, tag it at the **lowest/cheapest** layer that still fully verifies the behavior, then note in Expected Result if it should also get a thin E2E/Exploratory check for the customer-facing symptom.

## Required Output Structure
Output test structures matching this format strictly:

| Test ID | Title | Preconditions | Test Steps | Expected Result | Priority | Pyramid Layer |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TC-F-001 | [Feature] Short Descr | Prerequisites & test data | 1. Step one\n2. Step two | Exact verifiable state | High/Med/Low | Unit / Component (BE) / Component (FE) / Integration / E2E / Exploratory |

- **Formatting IDs**: Use prefix `TC-F` (Functional), `TC-E` (Edge), `TC-ERR` (Error Handling).
- **Pyramid summary**: After the table, include a one-line count of cases per layer (e.g., "Unit: 4, Component (BE): 3, Component (FE): 2, Integration: 4, E2E: 1, Exploratory: 1") and flag if E2E exceeds ~10% of total cases or if Unit/Component is under-represented (ice-cream-cone risk).
