# Threat Model Report: Demo-FromMermaid

> Purpose: Standard template, fresh independent STRIDE + OWASP/API + NIST + AI risk analysis (Mermaid-derived).

## 1. Executive Summary
- Scope Summary: Mobile App client, Security Token Service (STS), API Gateway, App Service (business logic + AI outbound integration), relational Database, external Third-Party AI Agent Service across Internet / Cloud / Third-Party trust boundaries.
- Top 5 Risks: T-001, T-002, T-003, T-004, T-005
- Overall Posture: Moderate; perimeter TLS & coarse auth present, but gaps in fine-grained authorization, supply chain integrity automation, AI output safeguarding, structured detection, and token lifecycle hardening reduce resilience.
- Key Themes: Authorization depth (BOLA/BOPLA), Injection & Unsafe Consumption, AI Output / Prompt Integrity, Supply Chain Integrity, Logging & Detection Coverage.
- Immediate Actions (≤30d):
  1. Centralize object/function-level authorization & explicit deny-by-default (T-001).
  2. Enforce universal parameterized data access + schema validation (T-002).
  3. Deploy AI response validation & outbound data minimization + guardrails (T-003).
  4. Shorten token TTL, implement revocation & replay detection (T-004).
  5. Expand structured security logging + anomaly & injection pattern alerts (T-005).

## 2. System Context
### 2.1 Architecture & Data Flow
- Diagram Reference: `projects/Demo-FromMermaid/resources/threat-model-dataflow-mermaid.md`
- Architecture Style: Service + Gateway with external AI augmentation.

Data Flows (from Mermaid numbering):
1. Client → STS: Auth Request (HTTPS)
2. STS → Client: Auth Token (JWT)
3. Client → API Gateway: API Request (HTTPS)
4. API Gateway → App Service: Internal Request (HTTP)
5. App Service → Database: SQL Query
6. Database → App Service: Result Set
7. App Service → AI Agent Service: Outbound AI Request (HTTPS)
8. AI Agent Service → App Service: Response (Markdown/Text)
9. App Service → API Gateway: Response (JSON)
10. API Gateway → Client: Response (JSON)

### 2.2 Component & Data Inventory
| ID | Name | Type (Ext / Proc / Store / Flow) | Description | Trust Boundary | Privilege | Data Sensitivity | Interfaces | Notes |
|----|------|----------------------------------|-------------|---------------|----------|------------------|------------|-------|
| C1 | Mobile App Client | Ext | End-user mobile application (presentation) | Internet | Least | User profile, session tokens (local secure storage) | HTTPS to STS & Gateway | Reverse engineering risk |
| C2 | STS | Proc | AuthN service issuing JWT/OIDC tokens | Cloud | High | Credentials, signing keys, identity claims | HTTPS from client | Key protection critical |
| C3 | API Gateway | Proc | Edge ingress, routing, coarse authZ, rate limiting | Cloud | Medium | Requests metadata, limited PII | HTTPS from client; HTTP to App | Apply WAF rules |
| C4 | App Service | Proc | Core business logic & AI integration | Cloud | High | PII, business records, AI prompts/responses | HTTP from Gateway; HTTPS outbound | Primary authorization & validation layer |
| C5 | Database | Store | Relational data store | Cloud (segmented) | High | PII, business data | SQL (secured) | Needs least-priv & encryption |
| C6 | AI Agent Service | Ext | Third-party AI processing | Third-Party | External | Prompt fragments, derived text | HTTPS | Treat output untrusted |
| F1 | Auth Flow | Flow | Credential submission & token issuance | Internet→Cloud | N/A | Credentials & tokens | TLS 1.2+ | Resist brute force / replay |
| F2 | Data Ops Flow | Flow | CRUD via Gateway→App→DB | Internet→Cloud internal | N/A | PII/business | HTTPS/HTTP/SQL | Enforce object-level auth |
| F3 | AI Flow | Flow | Outbound prompt & inbound response | Cloud→Third-Party | N/A | Context data | HTTPS | Sanitize outbound / inbound |

### 2.3 Trust Boundaries
| Boundary ID | Description | Components | Crossing Mechanisms | Notes |
|-------------|-------------|------------|---------------------|-------|
| TB1 | Internet to Cloud ingress | C1 ↔ C2,C3 | HTTPS (1,2,3,10) | Public exposure; edge hardening |
| TB2 | Internal Cloud segmentation | C2,C3,C4,C5 | HTTP, SQL (4,5,6,9) | Consider mTLS & network policy |
| TB3 | Cloud to Third-Party AI | C4 ↔ C6 | HTTPS (7,8) | Untrusted AI output path |

