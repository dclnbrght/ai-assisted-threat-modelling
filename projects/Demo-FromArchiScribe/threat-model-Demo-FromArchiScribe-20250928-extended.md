# Threat Model Report: Demo-FromArchiScribe

> Purpose: Extended report with comprehensive STRIDE, OWASP, NIST coverage for governance, detection, and residual risk tracking.

## 1. Executive Summary
- Scope Summary: Mobile App, STS (Secure Token Service), API Gateway, App Service, Database, AI Agent Service; data flows 1-10 per architecture; trust boundaries: Internet, Cloud Host, 3rd Party Host.
- Key High Risks (Top 5): T-001, T-002, T-003, T-004, T-005
- Overall Risk Posture: Elevated – multiple high-risk vectors around authorization enforcement, AI output handling, supply chain trust, and logging gaps. Core authentication is centralized (STS) but downstream authorization and AI content validation are under-specified.
- Notable Themes: Authorization depth gaps; AI service trust & output sanitization; Insufficient detection for abuse (enumeration / prompt injection); Supply chain (3rd party AI) reliance; Data protection of tokens and PII.
- Recommended Immediate Actions (≤30 days):
  1. Implement unified authorization enforcement & least-privilege review at API Gateway and App Service layers (addresses T-001/T-002).
  2. Add AI output sanitization and content filtering plus rate limiting on AI Agent calls (addresses T-003/T-005).

## 2. System Context
### 2.1 Architecture & Data Flow
- Data Flow Reference: `resources/threat-model-dataflow-archiscribe.md`
- Architecture Style: Microservices-style components behind a gateway with external AI integration.

### 2.2 Component & Data Asset Inventory
| ID | Name | Type (Ext Entity / Process / Data Store / Flow) | Description | Privilege Level | Data Classification | Trust Boundary | Interfaces | Notes |
|----|------|-----------------------------------------------|-------------|-----------------|--------------------|----------------|-----------|-------|
| C1 | Mobile App | External Entity | User client application | Low | PII (tokens transient) | Internet | HTTPS to STS & Gateway | Untrusted input source |
| C2 | STS | Process | Secure Token Service issuing auth tokens | High | Credentials / Tokens (High) | Cloud Host | HTTPS | Must harden keys & issuance |
| C3 | API Gateway | Process | Entry point routing & basic security | Medium | PII passthrough | Cloud Host | HTTPS outward / HTTP inward | Should enforce authN/Z consistently |
| C4 | App Service | Process | Core business logic & orchestration | High | PII & AI prompts/responses | Cloud Host | HTTP / SQL / HTTPS (AI) | Contains most logic |
| C5 | Database | Data Store | Persistence for application data | High | PII / Transactional | Cloud Host | SQL | Encryption & backup needed |
| C6 | AI Agent Service | External Process | LLM-based AI provider | Medium | Prompt & response (may contain sensitive data) | 3rd Party Host | HTTPS | Third-party trust & data minimization |
| F1 | Flow 1 | Flow | Auth Request (Mobile -> STS) | n/a | Credentials | Internet→Cloud Host | HTTPS | Rate limit, protect brute force |
| F2 | Flow 2 | Flow | Auth Token (STS -> Mobile) | n/a | Tokens (High) | Cloud Host→Internet | HTTPS | Token leakage risk |
| F3 | Flow 3 | Flow | API Request (Mobile -> Gateway) | n/a | PII / business data | Internet→Cloud Host | HTTPS | Input validation |
| F4 | Flow 4 | Flow | Request (Gateway -> App Service) | n/a | Auth context & payload | Cloud internal | HTTP | Consider mTLS |
| F5 | Flow 5 | Flow | SQL Query (App -> DB) | n/a | Data at rest | Internal | SQL | Injection risk if ORM misused |
| F6 | Flow 6 | Flow | Result Set (DB -> App) | n/a | PII | Internal | SQL | Need encryption in transit (TLS) |
| F7 | Flow 7 | Flow | AI Request (App -> AI Agent) | n/a | Prompt (may have sensitive) | Cloud→3rd Party | HTTPS | Data minimization needed |
| F8 | Flow 8 | Flow | AI Response (AI -> App) | n/a | Generated content | 3rd Party→Cloud | HTTPS | Output validation required |
| F9 | Flow 9 | Flow | Response (App -> Gateway) | n/a | JSON payload | Internal | HTTP | Consistent auth propagation |
| F10| Flow 10| Flow | Response (Gateway -> Mobile) | n/a | JSON response | Cloud Host→Internet | HTTPS | Caching & header security |

