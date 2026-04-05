# Review 3: Technical Architecture

**Reviewer perspective:** Senior distributed systems architect with expertise in protocol design, DID/VC standards, blockchain systems, and API design.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-04

---

## Findings

### TECH-001 — DID method not specified
**Severity:** Blocker
**Section:** Registration layer

The document says DIDs follow the W3C standard and are registered on a blockchain. But "DID" is a meta-standard — the actual behaviour depends entirely on the **DID method** (did:ethr, did:web, did:key, did:ebsi, etc.). Each method has fundamentally different properties:

- **did:ethr** — Ethereum-anchored, supports key rotation via smart contract, gas costs per operation
- **did:web** — DNS-based resolution, centralised trust, no blockchain needed
- **did:key** — Ephemeral, no on-chain state, no rotation
- **did:ebsi** — EBSI-specific, permissioned, eIDAS-aligned

The choice of DID method determines: key rotation capability, resolution mechanism, cost model, interoperability with eIDAS wallets, and post-quantum migration path. The document cannot be specified without this decision.

The roadmap says "Ethereum Sepolia testnet" for PoC and "L2 or EBSI" for production — these may require different DID methods entirely. did:ethr on Sepolia does not translate to did:ebsi on EBSI without a migration.

**Recommendation:** Select and justify the DID method for each phase. Define the migration path between PoC and production DID methods. Consider did:ethr for PoC and plan a dual-support period if EBSI is targeted for production.

---

### TECH-002 — Credential format not specified
**Severity:** Blocker
**Section:** Verification layer

The document describes verifiable credentials extensively (verification, identity, affiliation, logo) but never specifies the credential format:

- **W3C Verifiable Credentials Data Model 2.0** (JSON-LD) — the current W3C standard, verbose but interoperable
- **SD-JWT VC** — selective disclosure JWT credentials, adopted by IETF and the EU Digital Identity Wallet (EUDI) Architecture Reference Framework
- **AnonCreds** — Hyperledger-native, strong ZK support, poor interoperability outside Hyperledger
- **mdoc/mDL** — ISO 18013-5, used for mobile driving licences, adopted by some eIDAS implementations

The EU Digital Identity Wallet ARF has converged on **SD-JWT VC** as the primary format. If TARIDE claims eIDAS 2.0 alignment, this is likely the required choice. But SD-JWT VC has different properties from JSON-LD VCs (no linked data, different signature schemes, different selective disclosure mechanisms).

This choice affects: selective disclosure capabilities (critical for pseudonymity), signature algorithms, revocation mechanisms, wallet interoperability, and the ZK-proof roadmap.

**Recommendation:** Select SD-JWT VC as the primary format for eIDAS alignment. Define how each credential type (verification, identity, affiliation, logo) maps to the chosen format. Specify the credential schema for at least the verification credential.

---

### TECH-003 — Credential revocation mechanism not specified
**Severity:** Blocker
**Section:** Verification layer, revocation and enforcement

The document says "credentials can be revoked at any time" and "revocation propagates through the network in real time." But it never specifies the revocation mechanism:

- **Credential status list** (W3C Bitstring Status List) — compact, efficient, requires periodic polling
- **On-chain revocation registry** — immediate, but gas costs per revocation, publicly visible
- **OCSP-like real-time check** — immediate, but requires the issuer to be online
- **Short-lived credentials with frequent re-issuance** — avoids revocation entirely, but adds issuance load

"Real-time" revocation propagation and the claim that "all connected applications immediately see that the instance is no longer verified" is a strong architectural commitment. Bitstring Status List has polling delays. On-chain revocation is immediate but expensive. The choice fundamentally affects cache node design, freshness guarantees, and the 500ms lookup target.

**Recommendation:** Specify the revocation mechanism. Evaluate trade-offs between immediacy (on-chain or OCSP-like) and efficiency (status lists). Define the maximum revocation propagation delay as a protocol parameter.

---

### TECH-004 — On-chain vs. off-chain boundary is inconsistent
**Severity:** Major
**Section:** Throughout