### 2.4 Assumptions & Constraints
| Category | Assumption / Constraint | Risk if Invalid |
|----------|-------------------------|-----------------|
| TLS | Endpoints enforce TLS 1.2+ (no weak ciphers) | MitM or downgrade disclosure |
| Keys | STS signing keys stored in HSM/secure vault rotated ≤90d | Token forgery / long-lived compromise |
| AuthZ | App Service responsible for object/function authorization (Gateway coarse only) | Widespread BOLA/BOPLA gaps |
| Least Priv | DB credentials limited (no DDL) | Lateral movement & privilege escalation |
| Logging | Sensitive data excluded / masked in logs | PII or token leakage |
| Third-Party | AI Agent contract, no storage of secrets, SLA for incident notice | Uncontrolled data retention / exfil |
| Secret Mgmt | Central secret manager; no hard-coded secrets | Credential exposure in code/repos |
| Time Sync | Systems NTP-synchronized | Replay & repudiation issues |

### 2.5 Data Classification & CIA Baseline
| Data Asset | C (L/M/H) | I (L/M/H) | A (L/M/H) | Rationale |
|-----------|-----------|-----------|-----------|-----------|
| Auth Tokens | H | H | M | Enables impersonation & privilege |
| User PII | H | M | M | Regulatory & privacy impact |
| Business Records | M | H | M | Integrity drives business decisions |
| AI Prompts/Responses | M | M | L | Some contextual sensitivity; moderate integrity |
| Secrets / Config | H | H | M | Keys/creds pivot risk |
| Security Logs | M | H | M | Forensics & repudiation defense |

## 3. Method & Scoring
- STRIDE applied to components, flows, boundaries. OWASP Web 2021, API 2023, and LLM categories considered; only unique root causes listed.
- Likelihood (L) & Impact (I) scored 1–10; Risk = L×I (High ≥56, Medium 21–55, Low ≤20).
- High risks get expanded detail (≥56). Fresh independent scoring; not reusing prior report.

