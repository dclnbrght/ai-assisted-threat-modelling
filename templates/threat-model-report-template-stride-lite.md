# Threat Model Report: [Project Name]

> Purpose: Lite report with high-impact STRIDE coverage.

## 1. Executive Summary
- Scope: [Brief system/components]
- Top Risks (≤5): [T-001, T-002, ...]
- Overall Posture: [Low / Moderate / Elevated]
- Immediate Actions (≤30d):
  1. [Action]
  2. [Action]

## 2. System Context & Architecture
- Diagram Reference: [File/link/Embed-Mermaid-Markdown]
- Architecture Style: [Monolith / Microservices / Event-driven / AI Integration]
- Key Data Flows: [List or reference]

### 2.1 Component & Data Inventory
| ID | Name | Type (Ext / Proc / Store / Flow) | Description | Trust Boundary | Privilege | Data Sensitivity | Notes |
|----|------|----------------------------------|-------------|---------------|----------|------------------|-------|
| C1 |  |  |  |  |  |  |  |

### 2.2 Assumptions & Constraints
| Assumption / Constraint | Risk if Invalid |
|-------------------------|-----------------|
|  |  |

### 2.3 Data Classification (CIA Baseline)
| Data Asset | C (L/M/H) | I (L/M/H) | A (L/M/H) | Rationale |
|-----------|-----------|-----------|-----------|-------|
| DA1 |  |  |  |  |

## 3. Method & Scoring
- STRIDE applied to components & flows; de-duplicated by root cause.
- Likelihood 1–10; Impact 1–10; Risk = L×I (High ≥56, Medium 21–55, Low ≤20).
- Only High risks get detail section.

## 4. Threat Register
| ID | Component | STRIDE | Description (Concise) | L | I | Risk | Key Mitigations | Status |
|----|-----------|--------|-----------------------|---|---|------|-----------------|--------|
| T-001 |  |  | A [actor] could ... |  |  |  |  | Open |

## 5. High Risk Details
### [T-00X] [Title]
- Vector: [Main mechanism]
- Impact: [What happens]
- Existing Controls: [If any]
- Mitigations (prioritized):
  1. [Primary]
  2. [Secondary]
- Residual Target: [e.g., <30]

## 6. Mitigation Roadmap
> Guidance: Link each mitigation to a work item in your chosen tracking system (e.g., Jira, Azure DevOps, GitHub Issues). Use a stable ticket reference (e.g., `SEC-1234`). If multiple tickets support one action, reference the primary ticket and enumerate supporting tasks in the Action cell. Do not mark a threat mitigated here, status should live in the ticketing system; this table is a linkage index.

| Threat ID | Action | Owner | Target Date | Ticket # |
|-----------|--------|-------|-------------|----------|
| T-001 |  |  |  | TBA |

## 7. Accepted / Deferred Risks
| ID | Justification | Review Date | Compensating Controls |
|----|---------------|------------|-----------------------|
| T-00X |  |  |  |

## 8. Change Log / Review History
| Date | Author/Reviewer | Section Updated | Summary of Changes |
|------|----------|----------------|-------------------|
|  |  |  |  |

---