### 2.3 Trust Boundaries
| Boundary ID | Description | Enclosed Components | Crossing Mechanisms | Notes |
|-------------|-------------|---------------------|--------------------|-------|
| TB1 | Internet (untrusted clients) | C1 | HTTPS (F1,F3,F2,F10) | Primary ingress risk surface |
| TB2 | Cloud Host (controlled infra) | C2,C3,C4,C5 | HTTPS/HTTP/SQL internal flows (F2-F6,F9) | Internal segmentation & network policies needed |
| TB3 | 3rd Party Host (AI provider) | C6 | HTTPS (F7,F8) | External dependency, supply chain considerations |

### 2.4 Assumptions & Constraints
| Category | Assumption / Constraint | Risk if Invalid |
|----------|-------------------------|-----------------|
| Hosting | STS keys stored in HSM/KMS | Token forgery if compromised |
| Network | Internal traffic not TLS today but private network | MITM if lateral movement achieved |
| AI | AI provider contract enforces data retention limits | Sensitive data retained or leaked |
| Auth | Tokens are JWT signed with strong alg (RS256) | Token replay / forgery |
| Logging | Central log store tamper-resistant | Undetected attacks |

### 2.5 Data Classification & CIA Baseline (FIPS 199)
| Data Asset | Confidentiality (L/M/H) | Integrity (L/M/H) | Availability (L/M/H) | Rationale |
|-----------|-------------------------|------------------|----------------------|-----------|
| User Credentials & Tokens | H | H | M | Direct access = account takeover |
| PII Records | H | H | M | Privacy & compliance impact |
| AI Prompts/Responses | M | M | L | Some may contain sensitive but ephemeral |
| System Logs | M | H | M | Needed for forensics & detection |
| Configuration & Secrets | H | H | H | System compromise risk |

## 3. Methodology & Lenses Applied
| Lens | Scope Applied | Notes |
|------|---------------|-------|
| STRIDE | All components & flows | Core categorization |
| OWASP Web 2021 | Yes | Gateway & App exposure |
| OWASP API 2023 | Yes | API Gateway & App Service |
| OWASP Mobile | Partial | Mobile client baseline only |
| OWASP LLM Apps | Yes | AI Agent integration |
| LLM / AI | Yes | Prompt / output handling |
| NIST 800-53 | Core families | AC, IA, AU, SC, SI focus |
| NIST SSDF 800-218 | Partial | PW, RV initial mapping |
| Zero Trust (800-207) | Partial | Lateral movement controls pending |
| AI RMF | Partial | Govern/Map phases only |
| Supply Chain | Yes | AI provider & libs |
| Detection & Monitoring | In progress | Logging maturity low |

