# Threat Model Report: [Project Name]

> Purpose: Extended report with comprehensive STRIDE, OWASP, NIST coverage for governance, detection, and residual risk tracking.

## 1. Executive Summary
- Scope Summary: [In-scope systems/components]
- Key High Risks (Top 5): [IDs]
- Overall Risk Posture: [Narrative]
- Notable Themes: [e.g., Authorization gaps / Supply chain / Monitoring]
- Recommended Immediate Actions (≤30 days):
  1. [Action]
  2. [Action]

## 2. System Context
### 2.1 Architecture & Data Flow
- Diagram Reference: [File/link/Embed-Mermaid-Markdown]
- Architecture Style: [Monolith / Microservices / Event-driven / AI Integration]

### 2.2 Component & Data Asset Inventory
| ID | Name | Type (Ext Entity / Process / Data Store / Flow) | Description | Privilege Level | Data Classification | Trust Boundary | Interfaces | Notes |
|----|------|-----------------------------------------------|-------------|-----------------|--------------------|----------------|-----------|-------|
| C1 |  |  |  |  |  |  |  |  |

### 2.3 Trust Boundaries
| Boundary ID | Description | Enclosed Components | Crossing Mechanisms | Notes |
|-------------|-------------|---------------------|--------------------|-------|
| TB1 |  |  |  |  |

### 2.4 Assumptions & Constraints
| Category | Assumption / Constraint | Risk if Invalid |
|----------|-------------------------|-----------------|
| Hosting |  |  |

### 2.5 Data Classification & CIA Baseline (FIPS 199)
| Data Asset | Confidentiality (L/M/H) | Integrity (L/M/H) | Availability (L/M/H) | Rationale |
|-----------|-------------------------|------------------|----------------------|-----------|
| DA1 |  |  |  |  |

## 3. Methodology & Lenses Applied
| Lens | Scope Applied | Notes |
|------|---------------|-------|
| STRIDE | All components & flows |  |
| OWASP Web 2021 | [Yes/No/Partial] |  |
| OWASP API 2023 |  |  |
| OWASP Mobile |  |  |
| OWASP LLM Apps |  |  |
| LLM / AI |  |  |
| NIST 800-53 |  |  |
| NIST SSDF 800-218 |  |  |
| Zero Trust (800-207) |  |  |
| AI RMF |  |  |
| Supply Chain |  |  |
| Detection & Monitoring |  |  |

## 4. Threat Register
| ID | Component | STRIDE | OWASP Cat | NIST Families | CIA Impacted | Description | L (1-10) | I (1-10) | Risk | Ctrl Maturity | Detection | Key Mitigations | Owner | Status | Residual L | Residual I | Residual Risk |
|----|-----------|--------|-----------|---------------|--------------|-------------|----------|----------|------|---------------|-----------|----------------------|-------|--------|------------|------------|--------------|
| T-001 |  |  |  |  |  | A [actor] could ... |  |  |  | None | No |  |  | Open |  |  |  |

Legend: Ctrl Maturity = None / Partial / Adequate / Strong; Status = Open / Planned / In Progress / Mitigated / Accepted.

## 5. High Risk Detail
### [T-001] [Title]
- STRIDE Type(s): [ ]
- OWASP Category: [ ]
- NIST Controls: [AC, IA, AU, ...]
- AI RMF (if applicable): [Govern/Map/Measure/Manage]
- Description: [Expanded impact & context]
- Attack Vectors: [List]
- Affected Assets / Data: [ ]
- Existing Controls: [List]
- Gaps / Weaknesses: [ ]
- Recommended Mitigations:
  1. [Primary]
  2. [Secondary]
- Control Category Mapping: [Prevent / Detect / Respond / Recover]
- Owner: [ ]
- Target Timeline: [Date]
- Success Metrics / KPIs: [e.g., reduction in unauthorized calls]
- Validation Method: [Pen test / Red team / Automated test / Code review]
- Residual Target: [e.g., <30]

(Repeat for each High Risk.)

## 6. Risk Distribution
| Risk Band | Count | % | Notes |
|-----------|-------|---|-------|
| High (≥56) |  |  |  |
| Medium (21-55) |  |  |  |
| Low (≤20) |  |  |  |

## 7. Top Risks by Component
| Component | High | Medium | Low | Commentary |
|-----------|------|--------|-----|------------|
| C1 |  |  |  |  |

## 8. OWASP Coverage Summary
| OWASP Category | Threats Logged? (Y/N) | Gaps / Notes |
|----------------|-----------------------|--------------|
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
| M1 Platform Usage |  |  |
| M2 Insecure Data Storage |  |  |
| M3 Insecure Communication |  |  |
| M4 Insecure Authentication |  |  |
| M5 Insufficient Cryptography |  |  |
| M6 Insecure Authorization |  |  |
| M7 Client Code Quality |  |  |
| M8 Code Tampering |  |  |
| M9 Reverse Engineering |  |  |
| M10 Extraneous Functionality |  |  |
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

