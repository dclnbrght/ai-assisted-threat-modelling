# Threat Model Report: [Project Name]

> Purpose: Standard template, with broad STRIDE, OWASP, NIST coverage.

## 1. Executive Summary
- Scope Summary: [Systems/components]
- Top 5 Risks: [T-001, T-002, ...]
- Overall Posture: [Narrative]
- Key Themes: [e.g., AuthZ, Supply Chain]
- Immediate Actions (≤30d):
  1. [Action]
  2. [Action]

## 2. System Context
### 2.1 Architecture & Data Flow
- Diagram Reference: [File/link/Embed-Mermaid-Markdown]
- Architecture Style: [Monolith / Microservices / Event-driven / AI Integration]

### 2.2 Component & Data Inventory
| ID | Name | Type (Ext / Proc / Store / Flow) | Description | Trust Boundary | Privilege | Data Sensitivity | Interfaces | Notes |
|----|------|----------------------------------|-------------|---------------|----------|------------------|------------|-------|
| C1 |  |  |  |  |  |  |  |  |

### 2.3 Trust Boundaries
| Boundary ID | Description | Components | Crossing Mechanisms | Notes |
|-------------|-------------|------------|---------------------|-------|
| TB1 |  |  |  |  |

### 2.4 Assumptions & Constraints
| Category | Assumption / Constraint | Risk if Invalid |
|----------|-------------------------|-----------------|
|  |  |  |

### 2.5 Data Classification & CIA Baseline
| Data Asset | C (L/M/H) | I (L/M/H) | A (L/M/H) | Rationale |
|-----------|-----------|-----------|-----------|-----------|
| DA1 |  |  |  |  |

## 3. Method & Scoring
- STRIDE applied to components, data stores, trust boundary crossings.
- OWASP 2021 + API 2023 mapped where relevant; new lens only adds labels (no duplicate threats).
- Likelihood (1–10), Impact (1–10), Risk = L×I (High ≥56, Medium 21–55, Low ≤20).
- High risks get detailed expansion; Medium/Low summarized only.

## 4. Threat Register
| ID | Component | STRIDE | OWASP Cat | MITRE ATT&CK TTP(s) | Description | L | I | Risk | Key Mitigations | Status | Residual L | Residual I | Residual Risk |
|----|-----------|--------|-----------|---------------------|-------------|---|---|------|-----------------------|--------|------------|------------|--------------|
| T-001 |  |  |  |  | A [actor] could ... |  |  |  |  | Open |  |  |  |  |

## 5. High Risk Details
### [T-00X] [Title]
- STRIDE: [ ]
- OWASP: [ ]
- MITRE ATT&CK Mapping: [Tactic/Technique ID(s) and names]
- Description: [Expanded impact]
- Attack Vectors: [List]
- Existing Controls: [List]
- Gaps: [List]
- Mitigations (Prioritised):
  1. [Primary]
  2. [Secondary]
- Validation: [Test / Review]
- Residual Target: [e.g., <30]

## 6. OWASP Coverage Summary
| OWASP / API Category | Threat IDs | Notes |
|----------------------|-----------|-------|
| A01 Broken Access Control |  |  |
| A02 Cryptographic Failures |  |  |
| A03 Injection |  |  |
| A04 Insecure Design |  |  |
| A05 Security Misconfiguration |  |  |
| A06 Vulnerable & Outdated Components |  |  |
| A07 Identification & Authentication Failures |  |  |
| A08 Software & Data Integrity Failures |  |  |
| A09 Security Logging & Monitoring Failures |  |  |
| A10 SSRF |  |  |
| API1 BOLA |  |  |
| API2 Broken Authentication |  |  |
| API3 BOPLA |  |  |
| API4 Resource Consumption |  |  |
| API5 Function Level Auth |  |  |
| API6 Sensitive Business Flows |  |  |
| API7 SSRF |  |  |
| API8 Security Misconfiguration |  |  |
| API9 Inventory Management |  |  |
| API10 Unsafe Consumption |  |  |
| LLM01 Prompt Injection |  |  |
| LLM02 Insecure Output Handling |  |  |
| LLM03 Training Data Poisoning |  |  |
| LLM04 Model Denial of Service |  |  |
| LLM05 Supply Chain Vulnerabilities |  |  |
| LLM06 Sensitive Information Disclosure |  |  |
| LLM07 Insecure Plugin Design |  |  |
| LLM08 Excessive Agency |  |  |
| LLM09 Overreliance |  |  |
| LLM10 Model Theft |  |  |

## 7. Detection & Monitoring (Focused)
| Event Type | Logged? (Y/N) | Alerted? | Gap / Action |
|------------|---------------|----------|-------------|
| Auth Failures |  |  |  |
| Privilege Change |  |  |  |
| Data Export |  |  |  |
| Rate Limit Breach |  |  |  |
| Anomalous Enumeration |  |  |  |
| Critical Error |  |  |  |

## 8. Mitigation Roadmap
> Guidance: Link each mitigation to a work item in your chosen tracking system (e.g., Jira, Azure DevOps, GitHub Issues). Use a stable ticket reference (e.g., `SEC-1234`). If multiple tickets support one action, reference the primary ticket and enumerate supporting tasks in the Action cell. Do not mark a threat mitigated here, status should live in the ticketing system; this table is a linkage index.

| Threat ID | Action | Owner | Type | Target Date | Ticket # | Dependencies |
|-----------|--------|-------|----------|-------------|----------|------------|
| T-001 |  |  |  |  | TBA |  |

Legend: Type = Prevent / Detect / Respond / Recover

## 9. Residual Risk Snapshot
| Threat ID | Original Risk | Residual Target | Current Residual | Notes |
|-----------|---------------|-----------------|------------------|-------|
| T-001 |  |  |  |  |

## 10. Accepted / Deferred Risks
| ID | Justification | Review Date | Compensating Controls |
|----|---------------|------------|-----------------------|
| T-00X |  |  |  |

## 11. Change Log / Review History
| Date | Author/Reviewer | Section Updated | Summary of Changes |
|------|----------|----------------|-------------------|
|  |  |  |  |

## 12. Appendix (Optional)
### 12.1 Data Flow Descriptions
[List each data flow: Source → Destination, Data Type, Protocol, Security Controls]

### 12.2 Glossary
| Term | Definition |
|------|------------|
| BOLA | Broken Object Level Authorization |
| BOPLA | Broken Object Property Level Authorization |
| Residual Risk | Post-mitigation Likelihood × Impact |

---