## 4. Threat Register
| ID | Component | STRIDE | OWASP Cat | NIST Families | CIA Impacted | Description | L (1-10) | I (1-10) | Risk | Ctrl Maturity | Detection | Key Mitigations | Owner | Status | Residual L | Residual I | Residual Risk |
|----|-----------|--------|-----------|---------------|--------------|-------------|----------|----------|------|---------------|-----------|----------------------|-------|--------|------------|------------|--------------|
| T-001 | C3,C4 | Elevation of Privilege / Tampering | A01, API5 | AC, IA, AU | C,I | Inconsistent authorization between Gateway and App allows bypass of function-level checks. | 7 | 8 | 56 | Partial | No | Centralize authZ + policy as code; enforce scopes | SecEng | Open | 4 | 6 | 24 |
| T-002 | C5 | Tampering / Repudiation | A05, A09 | AC, SC, AU, SI | C,I | Lack of encryption & integrity for internal DB traffic enables data modification if internal compromise occurs. | 6 | 9 | 54 | None | No | TLS internal, DB IAM, integrity monitoring | DBA | Open | 3 | 6 | 18 |
| T-003 | C6,C4 | Information Disclosure | LLM06, A02 | AC, SC, SR | C | AI prompts may include sensitive PII sent to 3rd party without minimization or masking. | 6 | 7 | 42 | None | Partial | Data minimization, classification-based redaction | AppSec | Open | 3 | 5 | 15 |
| T-004 | C2 | Spoofing / Repudiation | A07, API2 | AC, IA, AU | C,I | Weak monitoring of STS token issuance could hide brute force or token abuse. | 5 | 8 | 40 | Partial | No | Add anomaly detection & issuance rate limits | SecEng | Open | 3 | 5 | 15 |
| T-005 | C4 | Injection / Tampering | A03, API4 | SC, SI | C,I,A | AI response not validated could enable injection into downstream processing or resource exhaustion. | 5 | 8 | 40 | None | No | Output sanitization, schema validation, rate limits | DevLead | Open | 3 | 5 | 15 |
| T-006 | C3 | Denial of Service | A04, API4 | SC, CP | A | Insufficient rate limiting & resource quotas at Gateway permit volumetric abuse. | 6 | 6 | 36 | Partial | Partial | Adaptive rate limiting, WAF rules | SecOps | Open | 3 | 4 | 12 |
| T-007 | C5 | Repudiation / Integrity | A09 | AU, SI, AC | I | Insufficient audit logging for DB write operations hinders forensics. | 5 | 7 | 35 | None | No | Structured immutable logging, write journaling | DBA | Open | 3 | 4 | 12 |
| T-008 | C6 | Supply Chain / Tampering | LLM05, A06 | SR, SA, CM | C,I | AI provider compromise or model poisoning alters outputs. | 4 | 8 | 32 | None | No | Provider due diligence, attestation, anomaly scoring | Vendor | Open | 3 | 5 | 15 |
| T-009 | C1 | Spoofing | M4, A07 | IA, AC | C | Weak mobile app hardening may allow credential interception / MITM if cert pinning absent. | 4 | 7 | 28 | None | Partial | Cert pinning, secure storage, hardening | Mobile | Open | 2 | 5 | 10 |
| T-010 | C4 | Information Disclosure | A01, A05 | AC, SC | C | Excessive data returned due to coarse response filters. | 5 | 5 | 25 | Partial | Partial | Response filtering, field-level masking | DevLead | Open | 2 | 4 | 8 |

Legend: Ctrl Maturity = None / Partial / Adequate / Strong; Status = Open / Planned / In Progress / Mitigated / Accepted.

## 5. High Risk Detail
### [T-001] Authorization Inconsistency Enables Privilege Escalation
- STRIDE Type(s): Elevation of Privilege, Tampering
- OWASP Category: A01 Broken Access Control, API5 Function Level Auth
- NIST Controls: AC-2, AC-3, AC-6, IA-2, AU-12
- Description: Disparate or missing enforcement layers between Gateway and App Service could allow direct invocation of privileged functions lacking proper scope checks.
- Attack Vectors: Crafted requests bypassing assumed filtering; direct internal path calls; replay of valid tokens with elevated claims if not validated.
- Affected Assets / Data: PII Records, Configuration & Secrets.
- Existing Controls: Basic JWT validation at Gateway; ad hoc checks in App Service.
- Gaps / Weaknesses: No centralized policy engine; lack of consistent scope claim validation.
- Recommended Mitigations:
  1. Deploy policy-as-code (OPA or equivalent) and enforce at both layers.
  2. Implement fine-grained scopes and deny by default for undefined routes.
- Control Category Mapping: Prevent / Detect (logs) / Respond.
- Owner: SecEng
- Target Timeline: 2025-10-31
- Success Metrics / KPIs: 0 unauthorized privileged function calls in SIEM baseline; 100% routes mapped to policy.
- Validation Method: Targeted pentest & automated authZ tests.
- Residual Target: <25 risk score.

### [T-002] Unencrypted Internal DB Traffic & Integrity Gaps
- STRIDE Type(s): Tampering, Repudiation
- OWASP Category: A05 Security Misconfiguration, A09 Logging & Monitoring Failures
- NIST Controls: SC-8, SC-12, AC-6, AU-2, SI-7
- Description: Plain internal SQL connections (if TLS disabled) allow interception or modification by an attacker with lateral foothold.
- Attack Vectors: Sniffing credentials; altering queries via MITM; injecting rogue responses.
- Affected Assets / Data: PII Records, System Logs.
- Existing Controls: Network segmentation only.
- Gaps / Weaknesses: No TLS, weak auditing of write ops.
- Recommended Mitigations:
  1. Enable TLS for DB connections and enforce cert validation.
  2. Implement immutable audit logs for all write operations.
