---
name: ticket-estimator
description: Estimate story points for software development tickets on a 1/2/3/5/8 scale with a weighted, explainable breakdown. Use whenever a user pastes a Jira or Linear ticket (URL or text) and asks for an estimate, story points, sizing, pointing, sprint planning, backlog grooming or refinement. Also trigger on casual phrasing like "how big is this ticket?" or "can we fit this in a sprint?".
---

# Ticket Estimator

Estimate development tickets on a **1 / 2 / 3 / 5 / 8** point scale (Fibonacci-style, 2-week sprint) and explain the score so the team can challenge it.

## Point Scale

| Points | Rough effort | Typical profile |
|--------|--------------|-----------------|
| **1** | < 1 day | Script, config or feature-flag change. Clear path, trivial testing. |
| **2** | 1–2 days | Easy change, easy to test. Single component, no unknowns. |
| **3** | 3–4 days | Easy change that is hard to test, **or** medium change that is easy to test. |
| **5** | ~1 week | A 3-pointer plus investigation, risk to other products, or dependencies on other teams. Or a complex change that needs design first. |
| **8** | Full sprint | Largest size that fits a sprint: big scope, high risk, many unknowns. Always recommend splitting or swarming. |

## Workflow

### Step 1: Get the ticket

- If the user gives a URL, fetch it (or use a connected Jira/Linear tool if available).
- If the page needs a login or returns nothing useful, ask the user to paste the ticket text.

Extract: title, description, acceptance criteria, scope of work, testing requirements, labels/components, linked tickets.

### Step 2: Score four dimensions

Score each dimension independently on the 1/2/3/5/8 scale.

| Dimension | Weight | Ask yourself |
|-----------|--------|--------------|
| **Scope of code changes** | 40% | How many files, services or systems change? New code or edits to existing code? DB or infra changes? |
| **Uncertainty / unknowns** | 25% | Is the approach clear? Open questions, missing specs, investigation or design needed? |
| **Testing / QA effort** | 20% | Unit, component, E2E, manual QA, regression across other products or customers? |
| **Cross-team dependencies** | 15% | Waiting on or blocking other teams? API contracts, shared services, third parties, migrations? |

### Step 3: Compute the weighted score

```
weighted = scope*0.40 + uncertainty*0.25 + testing*0.20 + dependencies*0.15
```

| Weighted score | Points |
|----------------|--------|
| ≤ 1.5 | **1** |
| 1.5 – 2.5 | **2** |
| 2.5 – 4.0 | **3** |
| 4.0 – 6.5 | **5** |
| > 6.5 | **8** |

Then apply the **adjustment rules** below. If a rule changes the result, say so in the output.

### Adjustment rules

1. **Easy change + hard testing = at least 3.** Testing effort counts as real work.
2. **3-pointer + cross-product risk or another team's dependency = 5.**
3. **Investigation or design needed before coding = at least 5.**
4. **Anything at 8 → recommend a split.** Suggest concrete sub-tickets.
5. **Torn between two values → pick the higher one.** Teams underestimate more often than they overestimate.

### Step 4: Output

Use this format:

---

**Ticket:** [title]

## Estimate: [X] points

| Dimension | Score | Rationale |
|-----------|-------|-----------|
| Scope of code changes | X | [1–2 sentences] |
| Uncertainty / unknowns | X | [1–2 sentences] |
| Testing / QA effort | X | [1–2 sentences] |
| Cross-team dependencies | X | [1–2 sentences] |

**Weighted score:** [value] → **[points]** [+ adjustment rule applied, if any]

**Closest calibration example:** [which example below it resembles and why]

### Key risks
- [missing acceptance criteria, hidden complexity, unclear ownership, etc.]

### Suggestions
- [how to reduce the estimate: split, clarify, timebox a spike, parallelize]

---

## Edge cases

- **No acceptance criteria:** flag it, raise uncertainty, and suggest adding criteria before committing.
- **Epic or meta-task:** don't score it as one ticket. Propose sub-tickets and score each.
- **Bug:** same rubric. Bugs range from 1 (typo) to 8 (system-level issue).
- **Spike / research:** score uncertainty high, recommend a timebox, and note that the output is a plan, not shipped code.
- **Several tickets at once:** score each one, then end with a summary table.

## Calibration examples

These examples are made up to show the patterns. Teams should replace them with their own past tickets and the points they agreed on.

### 1 point: scripts and config changes

**1a. Enable new payment method for two more countries**
Add Chile and Peru to the `enabledCountries` list for an existing payment method. Follow the existing activation runbook. Smoke-test checkout in both countries.
→ Config only, proven runbook, trivial testing.

**1b. Turn on the new pricing-table feature flag in production**
The flag was validated in staging. Check prices match the old table, then turn on the flag after the next release.
→ Flag flip plus a quick data check. No new code.

### 2 points: easy change, easy to test

**2a. Design the data contract for an order-history widget**
Define the JSON structure the UI will use (orders, status, totals) and document it with an example payload.
→ Design and documentation only. Approach is clear.

**2b. Add hover states to available/unavailable time slots**
Pointer cursor for available slots, "not-allowed" cursor for unavailable ones. Add a Storybook story.
→ Small frontend change, visual testing only.

### 3 points: easy change that is hard to test, or medium change that is easy to test

**3a. Filter recommendations by product attributes from tenant config**
Read a new attribute mapping from tenant config and apply it in the recommendation service so excluded items stop appearing. Compare API responses before and after, and run regression checks on other tenants.
→ Moderate code change. Before/after and regression testing adds real effort.

**3b. Move hardcoded plan types out of a utility class into config**
Move constants into tenant config, rename variables to match new plan names, and update the transformer to read from config.
→ Refactor across a few files. Easy to test.

### 5 points: 3-pointer plus risk, dependencies or design work

**5a. Show only the relevant offer on the confirmation page for some tenants**
Add aggregation logic in the backend-for-frontend to filter offers by channel flags, with a per-tenant config switch. Three acceptance scenarios, plus regression checks so tenants that don't use the switch are unaffected.
→ Medium change with risk to other customers and a large regression surface.

**5b. Transform a new layout element across all supplier integrations**
Each supplier API returns the element differently. Update every transformer, document the mapping, and add tests. Approach is based on another engineer's investigation.
→ Same change repeated across many integrations, plus remaining unknowns.

### 8 points: biggest that fits a sprint (recommend splitting)

**8a. Hourly snapshot service to S3**
New endpoint that reads a new config table, finds the entities that qualify, fetches and compresses each snapshot, and uploads it to S3 using a folder and file-naming convention. Includes the DB table, component tests and documentation.
→ New endpoint, new table, new infrastructure and a scheduled flow. Split into: config table, snapshot logic, S3 upload, tests and docs.

**8b. Add a "void" fallback to the payment rollback client**
Detect when a void is needed instead of a refund (inside the sale window, or after a refund fails), expose a new timestamp field through the data access layer, sign in to the correct merchant account before voiding, and test end to end in staging with the only provider that allows it.
→ Payment risk, conditional logic, a dependency on one external provider and heavy E2E testing.
