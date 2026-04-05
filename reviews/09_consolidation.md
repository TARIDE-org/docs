# Review Consolidation: TARIDE Foundation v0.51

**Date:** 2026-04-05

**Reviews consolidated:** Privacy (01), Security (02), Technical (03), Performance (04), Reliability (05), Commercial (06), Cost (07), Supportability (08)

---

## Executive Summary

Eight independent review passes produced **84 findings**: 14 blockers, 47 major, 23 minor. After deduplication and cross-referencing, these consolidate into **6 cross-cutting themes** and a **spec-readiness checklist of 12 items** that must be resolved before the foundation document can support a specification.

The document is strong on vision, architectural intent, and European positioning. It is weak on operational specifics, financial grounding, governance mechanics, and the bridge from concept to adoption. The five-layer architecture is sound. The gap is between describing *what* the protocol does and specifying *how* it does it.

---

## Cross-cutting Themes

### Theme 1: The Trust Chain Has No Formal Model

Across privacy, security, and technical reviews, the same gap appears: the document describes trust relationships but never formalises them.

- No threat model (SEC-001)
- No legal basis per processing activity (PRIV-001)
- Trust assumptions about attestation providers are implicit (SEC-003, SEC-010)
- Cache node trust is assumed, not verified (SEC-005)
- CAMARA API trust is assumed (SEC-012)

**The core question:** *Who trusts whom, for what, and what happens when that trust breaks?*

**Related findings:** PRIV-001, PRIV-003, SEC-001, SEC-003, SEC-005, SEC-010, SEC-012, TECH-005

---

### Theme 2: Key Lifecycle Is the Foundation's Foundation

The entire protocol's security rests on private key control. Yet key management appears in six reviews as an unresolved problem:

- Key storage requirements unspecified (SEC-002)
- Key loss = DID loss = reputation loss — no recovery (SEC-002)
- Key rotation undefined (SEC-002)
- DID deactivation procedure missing (SEC-009)
- Multi-device scenarios unaddressed (SEC-002)
- Key compromise response undefined (SEC-002)

Without solving this, every other protocol property is theoretical.

**Related findings:** SEC-002, SEC-009, TECH-001, REL-004

---

### Theme 3: On-chain vs. Off-chain Is Inconsistent and Consequential

The boundary between what lives on the blockchain and what lives in the resolver network is the most consequential architectural decision — it determines privacy, cost, performance, and erasure compliance. The document contradicts itself:

- Use case tables say "On-chain (credential)" (TECH-004)
- GDPR section says "only cryptographic commitments on-chain" (PRIV-002)
- Reputation is off-chain with daily hash on-chain (PERF-005)
- Instance-DID associations are off-chain with commitments on-chain
- DID age is "immutable" on-chain — erasure tension (PRIV-002)

**The core question:** *Exactly what bytes are written to the blockchain, and is that compatible with GDPR erasure?*

**Related findings:** PRIV-002, TECH-004, TECH-006, PERF-003, PERF-005, COST-002

---

### Theme 4: The Calmido-Foundation Entanglement

The separation between Calmido and the Foundation is structurally defined but operationally non-existent:

- Calmido is the only application (BIZ-002)
- Foundation likely funded by Calmido founders (COST-001, BIZ-004)
- Foundation likely staffed by Calmido personnel (SUP-005)
- PoC infrastructure likely Calmido infrastructure (REL-002)
- Protocol design influenced by Calmido's product needs (SUP-006)

This isn't necessarily wrong at the early stage — but it must be transparent and time-bound, or the "independent foundation" claim undermines credibility with telecoms, government, and grant bodies.

**Related findings:** BIZ-002, BIZ-004, COST-001, COST-006, REL-002, SUP-005

---

### Theme 5: No Numbers Anywhere

The document contains one financial figure (€0.001 × 10M lookups = €3.65M). Across eight reviews, the absence of quantitative analysis is the most consistent gap:

- No capacity model (PERF-001)
- No financial model (COST-001)
- No latency budget (PERF-002)
- No cost estimates for any component (COST-002, COST-003, COST-004)
- No adoption projections beyond 10K DIDs (BIZ-007)
- No availability targets (REL-001)
- No transaction volume estimates (PERF-003)