- Control Category Mapping: Prevent / Detect / Recover.
- Owner: DBA
- Target Timeline: 2025-10-15
- Success Metrics / KPIs: 100% encrypted DB sessions; audit log coverage >95% writes.
- Validation Method: Config scan & log sampling.
- Residual Target: <20 risk score.

### [T-003] Sensitive Data Leakage via AI Prompting
- STRIDE Type(s): Information Disclosure
- OWASP Category: A02 Cryptographic Failures, LLM06 Sensitive Information Disclosure
- NIST Controls: AC-6, SC-13, SR-3, SR-5
- Description: Application may transmit unredacted PII or secrets in prompts to external AI service.
- Attack Vectors: User-provided data embedded in prompts; system logs capturing prompts; model provider breach.
- Affected Assets / Data: PII Records, Configuration & Secrets.
- Existing Controls: None specific.
- Gaps / Weaknesses: No redaction; no classification gating.
- Recommended Mitigations:
  1. Implement automated prompt redaction based on data classification.
  2. Restrict outbound prompt fields to allow list.
- Control Category Mapping: Prevent / Detect.
- Owner: AppSec
- Target Timeline: 2025-10-31
- Success Metrics / KPIs: <1% prompts containing sensitive fields (sampled); blocked events logged.
- Validation Method: Static/dynamic prompt inspection.
- Residual Target: <20 risk score.

### [T-005] AI Output Injection & Resource Exhaustion
- STRIDE Type(s): Tampering, Denial of Service
- OWASP Category: A03 Injection, API4 Resource Consumption, LLM04 Model DoS
- NIST Controls: SI-10, SC-5, AC-6
- Description: Unvalidated AI responses could include malicious markup or oversized content leading to injection or service strain.
- Attack Vectors: Prompt injection causing model to output code or large payload; adversarial user prompts.
- Affected Assets / Data: App Service logic, PII Records.
- Existing Controls: None.
- Gaps / Weaknesses: No schema enforcement; no content filters.
- Recommended Mitigations:
  1. Enforce strict response schema & size caps.
  2. Apply content safety filters and anomaly detection for prompt injection patterns.
- Control Category Mapping: Prevent / Detect.
- Owner: DevLead
- Target Timeline: 2025-11-15
- Success Metrics / KPIs: 0 successful injection; 95% detection of anomalous size spikes.
- Validation Method: Fuzz & adversarial prompt tests.
- Residual Target: <20 risk score.

### [T-006] API Gateway DoS via Rate Abuse
- STRIDE Type(s): Denial of Service
- OWASP Category: A04 Insecure Design, API4 Resource Consumption
- NIST Controls: SC-5, CP-10
- Description: Absence of adaptive rate limiting allows attackers to exhaust resources.
- Attack Vectors: Scripted high-rate requests; distributed clients.
- Affected Assets / Data: Availability of API & downstream services.
- Existing Controls: Static basic limits.
- Gaps / Weaknesses: No anomaly-based throttling.
- Recommended Mitigations:
  1. Implement dynamic rate limiting per token/IP.
  2. Add circuit breakers and graceful degradation.
- Control Category Mapping: Prevent / Respond.
- Owner: SecOps
- Target Timeline: 2025-10-31
- Success Metrics / KPIs: Maintain 99.5% uptime under stress test.
- Validation Method: Load & chaos testing.
- Residual Target: <15 risk score.

## 6. Risk Distribution
| Risk Band | Count | % | Notes |
|-----------|-------|---|-------|
| High (≥56) | 1 | 10% | One critical authZ gap |
| Medium (21-55) | 7 | 70% | Majority require layered controls |
| Low (≤20) | 2 | 20% | Post-mitigation targets achievable |

## 7. Top Risks by Component
| Component | High | Medium | Low | Commentary |
|-----------|------|--------|-----|------------|
| C3 API Gateway | 1 | 1 | 0 | Central choke point, auth & DoS focus |
| C4 App Service | 0 | 3 | 0 | AI integration & validation gaps |
| C5 Database | 0 | 2 | 0 | Transport security & logging |
| C6 AI Agent | 0 | 2 | 0 | External supply chain & data disclosure |
| C2 STS | 0 | 1 | 0 | Token issuance monitoring |
| C1 Mobile App | 0 | 1 | 0 | Client hardening |

