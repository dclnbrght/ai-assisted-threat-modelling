# Threat Model Report: Demo-FromDescription

> Purpose: Lite report with high-impact STRIDE coverage.

## 1. Executive Summary
- Scope: Mobile App client, STS, API Gateway, App Service, Database, External AI Agent Service across Internet / Cloud Host / Third Party trust boundaries; auth (1–2), main request (3–6,9–10), AI enrichment (7–8) flows.
- Top Risks (≤5): T-001, T-002, T-003, T-004, T-005
- Overall Posture: Elevated (early-stage design; notable transport, injection, AI interaction, access control & supply chain gaps)
- Immediate Actions (≤30d):
  1. Enforce TLS (Gateway→App Service) with mTLS & strict token audience validation (T-001)
  2. Implement centralized input validation + parameterized DB abstraction & output encoding (T-002)

## 2. System Context & Architecture
- Diagram Reference: `projects/Demo-FromDescription/resources/threat-model-dataflow-mermaid.md`
- Architecture Style: Service-oriented (API Gateway + App Service) with external AI augmentation.
- Key Data Flows: Auth (1–2), Main Request (3–6,9–10), AI Enrichment (7–8)

### 2.1 Component & Data Inventory
| ID | Name | Type (Ext / Proc / Store / Flow) | Description | Trust Boundary | Privilege | Data Sensitivity | Notes |
|----|------|----------------------------------|-------------|---------------|----------|------------------|-------|
| C1 | Mobile App | External Entity | End-user client | Internet | User | Medium (session token, user data) | Untrusted input source |
| C2 | STS | Process | Issues auth tokens | Cloud Host | High (token signing) | High (tokens) | Protect keys & time sync |
| C3 | API Gateway | Process | Entry point / routing | Cloud Host | Medium | Medium | Rate limits, initial authN |
| C4 | App Service | Process | Core business & AI orchestration | Cloud Host | Medium-High | High (aggregated data) | Adds AI output |
| C5 | Database | Data Store | Persistent storage | Cloud Host | High | High (PII/business) | Encrypt, least privilege |
| C6 | AI Agent Service | External Entity | Third-party enrichment | Third Party | N/A | Potential High (context) | Treat as untrusted |
| F1 | Auth Flow (1–2) | Flow | Token issuance exchange | Internet→Cloud | Elevated | High (creds/tokens) | Requires TLS, replay defense |
| F2 | Main Request (3–6,9–10) | Flow | CRUD/Query path | Internet→Cloud | Elevated | High (user data) | Validate & encode |
| F3 | AI Enrichment (7–8) | Flow | External AI call | Cloud→Third Party | Medium | High (context leakage) | Prompt/output filtering |

### 2.2 Assumptions & Constraints
| Assumption / Constraint | Risk if Invalid |
|-------------------------|-----------------|
| STS tokens short-lived & signed | Replay / token theft impact worsens |
| API Gateway fully validates tokens & enforces TLS | Spoofing / downgrade risk |
| Internal hop soon upgraded to HTTPS/mTLS | MITM, tampering, credential harvest |
| DB access via parameterized queries only | SQL injection & mass disclosure |
| AI service does not persist sensitive prompts | Long-term data leakage/privacy breach |
| Logs exclude secrets/tokens | Credential leakage & lateral movement |
| Time synchronization maintained | Replay, expiration bypass, repudiation |

### 2.3 Data Classification (CIA Baseline)
| Data Asset | C (L/M/H) | I (L/M/H) | A (L/M/H) | Rationale |
|-----------|-----------|-----------|-----------|-----------|
| Auth Tokens | H | H | M | Grants access; tamper risk high |
| User / Domain Data | H | M | M | Potentially sensitive user/business info |
| Application Logs (sanitized) | M | M | M | Operational, moderate sensitivity |
| AI Prompt Context | H | M | L | May include user data sent externally |
| AI Responses | M | M | L | Augments output; moderate sensitivity |

## 3. Method & Scoring
- STRIDE applied to components & flows; de-duplicated by root cause.
- Likelihood 1–10; Impact 1–10; Risk = L×I (High ≥56, Medium 21–55, Low ≤20).
- High risks get detail section.