A specification encodes numbers: timeout values, cache TTLs, credential validity periods, SLA targets, rate limits. Without a foundation of quantitative analysis, those numbers will be arbitrary.

**Related findings:** PERF-001, PERF-002, PERF-003, COST-001, COST-002, COST-003, BIZ-007, REL-001, TECH-007

---

### Theme 6: Governance Is the Weakest Link

Every review that touched governance found it insufficient:

- No decision-making process (SUP-001)
- No enforcement due process (SUP-003)
- No protocol change process (SUP-006)
- No onboarding for any participant (SUP-002)
- No incident response (SEC-014)
- No dispute resolution (SUP-003)
- No audit process (SUP-007)
- Consumer representation missing (BIZ-009)
- Foundation operational SPOF (REL-010)

The document promises "multi-stakeholder governance" — the strongest version of that promise requires the strongest governance specification. Currently, governance is the section most disproportionate to the ambition.

**Related findings:** SUP-001, SUP-002, SUP-003, SUP-005, SUP-006, SUP-007, SEC-014, BIZ-009, REL-010

---

## Deduplicated Blocker List

14 blockers were identified across 8 reviews. After deduplication (some findings are the same issue seen from different angles), **12 unique blockers** remain:

| # | Blocker | Source | Theme |
|---|---|---|---|
| 1 | **Legal basis per processing activity undefined** | PRIV-001 | Trust model |
| 2 | **Right to erasure vs. blockchain unresolved** | PRIV-002 | On-chain/off-chain |
| 3 | **No threat model** | SEC-001 | Trust model |
| 4 | **Key lifecycle (storage, recovery, rotation) undefined** | SEC-002 | Key lifecycle |
| 5 | **Attestation provider compromise unmitigated** | SEC-003 | Trust model |
| 6 | **DID method not selected** | TECH-001 | Technical foundations |
| 7 | **Credential format not selected** | TECH-002 | Technical foundations |
| 8 | **Revocation mechanism not specified** | TECH-003 | Technical foundations |
| 9 | **No capacity model** | PERF-001 | No numbers |
| 10 | **No availability target** | REL-001 | No numbers |
| 11 | **Cold-start problem unsolved** | BIZ-001 | Calmido entanglement |
| 12 | **Governance structure not operational** | SUP-001 + SUP-002 | Governance |

---

## Spec-Readiness Checklist

The following 12 items must be resolved before the foundation document can credibly support a protocol specification. They are ordered by dependency — earlier items inform later ones.

### Tier 1: Resolve before any specification work begins

These are prerequisites. Specification decisions cannot be made without them.

| # | Item | Blockers addressed | Deliverable |
|---|---|---|---|
| **1** | **Write a threat model** | SEC-001, SEC-003 | Standalone document: adversary classes, capabilities, security goals, trust assumptions |
| **2** | **Select DID method, credential format, and revocation mechanism** | TECH-001, TECH-002, TECH-003 | Technical decision document with justification, migration path, and eIDAS wallet compatibility assessment |
| **3** | **Define the on-chain/off-chain boundary** | PRIV-002, TECH-004, PERF-003, COST-002 | Data location matrix: every data element, its storage location, its on-chain representation (if any), and its erasure path |
| **4** | **Determine legal basis per processing activity** | PRIV-001 | Legal analysis document, input to DPIA |

### Tier 2: Resolve before specification draft

These require Tier 1 outputs as input.

| # | Item | Blockers addressed | Deliverable |
|---|---|---|---|
| **5** | **Design key lifecycle** | SEC-002 | Key management specification: generation, storage requirements, rotation, recovery, compromise response, multi-device |
| **6** | **Build capacity model and financial model** | PERF-001, COST-001, REL-001 | Three-scenario model (PoC / Pilot / EU): DIDs, lookups/sec, transactions/day, infrastructure costs, revenue projections, break-even |
| **7** | **Define resolver network architecture** | TECH-005, PERF-002, PERF-004, REL-003 | Protocol specification for: data replication, cache invalidation, consistency model, query routing, API schema, SLA targets |
| **8** | **Draft governance bylaws** | SUP-001, SUP-002 | Operational governance: decision process, board mechanics, protocol change process, enforcement framework, onboarding requirements per participant type |