## 8. OWASP Coverage Summary
| OWASP Category | Threats Logged? (Y/N) | Gaps / Notes |
|----------------|-----------------------|--------------|
| A01 Broken Access Control | Y | Central authZ redesign needed |
| A02 Cryptographic Failures | Y | Token & transport review (internal TLS) |
| A03 Injection | Y | AI output & potential SQL via ORM misuse |
| A04 Insecure Design | Y | Rate limiting & AI trust patterns |
| A05 Security Misconfiguration | Y | Internal unencrypted traffic |
| A06 Vulnerable & Outdated Components | Y | AI provider libs & dependencies |
| A07 Identification & Authentication Failures | Y | STS monitoring gaps |
| A08 Software & Data Integrity Failures | P | Supply chain attestation pending |
| A09 Security Logging & Monitoring Failures | Y | DB write & auth issuance visibility |
| A10 SSRF | N | Not currently exposed endpoints |
| API1 BOLA | P | Potential if object auth not enforced |
| API2 Broken Authentication | Y | Token issuance brute force detection |
| API3 BOPLA | P | Field-level authorization review pending |
| API4 Resource Consumption | Y | Gateway rate abuse |
| API5 Function Level Auth | Y | Privileged route gaps |
| API6 Sensitive Business Flows | P | Data minimization WIP |
| API7 SSRF | N | No server-side fetch endpoints |
| API8 Security Misconfiguration | Y | Config hardening needed |
| API9 Inventory Management | P | Need API inventory & versioning |
| API10 Unsafe Consumption | P | 3rd party AI outputs | 
| M1 Platform Usage | P | Mobile security baseline incomplete |
| M2 Insecure Data Storage | P | Mobile secure storage validation needed |
| M3 Insecure Communication | Y | Internal TLS gap |
| M4 Insecure Authentication | Y | Mobile token handling |
| M5 Insufficient Cryptography | P | Ensure strong alg reuse |
| M6 Insecure Authorization | Y | Consistency issues |
| M7 Client Code Quality | P | Hardening & obfuscation |
| M8 Code Tampering | P | Anti-tamper not implemented |
| M9 Reverse Engineering | P | Obfuscation tasks pending |
| M10 Extraneous Functionality | N | No debug endpoints known |
| LLM01 Prompt Injection | Y | Output sanitization gap |
| LLM02 Insecure Output Handling | Y | Need schema validation |
| LLM03 Training Data Poisoning | P | Provider assurance needed |
| LLM04 Model Denial of Service | Y | Rate & size controls missing |
| LLM05 Supply Chain Vulnerabilities | Y | Provider risk, attestation absent |
| LLM06 Sensitive Information Disclosure | Y | Prompt redaction required |
| LLM07 Insecure Plugin Design | N | No plugins |
| LLM08 Excessive Agency | N | No autonomous actions |
| LLM09 Overreliance | P | Monitoring human review safeguards |
| LLM10 Model Theft | P | Access control to model keys |

## 9. NIST Alignment Matrices
### 9.1 Control Family Mapping (Aggregate)
| NIST Family | Threat IDs | Coverage Notes |
|-------------|-----------|----------------|
| AC | T-001,T-002,T-003,T-004,T-006,T-010 | Authorization & access control central |
| IA | T-001,T-004,T-009 | Identity assurance gaps |
| AU | T-001,T-002,T-004,T-007 | Logging & audit expansion needed |
| SC | T-002,T-003,T-005,T-006,T-010 | Communications & boundary protection |
| SI | T-002,T-005,T-007 | System & info integrity monitoring |
| CM | T-008 | Configuration / change mgmt for AI provider |
| IR | (future) | Incident response not yet mapped |
| CP | T-006 | Continuity via resilience design |
| CA | (future) | Assessment processes to add |
| SA | T-008 | Supply chain & acquisition |
| SR | T-003,T-008 | Supply chain risk management |
| PM | (future) | Governance program formalization |
| RA | (implicit) | Formal risk assessment iteration pending |

### 9.2 SSDF Practice Mapping
| SSDF Group | Related Threat IDs | Maturity (None/Low/Med/High) | Notes |
|------------|--------------------|------------------------------|-------|
| PO | T-008 | Low | Third-party policy formation |
| PS | T-001,T-003 | Low | Secure design reviews immature |
| PW | T-005,T-010 | Medium | Some secure coding practices |
| RV | T-001,T-002,T-007 | Low | Verification & logging gaps |