## 4. Threat Register
| ID | Component | STRIDE | Description (Concise) | L | I | Risk | Key Mitigations | Status |
|----|-----------|--------|-----------------------|---|---|------|-----------------|--------|
| T-001 | API Gateway ↔ App Service (F2) | S/T/E | Attacker could intercept or spoof internal traffic lacking TLS/mTLS causing tampering & privilege misuse | 7 | 9 | 63 | Enforce TLS+mTLS; audience & issuer validation; network policy | Open |
| T-002 | App Service / Database (F2) | T/I/E | Injection via insufficient validation enabling SQL/data exfiltration | 7 | 8 | 56 | Central validation; parameterized queries; least privilege; WAF rules | Open |
| T-003 | App Service / AI Service (F3) | I/T/E | Prompt/response injection leaks sensitive data or induces malicious output | 6 | 8 | 48 | Prompt sanitization; output guardrails; egress allowlist; content filters | Open |
| T-004 | STS / Auth Flow (F1) | S/E/R | Token replay if TTL long or clock skew; limited audit for misuse | 5 | 9 | 45 | Short TTL; nonce/audience binding; replay detection; clock sync | Open |
| T-005 | Supply Chain (All) | T/E | Dependency or model update compromise alters logic | 5 | 9 | 45 | SBOM; signed artifacts; dependency & model provenance scanning | Open |
| T-006 | Access Control (Object Level) | E/I | Missing object-level authZ allows data access (BOLA) | 6 | 7 | 42 | Object checks; ABAC policy; enumeration tests | Open |
| T-007 | Availability (Gateway/App) | D | Insufficient rate/resource limits allows DoS | 5 | 7 | 35 | Adaptive rate limiting; circuit breakers; autoscale; caching | Open |
| T-008 | Logging / Monitoring | R | Incomplete audit logging hinders incident attribution | 4 | 7 | 28 | Central structured logs; integrity & coverage reviews | Open |

## 5. High Risk Details
### T-001 Insecure Internal Transport & Lack of Mutual Authentication
- Vector: Internal plaintext or one-way TLS allows on-path entity (compromised pod, misconfigured proxy) to sniff/modify calls or replay tokens.
- Impact: Credential theft, request manipulation, lateral movement, potential mass data access via forged calls.
- Existing Controls: External TLS termination; signed JWT tokens (assumed); basic network segmentation.
- Mitigations (prioritized):
  1. Enforce TLS 1.2+ (or 1.3) on Gateway→App Service with mTLS (workload identities)
  2. Validate token audience, issuer, expiry & nonce in App Service
  3. Apply network policy restricting Gateway egress to App Service only
  4. Continuous security test scanning for plaintext endpoints in CI
  5. Rotate & monitor service certificates (short-lived)
- Residual Target: <30

### T-002 Input/Query Injection Risk
- Vector: Crafted input bypasses weak validation enabling SQL injection or ORM abuse; potential stacked queries or data exfiltration.
- Impact: Bulk disclosure/modification of high confidentiality data; integrity compromise of business records.
- Existing Controls: None confirmed beyond assumed parameterization (not yet enforced globally).
- Mitigations (prioritized):
  1. Central schema & type validation layer (reject on fail-fast)
  2. Mandatory parameterized queries / stored procedures
  3. Least-privilege DB role per service function
  4. WAF / RASP detection for anomaly patterns
  5. Security tests (fuzz/IAST) in CI/CD
- Residual Target: <30

## 6. Mitigation Roadmap
| Threat ID | Action | Owner | Target Date | Ticket # |
|-----------|--------|-------|-------------|----------|
| T-001 | Implement mTLS & internal TLS enforcement | Platform Eng | 2025-10-15 | TBA |
| T-002 | Deploy centralized validation & parameterization layer | App Dev | 2025-10-10 | TBA |
| T-003 | Add AI prompt/output guardrails & egress allowlist | App Dev | 2025-10-12 | TBA |
| T-004 | Configure short TTL, nonce & replay detection in STS | Identity | 2025-10-08 | TBA |
| T-005 | Establish SBOM & signed artifact verification pipeline | DevSecOps | 2025-10-20 | TBA |
| T-006 | Implement object-level authZ & enumeration tests | App Dev | 2025-10-18 | TBA |
| T-007 | Introduce rate limiting & circuit breaker policies | Platform Eng | 2025-10-22 | TBA |
| T-008 | Expand structured audit logging & integrity controls | Observability | 2025-10-25 | TBA |

## 7. Accepted / Deferred Risks
| ID | Justification | Review Date | Compensating Controls |
|----|---------------|------------|-----------------------|
| (none) |  |  |  |

## 8. Change Log / Review History
| Date | Author/Reviewer | Section Updated | Summary of Changes |
|------|-----------------|-----------------|-------------------|
| 2025-09-28 | AI Assistant | All | Draft lite threat model |

---