### Tier 3: Resolve before pilot

These can proceed in parallel with specification drafting but must be complete before real-world deployment.

| # | Item | Blockers addressed | Deliverable |
|---|---|---|---|
| **9** | **Conduct DPIA** | PRIV-002, PRIV-012 | Data Protection Impact Assessment before PoC touches real personal data |
| **10** | **Solve cold-start with concrete strategy** | BIZ-001, BIZ-002 | Minimum viable ecosystem definition, telecom acquisition strategy with timeline and fallback, Calmido-independent survival path |
| **11** | **Define onboarding paths** | SUP-002 | Participation brief per stakeholder type: prerequisites, integration steps, legal requirements, testing, ongoing obligations |
| **12** | **Establish incident response framework** | SEC-014, REL-007 | Incident classification, escalation paths, communication channels, emergency credential revocation procedure |

---

## Findings by Severity — Full Index

### Blockers (14)

| ID | Domain | Issue |
|---|---|---|
| PRIV-001 | Privacy | Legal basis not specified |
| PRIV-002 | Privacy | Erasure vs. blockchain unresolved |
| SEC-001 | Security | No threat model |
| SEC-002 | Security | Key lifecycle undefined |
| SEC-003 | Security | Attestation provider compromise unmitigated |
| TECH-001 | Technical | DID method not selected |
| TECH-002 | Technical | Credential format not selected |
| TECH-003 | Technical | Revocation mechanism not specified |
| PERF-001 | Performance | No capacity model |
| REL-001 | Reliability | No availability target |
| BIZ-001 | Commercial | Cold-start unsolved |
| BIZ-002 | Commercial | Calmido dependency existential |
| COST-001 | Cost | No financial model |
| SUP-001 | Supportability | Governance not operational |

### Major (47)

| ID | Domain | Issue |
|---|---|---|
| PRIV-003 | Privacy | Cache node controller/processor misclassification |
| PRIV-004 | Privacy | Reputation = profiling, unaddressed |
| PRIV-005 | Privacy | "Consent" term conflation |
| PRIV-006 | Privacy | Credential metadata lacks purpose limitation |
| PRIV-007 | Privacy | No data retention policy |
| PRIV-008 | Privacy | Transparency to data subjects underspecified |
| PRIV-009 | Privacy | Cross-border transfer mechanisms absent |
| PRIV-010 | Privacy | Continuous validation surveillance risk |
| SEC-004 | Security | On-chain correlation oracle |
| SEC-005 | Security | Cache node response authentication missing |
| SEC-006 | Security | Post-quantum migration not designed |
| SEC-007 | Security | UUID enforcement fragile |
| SEC-008 | Security | Reputation manipulation vectors |
| SEC-009 | Security | DID deactivation undefined |
| SEC-010 | Security | One-instance-one-DID enforcement depends on AP honesty |
| SEC-011 | Security | DDoS / availability unaddressed |
| TECH-004 | Technical | On-chain/off-chain boundary inconsistent |
| TECH-005 | Technical | Resolver network is a metaphor, not design |
| TECH-006 | Technical | Blockchain selection deferred but consequential |
| TECH-007 | Technical | 500ms target unvalidated |
| TECH-008 | Technical | Credential metadata schema undefined |
| TECH-009 | Technical | Credential dependency chain undefined |
| TECH-010 | Technical | No protocol versioning |
| PERF-002 | Performance | 500ms SLA not decomposed |
| PERF-003 | Performance | Blockchain throughput bottleneck |
| PERF-004 | Performance | Cache coherence undefined |
| PERF-005 | Performance | Reputation commitment scalability |
| REL-002 | Reliability | PoC single point of failure |
| REL-003 | Reliability | Attestation provider cascading failure |
| REL-004 | Reliability | Credential renewal lifecycle |
| REL-005 | Reliability | Portability transition window |
| REL-006 | Reliability | Blockchain availability ceiling |
| REL-007 | Reliability | No observability framework |
| BIZ-003 | Commercial | Telecom business case not demonstrated |
| BIZ-004 | Commercial | Revenue gap 2026-2029 |
| BIZ-005 | Commercial | Pricing misalignment |
| BIZ-006 | Commercial | Competitive positioning incomplete |
| BIZ-007 | Commercial | 10K DIDs by 2031 unambitious |
| COST-002 | Cost | Blockchain costs unquantified |
| COST-003 | Cost | Infrastructure costs not estimated |
| COST-004 | Cost | Compliance/legal costs invisible |
| COST-005 | Cost | Revenue-sharing unviable at small scale |
| COST-006 | Cost | EU grant dependency high-risk |
| SUP-002 | Supportability | No onboarding paths |
| SUP-003 | Supportability | Enforcement without due process |
| SUP-004 | Supportability | Documentation strategy absent |
| SUP-005 | Supportability | Staffing vs. operational requirements |
| SUP-006 | Supportability | Protocol evolution governance missing |
| SUP-007 | Supportability | Audit right without process |

