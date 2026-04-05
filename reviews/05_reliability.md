# Review 5: Reliability & Operations

**Reviewer perspective:** Senior SRE / platform reliability engineer with expertise in distributed systems operations, incident management, and telecom-grade availability.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-04

---

## Findings

### REL-001 — No availability target or SLA defined
**Severity:** Blocker
**Section:** Throughout

The protocol positions itself as trust infrastructure for real-time communications. The resolver network must respond within 500ms for incoming calls. But the document defines no availability target:

1. **What uptime is required?** Telecom-grade availability is "five nines" (99.999% = ~5 minutes downtime/year). Web-service-grade is "three nines" (99.9% = ~8.7 hours/year). The difference is orders of magnitude in cost and architectural complexity.
2. **Who commits to the SLA?** The Foundation? Cache node operators? Attestation providers? Each dependency in the chain must meet or exceed the overall target.
3. **What constitutes "down"?** Resolver unreachable? Stale data served? Incorrect data served? Partial degradation (some DIDs unavailable)?

If the protocol is positioned as infrastructure ("not a product, a protocol"), it must define infrastructure-grade availability expectations. Otherwise, every cache node operator sets their own standard and users get unpredictable service.

**Recommendation:** Define an availability target per component (attestation provider, cache node, on-chain layer). Define what "available" means (correct response within SLA latency). Specify minimum requirements for cache node operators as a condition of network participation.

---

### REL-002 — Single points of failure in the PoC architecture
**Severity:** Major
**Section:** Development roadmap, resolver layer

"In the proof of concept, the foundation operates both roles [attestation provider and cache node]." This means:

1. The Foundation is a single point of failure for the entire protocol
2. If the Foundation's infrastructure goes down, all lookups fail
3. If the Foundation's systems are compromised, all trust data is compromised
4. There is no failover, no redundancy, no geographic distribution

The document acknowledges this is temporary, but the PoC is the phase where real users (Calmido users) will experience the protocol. A PoC outage damages credibility with the telecom partners the document identifies as critical dependencies.

**Recommendation:** Even in the PoC, run at least two independent cache node instances in different availability zones. Define the transition plan to independent operators with concrete milestones — don't leave it as "production phase."

---

### REL-003 — Attestation provider dependency creates cascading failures
**Severity:** Major
**Section:** Verification layer, resolver layer

The attestation provider is a hard dependency for: credential issuance, credential renewal, credential revocation, continuous validation, and metadata updates. If KPN's attestation provider systems go down:

1. New user registrations with KPN numbers fail (no credential issuance)
2. Number portability to/from KPN stalls (no revocation or issuance)
3. Continuous validation for KPN numbers fails (SIM checks unavailable)
4. Metadata updates stop (last_sim_swap, account_age freeze)

The document does not address:
- Graceful degradation when an attestation provider is unreachable
- Whether cache nodes serve stale data or return "provider unavailable"
- Timeout and retry policies for attestation provider communication
- How the protocol distinguishes "provider down" from "credential revoked"

At EU scale with dozens of attestation providers, at least one will be experiencing issues at any given time.

**Recommendation:** Define degradation modes per attestation provider failure scenario. Specify cache TTLs — how long is a cached credential valid when the attestation provider is unreachable? Design "last known good" serving with explicit staleness indicators. Ensure the UX distinguishes "verified (cached, provider unreachable)" from "verified (live confirmation)."

---

### REL-004 — Credential renewal lifecycle creates maintenance windows
**Severity:** Major
**Section:** Verification layer, number portability

Credentials are "time-bound" (expiring) and revocable. This means:

1. Every credential has an expiry date requiring proactive renewal
2. Renewal requires the attestation provider to be available
3. If renewal fails (provider down, network issue, user's phone off), the credential expires and the user becomes "unverified" through no fault of their own
4. At scale, credential renewal creates a continuous background load on attestation providers

The document mentions "periodic renewal" for the PoC but doesn't specify:
- Credential validity period (hours? days? months?)
- Renewal window (how far before expiry does renewal start?)
- Failure handling (what if renewal fails repeatedly?)
- Whether renewal is user-initiated or automatic
- Grace period after expiry before the credential is treated as invalid

Mass credential expiry (e.g., an attestation provider sets all credentials to 30-day validity, 1/30th expire each day) creates a daily renewal load of thousands of operations. If the provider has a one-day outage, all credentials expiring that day become invalid.

**Recommendation:** Define credential validity periods and renewal mechanics. Design a grace period for expired-but-not-revoked credentials. Consider long validity periods (1 year) with short-lived presentation tokens to reduce renewal load while maintaining freshness.

---

### REL-005 — Number portability transition window is operationally undefined
**Severity:** Major
**Section:** Number portability, protocol details

"During the transition, there is a brief window in which the number's verification status is in flux." The document calls this a "credential renewal" but operationally:

1. Odido must revoke → propagation delay → KPN must issue → propagation delay
2. These are two independent attestation providers with no shared state
3. The user's phone may be unreachable during the port (SIM swap to new provider)
4. In the Netherlands, number porting takes 1-3 business days

During this window:
- The user's trust profile shows "unverified" or "in transition"
- If the user receives calls, they appear unverified to callers checking them
- If the user makes calls, recipients see an unverified caller

For businesses (ING Bank with ported numbers), this could last days and affect customer trust.

**Recommendation:** Define the portability state machine explicitly: states (verified → revoking → unverified → pending → verified), transitions, timeouts, and the trust profile displayed at each state. Consider a "porting" status distinct from "unverified" so the UX can differentiate.

---

### REL-006 — Blockchain dependency creates an availability ceiling
**Severity:** Major
**Section:** Registration layer, resolver layer

The protocol depends on the blockchain for: DID registration, credential commitments, and reputation anchoring. Blockchain availability limits:

1. **Ethereum L2 sequencer outages**: Arbitrum had multi-hour sequencer outages in 2023-2024. During a sequencer outage, no new L2 transactions are processed. New DID registrations and credential operations are blocked.
2. **EBSI availability**: EBSI is a pre-production network. Its availability track record is unknown. Permissioned networks with fewer validators are more vulnerable to correlated failures.
3. **Gas price spikes**: On public networks, sudden gas price increases can make transactions economically infeasible. A batch of credential revocations during a gas spike may be delayed.

The resolver network can serve cached data during blockchain downtime, but:
- No new DIDs can be registered
- No credentials can be committed
- No revocations can be anchored
- The integrity guarantee (verify against on-chain commitments) degrades

**Recommendation:** Design for blockchain unavailability. Define which operations can proceed off-chain during an outage (with deferred on-chain anchoring). Set a maximum acceptable blockchain outage duration. Monitor blockchain health as a first-class operational metric.

---

### REL-007 — No observability or monitoring framework
**Severity:** Major
**Section:** Throughout

The document describes a complex distributed system with multiple operator types but mentions no monitoring:

1. **Health metrics** — lookup latency, error rates, cache hit rates, credential issuance rates, revocation propagation times
2. **Anomaly detection** — sudden spike in DID registrations (sybil attack?), mass credential revocations (compromised provider?), cache node serving stale data
3. **Operational dashboards** — for the Foundation, for cache node operators, for attestation providers
4. **Alerting** — who gets paged when something breaks?
5. **Audit logs** — who changed what, when, in the resolver network

For a protocol with independent operators, observability is how you maintain system-wide reliability without controlling every component.

**Recommendation:** Define minimum observability requirements for each operator type. Specify protocol-level health metrics that the Foundation monitors. Include observability in the cache node operator requirements.

---

### REL-008 — No backup or disaster recovery model
**Severity:** Minor
**Section:** Throughout

The on-chain data is inherently replicated (blockchain). But the off-chain data in the resolver network (instance-DID associations, reputation scores, consent preferences) has no described backup strategy:

1. If a cache node loses its data store, how does it rebuild?
2. If an attestation provider loses its credential database, how are credentials re-issued?
3. If the Foundation's coordination systems fail, who takes over?
4. Is there a "source of truth" for off-chain data, or is it distributed with no single authoritative copy?

**Recommendation:** Define the source of truth for each data category. Specify backup and recovery procedures for attestation providers and cache nodes. Define the RTO (recovery time objective) and RPO (recovery point objective) per component.

---

### REL-009 — Revocation propagation has no delivery guarantee
**Severity:** Minor
**Section:** Revocation and enforcement, resolver layer

"Revocation propagates through the network in real time." But in a distributed network of independent cache nodes:

1. What if a cache node is temporarily unreachable when the revocation is published?
2. Is there a message queue with guaranteed delivery, or is it best-effort?
3. How does the system detect that a cache node is serving a revoked credential?
4. What is the maximum time a revoked credential can still appear valid on any cache node?

This is a safety-critical propagation — a revoked credential served as valid could enable fraud.

**Recommendation:** Specify the revocation propagation mechanism with delivery guarantees. Define maximum acceptable propagation delay. Implement a protocol-level mechanism for detecting stale revocations (e.g., cache nodes periodically check revocation status against the authoritative source).

---

### REL-010 — Foundation as single coordinator is a governance SPOF
**Severity:** Minor
**Section:** Governance, application registration

The Foundation performs several irreplaceable coordination roles:
- Maintains the attestation provider registry
- Manages application UUID registration and revocation
- Coordinates joint controller arrangements (GDPR)
- Operates (initially) both attestation provider and cache node roles
- Conducts audits of applications

If the Foundation becomes operationally incapacitated (funding crisis, legal dispute, key personnel departure), the entire ecosystem lacks coordination. The structural protections (cannot be sold, mission-protected) protect against commercial capture but not against operational failure.

**Recommendation:** Define a continuity plan for Foundation operational failure. Identify which Foundation functions can be delegated or automated. Consider whether the attestation provider registry and application registry should be on-chain (self-sustaining) rather than Foundation-operated.

---

## Open Questions

1. **What is the target availability for each component?** Without this, operators cannot design their infrastructure.
2. **Who operates the "resolver network" as a coordinated system?** Individual cache nodes are independent, but someone must manage the network topology, propagation channels, and health monitoring.
3. **What is the disaster recovery scenario for a compromised Foundation?** If the Foundation's signing keys are compromised, can it recover without resetting the entire ecosystem?
4. **How are cache node operators incentivised to maintain high availability?** The document mentions lookup fees, but is there a penalty for downtime?
5. **What is the operational runbook for a number portability event?** Is this documented for attestation providers, or does each operator figure it out independently?
6. **What happens when an attestation provider permanently shuts down?** (Telecom goes bankrupt, exits market) Are all their credentials orphaned?
7. **How is the system tested at scale before production?** Is there a load-testing strategy for the pilot-to-production transition?

---

## Spec Readiness Verdict: Reliability & Operations

**Not ready for specification.**

One blocker:
- **REL-001**: No availability target defined. A protocol cannot be specified without knowing what "reliable" means in concrete terms.

Six major findings need resolution:
- REL-002 (PoC SPOF), REL-003 (attestation provider cascading failure), REL-004 (credential renewal lifecycle), REL-005 (portability state machine), REL-006 (blockchain availability ceiling), REL-007 (no observability)

The document's distributed architecture (multiple cache nodes, multiple attestation providers) is inherently resilient in theory. But resilience must be designed, not assumed. The gap between "the architecture supports redundancy" and "the specification requires specific redundancy levels with defined failure modes" is where operational reliability lives. The document is on the right side of architectural intent but has not crossed into operational specification.