The document makes contradictory or ambiguous statements about what lives where:

| Data | Location per document | Concern |
|---|---|---|
| DIDs | On-chain | Clear |
| Credential hashes | On-chain ("cryptographic commitment") | Clear |
| Credentials themselves | "On-chain (credential)" in use case tables | Contradicts the "only commitments on-chain" claim in GDPR section |
| Instance-DID associations | "Off-chain in the resolver network" (resolver layer section) | But cryptographic commitments on-chain |
| Reputation scores | Off-chain, with periodic hash on-chain | Clear |
| Consent preferences | Off-chain in resolver network | Clear |

The use case tables repeatedly say "On-chain (credential)" and "On-chain (identity credential)" — suggesting full credentials are stored on-chain. But the GDPR section says only "cryptographic commitments" are on-chain. These cannot both be true. If full credentials are on-chain, the privacy design is fundamentally broken. If only commitments are on-chain, the use case tables are misleading.

**Recommendation:** Create a definitive data location matrix. Correct the use case tables if credentials are only committed (not stored) on-chain. Be precise: "on-chain" should mean the actual data is on the ledger; "anchored on-chain" should mean only a commitment/hash is on-chain.

---

### TECH-005 — Resolver network architecture is a sketch, not a design
**Severity:** Major
**Section:** Resolver layer

The document says the resolver follows "the DNS model" but provides no architectural detail:

1. **Data replication** — how do cache nodes get data from attestation providers? Push? Pull? Pub-sub? What protocol?
2. **Consistency model** — is it eventually consistent? How long can a cache node serve stale data? What if two cache nodes disagree?
3. **Cache invalidation** — the hardest problem in distributed systems. How are revocations propagated? Expiry-based? Event-driven? Both?
4. **Query routing** — how does an application discover and select a cache node? Static configuration? DNS-like hierarchy? DHT?
5. **Data partitioning** — does every cache node hold all data, or is data sharded by region/number range?
6. **API specification** — what does the query/response look like? REST? gRPC? GraphQL? What's the schema?

The DNS analogy is useful for communicating the concept, but DNS itself is a specific protocol (RFC 1035, DNSSEC RFC 4033-4035) with decades of specification. The TARIDE resolver is currently just a metaphor.

**Recommendation:** Define the resolver network protocol at sufficient detail for independent implementation: data flow between attestation providers and cache nodes, consistency guarantees, cache invalidation strategy, query routing, and API schema.

---

### TECH-006 — Blockchain selection has major architectural implications that are deferred
**Severity:** Major
**Section:** Development roadmap, incentives

The document lists three options: Ethereum Sepolia (PoC), L2 (Arbitrum/Optimism), and EBSI. These have radically different properties:

| Property | Public L2 | EBSI |
|---|---|---|
| Permissioning | Permissionless | Permissioned (EU member states) |
| Gas costs | Paid by users/Foundation | Free |
| Finality | Probabilistic (~minutes) | Deterministic |
| Privacy | Fully public | Permissioned access |
| Governance | Decentralised | EU-governed |
| Post-quantum | Not yet | Potentially aligned with EU requirements |

The choice affects: cost model, privacy (public chain = all data publicly queryable), governance alignment, and eIDAS compatibility. EBSI's conformance requirements may dictate specific DID methods (did:ebsi) and credential formats that differ from the PoC choices.

Deferring this to Phase 3 means the PoC and pilot are built on architectural assumptions that may not hold for production. A migration from public L2 to EBSI is not a deployment decision — it's a redesign.

**Recommendation:** Make a provisional blockchain selection for production now, even if it changes. Design the PoC abstraction layer to minimise migration cost. At minimum, define the interface between the protocol and the ledger so the ledger is swappable.

---

### TECH-007 — 500ms lookup target is unvalidated
**Severity:** Major
**Section:** Resolver layer, executive summary

The "sub-500ms" lookup target is stated as a requirement for incoming call scenarios. But:

1. **Is 500ms actually sufficient?** In telephony, the interval between network signalling and the phone ringing varies by network. If the lookup must complete before the first ring, the budget may be tighter (200-300ms including network overhead).
2. **What's in the 500ms?** App-to-cache-node RTT + cache-node processing + any on-chain verification. If the cache node needs to verify against the blockchain, add blockchain query latency.
3. **Cache miss scenario** — if the trust profile isn't cached, the cache node must fetch from the attestation provider and potentially verify on-chain. This will exceed 500ms.
4. **Geographic constraints** — a cache node in Amsterdam serving a user in Spain adds 30-50ms RTT. At EU scale, where must cache nodes be deployed?

**Recommendation:** Break down the 500ms budget into component latencies. Define cache hit vs. cache miss behaviour. Specify whether the 500ms is a P50, P95, or P99 target. Define the degraded experience when the target is missed.

---

### TECH-008 — Credential metadata schema undefined
**Severity:** Major
**Section:** Prepaid numbers and credential metadata, verification layer

The document lists metadata fields (registration_type, account_age, identification_level, last_sim_swap, last_number_reassignment, last_time_active) but provides no schema:

1. What are the exact field names, types, and allowed values?
2. Which fields are mandatory vs. optional?
3. What happens when an attestation provider doesn't support a field? (Not all telecoms expose SIM swap data)
4. How are fields versioned when new metadata is added?
5. Is there a mechanism for attestation providers to declare which fields they support?

Without a schema, interoperability between attestation providers is impossible. One telecom reports account_age as a timestamp, another as an integer in days, a third omits it entirely.

**Recommendation:** Define a credential metadata schema with field names, types, cardinality, and mandatory/optional classification. Version the schema from day one.

---

### TECH-009 — Organisation affiliation creates a credential dependency chain
**Severity:** Major
**Section:** Organisation affiliation

The affiliation credential creates a three-layer dependency:
1. Employee DID must have a verification credential (from telecom)
2. Organisation DID must have an identity credential (from KvK)
3. Organisation DID issues affiliation credential to employee DID

If any link breaks (telecom revokes employee credential, KvK revokes organisation credential), what happens to the affiliation display? The document says the organisation "must hold a valid identity credential" to issue affiliations — but is this checked at issuance time only, or continuously? If the organisation's KvK credential expires, are all outstanding affiliation credentials invalidated?

**Recommendation:** Define the credential dependency graph and the propagation rules for revocation/expiration across dependent credentials. Specify whether dependencies are checked at issuance, at query time, or both.

---

### TECH-010 — No protocol versioning or extension mechanism
**Severity:** Major
**Section:** Throughout

The document describes a protocol that will evolve over four phases across five years. But there is no discussion of:

1. Protocol version negotiation between participants
2. Backwards compatibility requirements
3. Extension points for new credential types, metadata fields, or consent levels
4. How attestation providers and cache nodes handle protocol upgrades
5. How applications detect and adapt to new protocol features

Adding email and messaging channels later is described as architectural ("the architecture supports extension") but without a concrete extension mechanism.

**Recommendation:** Define a protocol versioning scheme and extension mechanism. Specify how new credential types and metadata fields can be added without breaking existing participants.

---

### TECH-011 — EU Digital Identity Wallet integration is hand-waved
**Severity:** Minor
**Section:** Registration layer, European positioning

"When the EU Digital Identity Wallet becomes available, it can serve the same role, making the DID portable across applications." The EUDI Wallet has specific architectural requirements (ARF v1.4):

1. PID (Person Identification Data) and QEAA issuance flows
2. SD-JWT VC as credential format
3. OpenID4VP for credential presentation
4. OpenID4VCI for credential issuance
5. Specific trust frameworks for attestation providers (QTSP requirements)

The TARIDE protocol must be designed to be compatible with these flows, or wallet integration will require a protocol redesign. "Can serve the same role" is insufficient — the specification must define the integration points.