### Minor (23)

| ID | Domain | Issue |
|---|---|---|
| PRIV-011 | Privacy | Organisation affiliation surveillance risk |
| PRIV-012 | Privacy | DPIA timing too late |
| PRIV-013 | Privacy | Lookup logs as surveillance dataset |
| SEC-012 | Security | CAMARA API trust assumed |
| SEC-013 | Security | Smart contract audit/upgrade absent |
| SEC-014 | Security | No incident response framework |
| TECH-011 | Technical | EUDI Wallet integration hand-waved |
| TECH-012 | Technical | Reputation algorithm unspecified |
| TECH-013 | Technical | Logo delivery performance/security |
| PERF-006 | Performance | Logo as CDN problem |
| PERF-007 | Performance | Reputation ingestion underestimated |
| PERF-008 | Performance | On-chain verification in hot path |
| REL-008 | Reliability | No backup/disaster recovery |
| REL-009 | Reliability | Revocation propagation no delivery guarantee |
| REL-010 | Reliability | Foundation operational SPOF |
| BIZ-008 | Commercial | IP strategy incomplete |
| BIZ-009 | Commercial | Consumer governance representation missing |
| COST-007 | Cost | "Free to users" misleading |
| COST-008 | Cost | Premium credential pricing no benchmark |
| SUP-008 | Supportability | No ecosystem health metrics |
| SUP-009 | Supportability | Open-source community model undefined |
| SUP-010 | Supportability | Internationalisation not considered |

---

## Overall Assessment

**The document is not ready for specification, but it is a strong foundation document.**

What works well:
- The five-layer architecture is well-conceived and the layer separation is clean
- "Pseudonymity by default, identification by choice" is a genuine differentiator
- "Time as trust" is an elegant anti-sybil mechanism
- The GDPR section (v0.51) shows serious engagement with data protection
- The SIM swap analysis is honest about what the protocol can and cannot do
- European positioning and eIDAS alignment are compelling
- The writing is clear and accessible

What must improve:
- **Formalise the trust model.** The document describes trust intuitively but never formally. A threat model and trust assumptions document is prerequisite to specification.
- **Make technical choices.** DID method, credential format, revocation mechanism, blockchain — these are not specification details, they are foundation-level decisions that affect everything else.
- **Add numbers.** Capacity model, financial model, latency budgets, availability targets. The specification will encode quantitative parameters; the foundation document must provide the basis.
- **Build governance.** The governance section must match the ambition. Multi-stakeholder governance requires operational mechanics, not just structural protections.
- **Solve the bootstrap.** The commercial viability of the protocol is not a post-specification concern. If the cold-start problem doesn't have a credible solution, the specification is academic.

The path from v0.51 to spec-ready is substantial but clear. The 12-item checklist above provides a prioritised sequence. Tier 1 items (threat model, technical selections, data boundary, legal basis) should be addressed as a v0.6 update to the foundation document. Tier 2 items (key lifecycle, capacity model, resolver architecture, governance bylaws) can be separate companion documents that the specification references. Tier 3 items (DPIA, cold-start strategy, onboarding, incident response) can proceed in parallel with specification drafting.