### 9.3 Zero Trust & Identity
| Domain | Gap / Observation | Related Threats | Action Needed |
|--------|-------------------|-----------------|---------------|
| Continuous Verification | Minimal token anomaly detection | T-004 | Add behavioral analytics |
| Least Privilege / JIT | Coarse role scopes | T-001 | Redesign scopes & deny default |
| Micro-Segmentation | Flat internal network | T-002,T-006 | Network policy & mTLS |
| Workload Identity | Service identity conflated | T-001 | Split per-service identities |
| Session Resilience | Token revocation latency | T-004 | Shorter TTL + revoke list |

### 9.4 AI RMF Mapping (If Applicable)
| AI RMF Function | Threat IDs | Metrics / Controls | Notes |
|-----------------|-----------|--------------------|-------|
| Govern | T-003,T-008 | Data handling policy | Formalization needed |
| Map | T-003 | Data inventory for prompts | In progress |
| Measure | T-005 | Injection detection metrics | To be defined |
| Manage | T-003,T-005 | Redaction & filtering processes | Pending automation |

## 10. Supply Chain & Dependency Integrity
| Area | Current State | Gaps | Related Threats | Action |
|------|---------------|------|-----------------|--------|
| SBOM Generation | Manual ad hoc | Lacks continuous SBOM | T-008 | Automate CI SBOM (CycloneDX) |
| Vulnerability Scanning | Basic CVE scan | No SCA for AI libs | T-008 | Integrate SCA pipeline |
| Artifact Signing | Not implemented | Integrity unverified | T-002,T-008 | Adopt signing (Sigstore) |
| Build Pipeline Hardening | Basic | Limited isolation | T-002 | Harden runners, secrets mgmt |
| Model/Dataset Provenance | Unknown | No attestation | T-003,T-008 | Request provider attestations |
| Third-Party Services | Single provider | No redundancy | T-003,T-008 | Evaluate multi-provider strategy |
| License / Policy Gates | Manual review | Not automated | T-008 | Add policy-as-code gate |

## 11. Detection & Monitoring Coverage
| Event Type | Logged? (Y/N) | Integrity Protected? | Alerted? | MTTD Target | MTTR Target | Gaps / Notes | Threat IDs |
|------------|---------------|----------------------|----------|-------------|-------------|--------------|------------|
| Auth Success/Failure | P | N | N | <15m | <4h | Need central STS log aggregation | T-004 |
| Privilege Changes | N | N | N | <30m | <4h | Scope changes not tracked | T-001 |
| Data Export / Exfil | N | N | N | <30m | <8h | No egress monitoring | T-003 |
| Model Override / Config Change | N | N | N | <30m | <8h | No model config audit | T-005,T-008 |
| Rate Limit Breaches | P | N | P | <10m | <2h | Static threshold only | T-006 |
| Anomaly (Enumeration) | N | N | N | <15m | <4h | No behavior analytics | T-001,T-004 |
| Prompt Injection Flag | N | N | N | <15m | <4h | No detection logic | T-005 |
| Critical Service Failure | P | N | P | <5m | <1h | Health only, no root cause trace | T-006 |

## 12. Mitigation Roadmap
| Threat ID | Mitigation Action | Type | Owner | Start | Target Date | Ticket # | SSDF Group | Dependencies |
|-----------|-------------------|----------|-------|-------|--------|----------|------------|--------------|
| T-001 | Deploy centralized policy engine & route mapping | Prevent | SecEng | 2025-10-01 | 2025-10-31 | TBA | PS | None |
| T-002 | Enable DB TLS & auditing | Prevent | DBA | 2025-10-01 | 2025-10-15 | TBA | RV | Network readiness |
| T-003 | Implement prompt redaction service | Prevent | AppSec | 2025-10-05 | 2025-10-31 | TBA | PS | Data classification |
| T-005 | Add AI response schema validator & filters | Prevent | DevLead | 2025-10-10 | 2025-11-15 | TBA | PW | Prompt redaction |
| T-006 | Adaptive rate limiting & circuit breakers | Prevent | SecOps | 2025-10-05 | 2025-10-31 | TBA | PW | Gateway upgrade |
| T-007 | Immutable DB write audit logging | Detect | DBA | 2025-10-05 | 2025-11-01 | TBA | RV | TLS (T-002) |
| T-008 | Provider attestation & SCA integration | Prevent | Vendor | 2025-10-05 | 2025-11-15 | TBA | PO | Contract reviews |
| T-004 | Token issuance anomaly detection | Detect | SecEng | 2025-10-05 | 2025-10-31 | TBA | RV | Central logs |
| T-009 | Mobile cert pinning & secure storage | Prevent | Mobile | 2025-10-10 | 2025-11-10 | TBA | PW | App release cycle |
| T-010 | Response field-level filtering | Prevent | DevLead | 2025-10-10 | 2025-11-01 | TBA | PW | Policy engine |