**Recommendation:** Map TARIDE protocol flows to EUDI Wallet ARF interfaces. Identify gaps where TARIDE's architecture doesn't align with wallet requirements. Design the PoC with wallet integration points even if the wallet isn't available yet.

---

### TECH-012 — Reputation aggregation algorithm is unspecified
**Severity:** Minor
**Section:** Reputation layer

"The aggregation algorithm weights recent feedback more heavily and includes safeguards against coordinated manipulation." This is a description of desired properties, not an algorithm. Key unknowns:

1. What is the scoring function? (Weighted average? Bayesian? Decay function?)
2. What time window defines "recent"?
3. What constitutes "coordinated manipulation" and how is it detected?
4. How are scores normalised across applications with different feedback volumes?
5. Is the algorithm public (auditable) or proprietary?

**Recommendation:** Specify the reputation algorithm or at minimum its properties (input, output, bounds, update frequency) and manipulation resistance requirements. For an open protocol, the algorithm should be open and auditable.

---

### TECH-013 — Verified logo delivery has performance and security implications
**Severity:** Minor
**Section:** Verified logo

The verified logo is "delivered alongside other trust data during a lookup." Logos are images (SVG, PNG), typically kilobytes to tens of kilobytes. In a sub-500ms lookup for an incoming call:

1. Logo delivery adds payload size — a 10KB logo over mobile network adds latency
2. Logos are an injection vector — SVG can contain JavaScript, PNG can exploit image parsers
3. Logo caching strategy is unspecified — how often does the logo change? Can cache nodes serve stale logos?
4. Logo size limits are unspecified

**Recommendation:** Specify logo format constraints (dimensions, file size, allowed formats), delivery mechanism (inline vs. URL reference), caching policy, and security sanitisation requirements.

---

## Open Questions

1. **Which DID method?** (See TECH-001) This is the most consequential technical decision in the protocol.
2. **What is the credential presentation flow?** The document describes issuance and revocation but not how a credential is actually presented and verified in a lookup. Is the full credential returned by the cache node? A proof? A status?
3. **How does the protocol handle offline scenarios?** If the user's device has no connectivity, can cached trust profiles be used? For how long?
4. **What is the smart contract architecture?** How many contracts? What functions? Upgrade mechanism?
5. **How is the resolver network bootstrapped?** In the PoC, the Foundation runs everything. How does the transition to independent operators work technically?
6. **What is the data model for the resolver network?** What does a "trust profile" look like as a data structure?
7. **How does multi-channel identity work in practice?** A DID links a phone number, an email, and a WhatsApp handle. When a lookup comes in for the phone number, does the response include the email and WhatsApp credentials? That would be a privacy leak.
8. **What happens during a blockchain reorg?** On L2s, reorgs are possible. If a DID registration is included in a block that gets reorganised, the DID temporarily exists and then doesn't.
9. **How large is the on-chain storage footprint per DID?** At 10,000+ DIDs (2031 target) this may be trivial, but at pan-European scale (hundreds of millions), gas costs and storage become significant.

---

## Spec Readiness Verdict: Technical Architecture

**Not ready for specification.**

Three blockers must be resolved:
- **TECH-001**: DID method must be selected — every other technical decision depends on this
- **TECH-002**: Credential format must be selected — affects signature schemes, selective disclosure, wallet compatibility
- **TECH-003**: Revocation mechanism must be specified — "real-time revocation" is a core claim that needs a concrete implementation

Seven major findings require design decisions:
- TECH-004 (on-chain/off-chain boundary), TECH-005 (resolver architecture), TECH-006 (blockchain selection), TECH-007 (500ms validation), TECH-008 (metadata schema), TECH-009 (credential dependencies), TECH-010 (versioning)

The document's five-layer architecture is well-conceived and the separation of concerns (registration, verification, reputation, consent, resolution) is sound. The gap is that the document describes architectural intent rather than architectural design. For a foundation document, that's appropriate — for a specification, every layer needs concrete protocol definitions. The path from here to spec is clear but substantial.
