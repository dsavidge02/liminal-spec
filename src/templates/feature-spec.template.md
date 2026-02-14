## Feature Spec Template

Use the following template when producing a feature spec.

---

### Template Start

```markdown
# Feature: [Feature Name]

This specification defines the complete requirements for [feature name]. It serves as the
source of truth for the Tech Lead's design work.

---

## User Profile

**Primary User:** [Role]
**Context:** [Situation when they use this]
**Mental Model:** [How they conceptualize the task]
**Key Constraint:** [Environmental or technical limitation]

---

## Feature Overview

[What the user will be able to do after this feature ships that they cannot do today.
Plain description of the new capability and why it matters.]

---

## Scope

### In Scope

Brief prose description of what this feature delivers and why:

- Capability 1
- Capability 2
- Capability 3

### Out of Scope

- Excluded capability (see LS-XXX if planned)
- Future enhancement (planned for Phase N)
- Related but separate concern

### Assumptions

| ID | Assumption | Status | Owner | Notes |
|----|------------|--------|-------|-------|
| A1 | Assumption text | Unvalidated | [stakeholder] | Confirm by [date] |

---

## Flows & Requirements

### 1. [Flow/Capability Name]

[Prose description of this flow — what it covers, when it applies, why it matters.]

1. User [entry action]
2. System [response]
3. User [next action]
4. System [feedback]
5. User [completion action]
6. System [final response]

#### Acceptance Criteria

**AC-1.1:** [Testable statement]

- **TC-1.1a: [Descriptive name]**
  - Given: [Precondition]
  - When: [Action]
  - Then: [Expected result]
- **TC-1.1b: [Edge case name]**
  - Given: [Different precondition]
  - When: [Action]
  - Then: [Expected result]

**AC-1.2:** [Testable statement]

- **TC-1.2a: [Descriptive name]**
  - Given: [Precondition]
  - When: [Action]
  - Then: [Expected result]

### 2. [Flow/Capability Name]

[Prose description.]

1. User [entry action]
2. System [response]
3. User [action]
4. System [response]

#### Acceptance Criteria

**AC-2.1:** [Testable statement]

- **TC-2.1a: [Descriptive name]**
  - Given: [Precondition]
  - When: [Action]
  - Then: [Expected result]

### 3. [Error/Cancel Flow]

[How users exit without completing, and what the system preserves or discards.]

1. User [triggers cancel/close]
2. System [cleanup behavior]
3. System [return behavior]

#### Acceptance Criteria

**AC-3.1:** [Testable statement]

- **TC-3.1a: [Descriptive name]**
  - Given: [Precondition]
  - When: [Action]
  - Then: [Expected result]

---

## Data Contracts (if applicable)

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| param1 | string | Yes | Description |

### Response Types

[TypeScript interfaces or equivalent typed shapes]

### Error Responses

| Status | Code | Description |
|--------|------|-------------|
| 400 | ERROR_CODE | Description |

---

## Dependencies

Technical dependencies:
- [API, library, or feature dependency]

Process dependencies:
- [Approval, sign-off, or sequencing dependency]

---

## Non-Functional Requirements (if applicable)

[Cross-cutting constraints: performance, security, observability, accessibility.
These constrain how things are built rather than what is built.
NFRs become Tech Design constraints, not TCs.]

---

## Tech Design Questions

Questions for the Tech Lead to address during design:

1. [Technical question raised during spec writing or validation]

---

## Recommended Story Breakdown

### Story 0: Infrastructure
[Types, error classes, fixtures, test utilities needed by all stories]

### Story 1: [Title — first vertical slice]
**Delivers:** [What the user can do after]
**Prerequisite:** Story 0
**ACs covered:**
- AC-1.1 ([summary])
- AC-1.2 ([summary])
- AC-1.3 ([summary])

### Story 2: [Title — next slice]
**Delivers:** [What the user can do after]
**Prerequisite:** Story 1
**ACs covered:**
- AC-2.1 ([summary])
- AC-2.2 ([summary])

### Story N: [Continue as needed]

---

## Validation Checklist

- [ ] User Profile has all four fields + Feature Overview
- [ ] Flows cover all paths (happy, alternate, cancel/error)
- [ ] Every AC is testable (no vague terms)
- [ ] Every AC has at least one TC
- [ ] TCs cover happy path, edge cases, and errors
- [ ] Data contracts are fully typed (if applicable)
- [ ] Scope boundaries are explicit (in/out/assumptions)
- [ ] Story breakdown covers all ACs
- [ ] Stories sequence logically (read before write, happy before edge)
- [ ] All validator issues addressed (Critical, Major, and Minor)
- [ ] Validation rounds complete (no substantive changes remaining)
- [ ] Self-review complete
```

### Template End