## 4. Threat Register
| ID | Component | STRIDE | OWASP Cat | Description | L | I | Risk | Key Mitigations | Status | Residual L | Residual I | Residual Risk |
|----|-----------|--------|-----------|-------------|---|---|------|-----------------------|--------|------------|------------|--------------|
| T-001 | App Service (C4) | E,I | A01,API1,API5 | A malicious user could access or manipulate unauthorized records via missing object/function-level authZ checks (BOLA/BOPLA) leading to data disclosure & privilege escalation (NIST AC, IA, CM). | 8 | 8 | 64 | Central policy engine, deny-by-default, ownership checks, authZ tests & decision logs. | Open | 3 | 5 | 15 |
| T-002 | App Service ↔ DB (C4,C5) | T,I,E | A03,API10 | An attacker could exploit unparameterized queries or weak validation to perform SQL injection altering or exfiltrating data (NIST SC, SI, CM). | 7 | 9 | 63 | Parameterized queries/ORM, schema validation, least-priv DB role, SAST/DAST gates, WAF signatures. | Open | 3 | 5 | 15 |
| T-003 | App Service ↔ AI Agent (C4,C6) | I,T | API10,LLM02,LLM06 | Untrusted AI response/prompt injection could leak sensitive context or manipulate logic via crafted Markdown/text (NIST SR, SC, SI; AI RMF Measure/Manage). | 7 | 9 | 63 | Output schema & pattern validation, PII redaction, outbound minimization, guardrails, anomaly monitoring. | Open | 4 | 5 | 20 |
| T-004 | STS / Token Lifecycle (C2) | S,E | A07,API2 | Stolen/replayed long-lived tokens without rotation/binding could enable broad impersonation (NIST IA, AC). | 6 | 8 | 48 | Short TTL, refresh rotation, binding (aud/nonce), revoke/jti tracking, mTLS internal. | Open | 3 | 4 | 12 |
| T-005 | Logging & Monitoring (Multi) | R | A09 | Insufficient structured security event logging (auth anomalies, enumeration, AI abuse) impairs detection & non-repudiation (NIST AU, IR). | 6 | 7 | 42 | Expand structured logs, WORM integrity, anomaly & enumeration detection, time sync validation. | Open | 3 | 4 | 12 |
| T-006 | Dependency / Supply Chain (C4) | T,E | A06,A08 | Vulnerable or tampered dependency/container could introduce RCE or bypass controls (NIST SA, SR, CM). | 5 | 8 | 40 | SBOM, signature verification, pinned versions, daily scans, ephemeral CI runners. | Open | 3 | 5 | 15 |
| T-007 | Rate Limiting / DoS (C3) | D | A04,API4 | Excessive or adaptive request bursts exploit inadequate rate & quota controls, causing resource exhaustion (NIST SC, CP). | 5 | 7 | 35 | Adaptive per-identity quotas, circuit breakers, autoscale guardrails, backpressure. | Open | 3 | 5 | 15 |
| T-008 | Sensitive Data in Logs (C4,C5) | I | A02 | PII/tokens inadvertently logged enable lateral movement & privacy breach (NIST SC, MP). | 4 | 8 | 32 | Log scrubbing, secrets scanning, masking policies, pre-prod log reviews. | Open | 2 | 4 | 8 |
| T-009 | Economic AI Abuse (C4,C6) | D | API4,LLM04 | Automated abusive prompts inflate AI usage & cost, degrading availability/budget (NIST SI, IR). | 5 | 6 | 30 | Usage quotas, cost anomaly dashboards, approval for bulk jobs, token rate alerts. | Open | 2 | 4 | 8 |
| T-010 | Configuration Drift (Infra) | T,E | A05,API8 | Misconfiguration (headers, CORS, TLS, auth settings) introduces exploitable weaknesses (NIST CM, SC). | 5 | 6 | 30 | IaC policy checks, drift detection, hardened baselines, config CI tests. | Open | 3 | 4 | 12 |
| T-011 | Insufficient Log Integrity (Pipeline) | R,T | A09 | Lack of log integrity (signing/WORM) allows tampering & repudiation (NIST AU, IR, SI). | 4 | 7 | 28 | Log signing, immutable storage, integrity verification jobs, chain-of-custody docs. | Open | 2 | 4 | 8 |
| T-012 | AI Prompt Data Overexposure (C4) | I | LLM06 | Excess contextual data sent outbound could leak sensitive fields if third-party mishandles or model echoes (NIST AC, SC; AI RMF Manage). | 5 | 6 | 30 | Prompt minimization, data classification filter, redaction rules, contract review, token usage diffs. | Open | 3 | 4 | 12 |
| T-013 | Replay / Ordering Gaps (Flows) | S,R | A07 | Missing nonce/jti + clock skew could allow limited replay or challenge tampering affecting audit trails (NIST IA, AU). | 4 | 6 | 24 | Nonce & jti enforcement, clock sync alarms, replay cache, signed request timestamps. | Open | 2 | 4 | 8 |
| T-014 | Model Output Hallucination Impact (C4,C6) | I | LLM02,LLM09 | Hallucinated AI output improperly trusted could mislead business decisions causing integrity erosion (NIST SI; AI RMF Measure). | 4 | 6 | 24 | Human review for critical actions, confidence scoring, output disclaimer & validation rules. | Open | 2 | 4 | 8 |

## 5. High Risk Details (≥56)
### T-001 Insufficient Object & Function-Level Authorization
- STRIDE: Elevation of Privilege, Information Disclosure
- OWASP/API: A01 Broken Access Control, API1 BOLA, API5 Function Level Auth
- Description: Absence/inconsistency of per-object and function-level authorization in the App Service enables enumeration and unauthorized access to records & privileged endpoints, risking confidentiality & integrity of high-value assets (PII, business records, tokens context).
- Attack Vectors: ID tampering (numeric or UUID); mass assignment; calling hidden/privileged endpoints; manipulating filters for overbroad dataset return; chaining with token replay.
- Existing Controls: JWT validation at Gateway; coarse role claims.
- Gaps: No central policy engine; lack of ownership checks; limited negative test coverage.
- Mitigations (Prioritized):
  1. Central ABAC/RBAC policy middleware (deny-by-default) with ownership/resource scoping.
  2. Route registration requiring explicit policy mapping (failing closed).
  3. Comprehensive unit/integration authZ tests (positive/negative & fuzz enumeration).
  4. Authorization decision logging & anomaly detection.
- Validation: Automated test suite; manual abuse tests; log anomaly review.
- Residual Target: <30