## 13. Residual Risk Register
| Threat ID | Original Risk | Residual Risk | Delta | Rationale for Acceptance / Further Action |
|-----------|---------------|---------------|-------|-------------------------------------------|
| T-001 | 56 | 24 | -32 | Central policy reduces likelihood |
| T-002 | 54 | 18 | -36 | TLS & auditing lower likelihood & impact |
| T-003 | 42 | 15 | -27 | Redaction reduces sensitive exposure |
| T-004 | 40 | 15 | -25 | Detection lowers likelihood & impact |
| T-005 | 40 | 15 | -25 | Schema & size controls reduce risk |
| T-006 | 36 | 12 | -24 | Adaptive limits reduce attack success |
| T-007 | 35 | 12 | -23 | Immutable logs enable rapid response |
| T-008 | 32 | 15 | -17 | Provider assurance & SCA reduce risk |
| T-009 | 28 | 10 | -18 | Cert pinning & storage hardening |
| T-010 | 25 | 8 | -17 | Filtering restricts data disclosure |

## 14. Accepted / Deferred Risks
| ID | Description | Risk Level | Justification | Review Date | Compensating Controls |
|----|-------------|-----------|--------------|-------------|-----------------------|
| EX-01 | Lack of redundancy for AI provider | Medium (T-008 residual) | Cost & timeline constraints | 2026-01-15 | Monitoring anomaly scoring |

## 15. Change Log / Review History
| Date | Author/Reviewer | Section Updated | Summary of Changes |
|------|----------|----------------|-------------------|
| 2025-09-28 | System (AI) / Reviewer TBD | Initial | First full extended threat model from ArchiScribe data |

## 16. Appendix (Optional)
### 16.1 Data Flow Descriptions
1. Mobile App → STS (Auth Request, HTTPS) – User credentials exchanged for token
2. STS → Mobile App (Auth Token, HTTPS) – JWT or similar returned
3. Mobile App → API Gateway (Request, HTTPS) – Authenticated API call
4. API Gateway → App Service (Request, HTTP) – Routed internal request
5. App Service → Database (SQL Query) – ORM query execution
6. Database → App Service (Result Set) – Data returned
7. App Service → AI Agent Service (Request, HTTPS) – Prompt with user/context
8. AI Agent Service → App Service (Response, HTTPS) – Generated content
9. App Service → API Gateway (Response, JSON) – Application response
10. API Gateway → Mobile App (Response, JSON) – Final result to client

### 16.2 Glossary
| Term | Definition |
|------|------------|
| BOLA | Broken Object Level Authorization |
| BOPLA | Broken Object Property Level Authorization |
| Residual Risk | Post-mitigation Likelihood × Impact |

### 16.3 Risk Scoring Method
Likelihood (L) and Impact (I) scored 1–10 using qualitative scales (Low=1-3, Moderate=4-6, High=7-8, Critical=9-10). Raw Risk = L × I. Bands: Low (≤20), Medium (21–55), High (≥56). Residual Risk recalculated post-control estimates.

### 16.4 Assumption Validation Actions
- Validate HSM/KMS configuration for STS keys (evidence: KMS policy export)
- Confirm internal network path encryption feasibility and plan (architect approval)
- Obtain AI provider data retention & deletion SLA (contract addendum)
- Pen-test authorization policy engine after deployment (security test report)

### 16.5 Next Steps (90-Day Outlook)
1. Complete foundational Prevent controls (T-001, T-002, T-003) – prerequisite for downstream detection tuning.
2. Implement detection pipeline enhancements (privilege change logging, anomaly detection for tokens & AI prompts).
3. Establish SBOM automation & signing to reduce supply chain unknowns.
4. Formalize Zero Trust roadmap (segmentation + workload identity separation).
5. Schedule reassessment workshop after control deployments to update residual scores.