## 9. NIST Alignment Matrices
### 9.1 Control Family Mapping (Aggregate)
| NIST Family | Threat IDs | Coverage Notes |
|-------------|-----------|----------------|
| AC |  |  |
| IA |  |  |
| AU |  |  |
| SC |  |  |
| SI |  |  |
| CM |  |  |
| IR |  |  |
| CP |  |  |
| CA |  |  |
| SA |  |  |
| SR |  |  |
| PM |  |  |
| RA |  |  |

### 9.2 SSDF Practice Mapping
| SSDF Group | Related Threat IDs | Maturity (None/Low/Med/High) | Notes |
|------------|--------------------|------------------------------|-------|
| PO |  |  |  |
| PS |  |  |  |
| PW |  |  |  |
| RV |  |  |  |

### 9.3 Zero Trust & Identity
| Domain | Gap / Observation | Related Threats | Action Needed |
|--------|-------------------|-----------------|---------------|
| Continuous Verification |  |  |  |
| Least Privilege / JIT |  |  |  |
| Micro-Segmentation |  |  |  |
| Workload Identity |  |  |  |
| Session Resilience |  |  |  |

### 9.4 AI RMF Mapping (If Applicable)
| AI RMF Function | Threat IDs | Metrics / Controls | Notes |
|-----------------|-----------|--------------------|-------|
| Govern |  |  |  |
| Map |  |  |  |
| Measure |  |  |  |
| Manage |  |  |  |

## 10. Supply Chain & Dependency Integrity
| Area | Current State | Gaps | Related Threats | Action |
|------|---------------|------|-----------------|--------|
| SBOM Generation |  |  |  |  |
| Vulnerability Scanning |  |  |  |  |
| Artifact Signing |  |  |  |  |
| Build Pipeline Hardening |  |  |  |  |
| Model/Dataset Provenance |  |  |  |  |
| Third-Party Services |  |  |  |  |
| License / Policy Gates |  |  |  |  |

## 11. Detection & Monitoring Coverage
| Event Type | Logged? (Y/N) | Integrity Protected? | Alerted? | MTTD Target | MTTR Target | Gaps / Notes | Threat IDs |
|------------|---------------|----------------------|----------|-------------|-------------|--------------|------------|
| Auth Success/Failure |  |  |  |  |  |  |  |
| Privilege Changes |  |  |  |  |  |  |  |
| Data Export / Exfil |  |  |  |  |  |  |  |
| Model Override / Config Change |  |  |  |  |  |  |  |
| Rate Limit Breaches |  |  |  |  |  |  |  |
| Anomaly (Enumeration) |  |  |  |  |  |  |  |
| Prompt Injection Flag |  |  |  |  |  |  |  |
| Critical Service Failure |  |  |  |  |  |  |  |

## 12. Mitigation Roadmap
> Guidance: Link each mitigation to a work item in your chosen tracking system (e.g., Jira, Azure DevOps, GitHub Issues). Use a stable ticket reference (e.g., `SEC-1234`). If multiple tickets support one action, reference the primary ticket and enumerate supporting tasks in the Action cell. Do not mark a threat mitigated here, status should live in the ticketing system; this table is a linkage index.

| Threat ID | Mitigation Action | Type | Owner | Start | Target Date | Ticket # | SSDF Group | Dependencies |
|-----------|-------------------|----------|-------|-------|--------|----------|------------|--------------|
| T-001 |  |  |  |  |  | TBA |  |  |

Legend: Type = Prevent / Detect / Respond / Recover

## 13. Residual Risk Register
| Threat ID | Original Risk | Residual Risk | Delta | Rationale for Acceptance / Further Action |
|-----------|---------------|---------------|-------|-------------------------------------------|
| T-001 |  |  |  |  |

## 14. Accepted / Deferred Risks
| ID | Description | Risk Level | Justification | Review Date | Compensating Controls |
|----|-------------|-----------|--------------|-------------|-----------------------|
| EX-01 |  |  |  |  |  |

## 15. Change Log / Review History
| Date | Author/Reviewer | Section Updated | Summary of Changes |
|------|----------|----------------|-------------------|
|  |  |  |  |

## 16. Appendix (Optional)
### 16.1 Data Flow Descriptions
[List each data flow: Source → Destination, Data Type, Protocol, Security Controls]

### 16.2 Glossary
| Term | Definition |
|------|------------|
| BOLA | Broken Object Level Authorization |
| BOPLA | Broken Object Property Level Authorization |
| Residual Risk | Post-mitigation Likelihood × Impact |

---