### T-002 SQL Injection via Unsafe Query Construction
- STRIDE: Tampering, Information Disclosure, Elevation of Privilege
- OWASP/API: A03 Injection, API10 Unsafe Consumption
- Description: Dynamic SQL or improperly sanitized inputs allow structural manipulation of queries exposing or altering data; high integrity & confidentiality impact with potential lateral escalation.
- Attack Vectors: Crafted parameters (ID, filter), encoded payloads, timing inference, stacked queries if driver permits, error enumeration.
- Existing Controls: Partial input validation; network segmentation.
- Gaps: Non-uniform parameterization, insufficient schema validation, limited SAST rules.
- Mitigations (Prioritized):
  1. Enforce parameterized/ORM repository pattern (ban raw concatenation in CI).
  2. Strict schema validation & canonicalization layer pre-query.
  3. Least-privileged DB account (no DDL; row-level security if supported).
  4. SAST/DAST injection gates & query linting.
  5. WAF anomaly & error spike monitoring.
- Validation: Automated injection tests; review DB logs for anomalous patterns.
- Residual Target: <30

### T-003 Untrusted AI Response / Prompt Injection
- STRIDE: Information Disclosure, Tampering
- OWASP/API/LLM: API10 Unsafe Consumption, LLM02 Insecure Output Handling, LLM06 Sensitive Info Disclosure
- Description: AI responses (Markdown/text) ingested without robust validation allow malicious or manipulated content to leak sensitive context or influence logic/UI, potentially cascading into decision errors or data disclosure.
- Attack Vectors: Prompt injection altering system instructions; crafted responses embedding sensitive echoes; oversized payload causing parser edge cases; indirect prompt from user-sourced content.
- Existing Controls: TLS; contractual commitments.
- Gaps: Lack of schema/size validation; no redaction; minimal anomaly monitoring; absent guardrails.
- Mitigations (Prioritized):
  1. Response validation pipeline (schema, max size, prohibited pattern & secret scan).
  2. Outbound prompt minimization & context segmentation.
  3. Guardrail content filters & hallucination/confidence scoring; quarantine suspicious.
  4. Metadata logging (prompt hash, token counts) for anomaly detection.
  5. Third-party SLA & security attestation tracking.
- Validation: Red-team prompt tests; automated guardrail CI; monitoring dashboards.
- Residual Target: <35

## 6. OWASP & LLM Coverage Summary
| OWASP / API / LLM Category | Threat IDs | Notes |
|----------------------------|-----------|-------|
| A01 Broken Access Control | T-001 | AuthZ depth gap |
| A02 Cryptographic Failures | (N/A) | Current controls assumed adequate; verify key rotation |
| A03 Injection | T-002 | SQL injection risk |
| A04 Insecure Design | (Implicit) | Address via T-001,T-002 architecture controls |
| A05 Security Misconfiguration | T-010 | Drift & insecure defaults |
| A06 Vulnerable & Outdated Components | T-006 | Dependency governance |
| A07 Identification & Authentication Failures | T-004,T-013 | Token lifecycle & replay controls |
| A08 Software & Data Integrity Failures | T-006 | Supply chain tampering |
| A09 Security Logging & Monitoring Failures | T-005,T-011 | Coverage & integrity gaps |
| A10 SSRF | (N/A) | No user-supplied outbound fetch endpoints |
| API1 BOLA | T-001 | Object-level authZ |
| API2 Broken Authentication | T-004 | Token misuse |
| API3 BOPLA | (Covered) | Included in T-001 scope |
| API4 Resource Consumption | T-007,T-009 | DoS & economic abuse |
| API5 Function Level Auth | T-001 | Function auth gaps |
| API6 Sensitive Business Flows | (N/A) | Not distinct beyond T-001 currently |
| API7 SSRF | (N/A) | No dynamic URL fetch from user input |
| API8 Security Misconfiguration | T-010 | Config drift |
| API9 Inventory Management | (N/A) | Limited surface presently |
| API10 Unsafe Consumption | T-002,T-003 | SQL + AI output |
| LLM01 Prompt Injection | (Covered in T-003) | Avoid duplicate entry |
| LLM02 Insecure Output Handling | T-003 | Output validation gap |
| LLM03 Training Data Poisoning | (N/A) | No internal model training in scope |
| LLM04 Model Denial of Service | T-009 | Cost/DoS via prompt spam |
| LLM05 Supply Chain Vulnerabilities | T-006 | Dependency/model provenance |
| LLM06 Sensitive Information Disclosure | T-003,T-012 | Context leakage |
| LLM07 Insecure Plugin Design | (N/A) | No plugin system in scope |
| LLM08 Excessive Agency | (N/A) | No autonomous action layer |
| LLM09 Overreliance | T-014 | Hallucination trusted improperly |
| LLM10 Model Theft | (N/A) | Third-party hosted, no proprietary weights |

## 7. Detection & Monitoring (Focused)
| Event Type | Logged? (Y/N) | Alerted? | Gap / Action |
|------------|---------------|----------|-------------|
| Auth Failures | Partial | Partial | Normalize structured events & threshold alerts |
| Privilege Change | No | No | Emit detailed role/permission change events |
| Data Export | Partial | No | Add volume anomaly & export tagging |
| Rate Limit Breach | Partial | Partial | Adaptive per-identity rate modeling |
| Anomalous Enumeration | No | No | Pattern detection (sequential IDs, high 404/403) |
| Critical Error | Partial | No | Classify severity + alert on security exceptions |
| AI Prompt Injection Indicators | No | No | Guardrail log events + heuristic detection |
| Token Replay Suspicion | No | No | jti/nonce duplication & geo/time anomaly rules |
| Supply Chain Scan Failure | Partial | No | Alert on critical unpatched > SLA |
| Log Integrity Violation | No | No | Integrity verification & alerting |

## 8. Mitigation Roadmap
| Threat ID | Action | Owner | Type | Target Date | Ticket # | Dependencies |
|-----------|--------|-------|------|-------------|----------|------------|
| T-001 | Implement central policy engine & authZ test suite | App Team Lead | Prevent | 2025-10-31 | SEC-1101 | None |
| T-002 | Enforce ORM + schema validation + SAST gate | Platform Eng | Prevent | 2025-10-31 | SEC-1102 | SEC-1105 |
| T-003 | Deploy AI response validation & guardrails | AI Integrations | Prevent/Detect | 2025-11-15 | SEC-1103 | None |
| T-004 | Token TTL reduction + revocation service & replay cache | Identity | Prevent | 2025-11-01 | SEC-1104 | SEC-1101 |
| T-005 | Expand structured security logging & anomaly detection | SecOps | Detect | 2025-10-25 | SEC-1105 | None |
| T-006 | SBOM generation + signature verification in CI | DevSecOps | Prevent | 2025-10-25 | SEC-1106 | None |
| T-007 | Adaptive rate limiting & circuit breaker rules | Platform Eng | Prevent | 2025-11-10 | SEC-1107 | SEC-1105 |
| T-008 | Log scrubbing & secrets detection pipeline | SecOps | Prevent | 2025-10-28 | SEC-1108 | SEC-1105 |
| T-009 | Cost anomaly dashboards & quota enforcement | FinOps | Detect | 2025-11-05 | SEC-1109 | SEC-1103 |
| T-010 | Config drift detection & policy-as-code checks | DevSecOps | Prevent | 2025-11-12 | SEC-1110 | None |
| T-011 | Log signing & integrity verification job | SecOps | Prevent/Detect | 2025-11-08 | SEC-1111 | SEC-1105 |
| T-012 | Prompt minimization & classification filter | AI Integrations | Prevent | 2025-11-05 | SEC-1112 | SEC-1103 |
| T-013 | Nonce/jti replay detection & clock sync alarms | Identity | Detect | 2025-11-07 | SEC-1113 | SEC-1104 |
| T-014 | Human review workflow & confidence scoring | AI Integrations | Detect | 2025-11-12 | SEC-1114 | SEC-1103 |

Legend: Type = Prevent / Detect / Respond / Recover

## 9. Residual Risk Snapshot (High Only)
| Threat ID | Original Risk | Residual Target | Current Residual | Notes |
|-----------|---------------|-----------------|------------------|-------|
| T-001 | 64 | <30 | 15 | Pending policy engine deployment |
| T-002 | 63 | <30 | 15 | ORM & validation rollout in progress |
| T-003 | 63 | <35 | 20 | Guardrail framework underway |

## 10. Accepted / Deferred Risks
| ID | Justification | Review Date | Compensating Controls |
|----|---------------|------------|-----------------------|
| (None) |  |  |  |

## 11. Change Log / Review History
| Date | Author/Reviewer | Section Updated | Summary of Changes |
|------|-----------------|----------------|-------------------|
| 2025-09-28 | Automated Assistant | All | Draft STRIDE/OWASP/LLM/NIST assessment |

## 12. Appendix (Optional)
### 12.1 Data Flow Descriptions
See 2.1; additional granularity not required for current risk decisions.

### 12.2 Glossary (Selected)
- BOLA: Broken Object Level Authorization
- BOPLA: Broken Object Property Level Authorization
- Guardrails: Policy & safety enforcement layer for AI outputs
- Residual Risk: Post‑mitigation Risk (Residual L × Residual I)

---
