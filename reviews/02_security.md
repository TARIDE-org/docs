# Review 2: Security

**Reviewer perspective:** Senior security architect with expertise in cryptographic protocols, decentralised systems, threat modelling, and telecom security.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-04

---

## Findings

### SEC-001 — No threat model
**Severity:** Blocker
**Section:** Throughout

The document describes defences (time-as-trust, one-instance-one-DID, SIM swap mitigation) but never defines the threat model they defend against. A foundation document heading toward specification must enumerate:

1. **Adversary classes** — script-kiddie spammer, organised crime (SIM swap farms), state-level actor, rogue attestation provider, compromised cache node, malicious application developer, insider at the Foundation.
2. **Adversary capabilities** — per class, what can they control? Keys, SIM cards, telecom insider access, blockchain write access, cache node operation?
3. **Security goals** — what properties must hold? (Authenticity, integrity, pseudonymity, non-repudiation, availability)
4. **Trust assumptions** — what must be trusted and what happens when that trust breaks?

Without this, every security claim in the document is an assertion without a reference frame. "Time is the one resource that cannot be manufactured, purchased, or stolen" — a state actor with telecom insider access can backdate DID registrations on a compromised provider. Is that in scope?

**Recommendation:** Write a formal threat model (STRIDE, attack trees, or similar) before specification. It should be a standalone document referenced by the foundation document.

---

### SEC-002 — Private key management is critically underspecified
**Severity:** Blocker
**Section:** Registration layer

"The private key remains solely with the holder" and "private key stored in secure enclave." This is the single most important security property of the entire protocol — the DID holder's sovereignty depends entirely on key security — yet the document provides no detail on:

1. **Key storage** — "secure enclave" is mentioned for Calmido, but the protocol is open to any application. What minimum key storage requirements does the specification impose? Software keystore? TEE? HSM? What about desktop apps or web apps without secure enclaves?
2. **Key recovery** — what happens when a user loses their phone? The DID is self-sovereign and the private key is device-local. No recovery mechanism is described. Loss of key = loss of DID = loss of all accumulated trust and reputation. This will happen to thousands of users.
3. **Key rotation** — can a DID holder rotate their key pair without creating a new DID? If not, key compromise means permanent DID abandonment. If yes, how is key rotation authenticated?
4. **Multi-device** — users have phones, tablets, laptops. Can a DID be used from multiple devices? How are keys synchronised?
5. **Key compromise response** — if a private key is stolen (malware, physical access), what is the revocation and recovery procedure?

The EU Digital Identity Wallet is mentioned as a future key management option, but that's 2026-2027 at earliest. The PoC needs a concrete answer now.

**Recommendation:** Define key lifecycle requirements (generation, storage, rotation, recovery, revocation) as protocol-level mandates with minimum security levels for compliant applications. This is foundational — every other security property depends on it.

---

### SEC-003 — Attestation provider compromise is an existential risk with no mitigation
**Severity:** Blocker
**Section:** Verification layer, resolver layer

The attestation provider (telecom) is the root of trust for instance verification. The document states: "The telecom provider occupies a unique position in this architecture. It is the only party that can confirm that a number is registered to the party claiming it."

If an attestation provider is compromised (as Odido was in the document's own problem statement):
1. The attacker can issue fraudulent verification credentials for any number in the provider's range
2. The attacker can revoke legitimate credentials, effectively de-verifying real users
3. The attacker can associate numbers with attacker-controlled DIDs
4. The attacker can manipulate credential metadata (account age, SIM swap timestamps)

The document's attestation provider registry validates "that an attestation provider is authorised to issue credentials for a given instance" — but this only prevents cross-range fraud, not fraud within the provider's own range. The Odido breach exposed 6.2 million records. A compromised attestation provider could corrupt 6.2 million trust profiles.

**Recommendation:** Define:
- Multi-signature or threshold schemes for credential issuance (no single compromised system can issue credentials)
- Anomaly detection for bulk credential operations (mass issuance/revocation triggers alerts)
- Credential issuance rate limits per attestation provider
- An incident response protocol for attestation provider compromise
- Whether cache nodes can detect and quarantine suspicious credential changes

---

### SEC-004 — On-chain data provides a correlation oracle
**Severity:** Major
**Section:** Registration layer, reputation layer

The DID is registered on-chain with a public key and a creation timestamp. Credential hashes are on-chain. Reputation commitment hashes are on-chain. This creates a public, permanent, queryable dataset that:

1. Links DID creation events to specific blockchain blocks (timestamps)
2. Shows credential issuance/revocation events over time
3. Reveals the pattern of a DID's lifecycle (creation, first credential, additional credentials, revocations)

Even though phone numbers and personal data are off-chain, the on-chain activity pattern is a **fingerprint**. If an observer knows approximately when someone registered (e.g., "my colleague signed up last Tuesday"), they can correlate that with DID creation timestamps. At low adoption, this is highly identifying.

**Recommendation:** Evaluate privacy-preserving on-chain designs:
- Batch DID registrations (aggregate multiple registrations into a single transaction to reduce timing precision)
- Delayed on-chain anchoring (register off-chain immediately, anchor on-chain periodically)
- Evaluate whether ZK-proofs (mentioned for Phase 2) should be brought forward for on-chain operations

---

### SEC-005 — Cache node integrity — no authentication of queries or responses
**Severity:** Major
**Section:** Resolver layer

Cache nodes serve trust profiles to applications. The document describes verification against on-chain commitments but does not specify:

1. **How does an application authenticate a cache node?** Without mutual authentication, a man-in-the-middle could impersonate a cache node and serve false trust profiles ("this scam number is verified, high trust").
2. **How does an application verify the response?** The document says cache nodes "verify data against on-chain commitments" — but does the querying application also verify, or does it trust the cache node?
3. **What prevents a malicious cache node operator from serving manipulated data?** Cache nodes can be operated by "any party meeting the technical and governance requirements." A malicious operator could selectively downgrade trust profiles of competitors or upgrade profiles of allies.

The DNS analogy is instructive here — DNS is notoriously vulnerable to spoofing and poisoning, which is why DNSSEC exists. The TARIDE equivalent is unspecified.

**Recommendation:** Specify:
- Authenticated responses (cache nodes cryptographically sign responses, applications verify)
- Application-side verification against on-chain commitments (don't trust the cache node blindly)
- A cache node audit mechanism (the Foundation or other parties can verify cache node correctness)

---

### SEC-006 — "Cryptographic agility" is stated but not designed
**Severity:** Major
**Section:** Registration layer

"The protocol is designed with cryptographic agility, enabling migration to post-quantum signature schemes as these standards mature." This is a single sentence with no architectural backing. Cryptographic agility in a decentralised protocol with on-chain data is exceptionally difficult because:

1. Existing DIDs have public keys in a specific algorithm. Migration requires either re-registering all DIDs or supporting multiple algorithms simultaneously.
2. Verifiable credentials signed with pre-quantum algorithms become retroactively forgeable once quantum computers can break them ("harvest now, decrypt later" attack).
3. The blockchain itself must support the new signature schemes.

NIST finalised post-quantum standards (ML-KEM, ML-DSA, SLH-DSA) in 2024. The document should at least acknowledge the migration path.

**Recommendation:** Define a key migration strategy at the protocol level. Specify which cryptographic algorithms are used in the PoC. Identify the post-quantum migration trigger and mechanism. Address "harvest now, verify later" risk for credentials with long-term value.

---

### SEC-007 — Application UUID as single enforcement point is fragile
**Severity:** Major
**Section:** Application registration

The UUID is the sole mechanism for application identification, rate limiting, abuse prevention, and enforcement. The document states: "the foundation can revoke its UUID. Revocation is immediate."

Problems:
1. **UUID theft/replay** — how is the UUID authenticated? If it's just included in API calls, it can be extracted from a compromised app and used by an impersonator.
2. **Sybil applications** — what prevents a malicious developer from registering multiple applications under different business identities to evade UUID-level enforcement?
3. **Revocation circumvention** — after UUID revocation, the developer registers a new entity and gets a new UUID. The document's "lightweight verification model" ("no waiting period," "any legitimate party can register within hours") makes this trivial.
4. **No application integrity verification** — the protocol cannot verify that the application actually does what it claims (e.g., respects consent preferences, doesn't leak trust profiles).

**Recommendation:** 
- Authenticate UUIDs with API keys or mutual TLS, not just identifier inclusion
- Define rate limits and monitoring for new application registrations
- Consider a probationary period for new applications (limited lookup volume, no reputation submission)
- Specify what "audit right" means in practice — scope, frequency, enforcement

---

### SEC-008 — Reputation manipulation vectors not fully addressed
**Severity:** Major
**Section:** Reputation layer

The document acknowledges the risk ("fake positive reviews, coordinated downvoting") and proposes quality scoring. But several attack vectors are unaddressed:

1. **Reputation laundering** — a scammer builds a high-trust DID over months of legitimate use, then uses it for a high-value scam. The "time-as-trust" mechanism amplifies this: a patient attacker gains maximum trust.
2. **Reputation poisoning at the application level** — individual user feedback stays within the app, only aggregated scores are submitted. A compromised or malicious app can fabricate aggregated scores without any individual feedback to audit.
3. **Feedback authenticity** — how does the protocol verify that a reputation submission corresponds to an actual communication event? Can an app submit feedback for a call that never happened?
4. **Strategic timing** — a bad actor accumulates positive reputation, then rapidly executes a scam campaign during the window before reputation degrades.

**Recommendation:** Specify:
- Correlation between reputation submissions and verifiable communication events
- Rate-of-change limits on reputation scores (sharp changes trigger review)
- Cool-down periods after reputation changes before trust profile updates propagate
- Whether reputation decay over time (inactive DIDs gradually lose trust score) is part of the model

---

### SEC-009 — DID deactivation procedure undefined
**Severity:** Major
**Section:** Registration layer

If a DID is compromised, abandoned, or subject to an erasure request, it must be deactivated. The document does not describe:

1. Who can initiate deactivation? Only the private key holder? What if the key is lost?
2. How is deactivation propagated? On-chain transaction? Resolver network update?
3. What happens to linked instances? Are they released immediately (creating a window for hijacking)?
4. Can a deactivated DID be reactivated?
5. Is there a social recovery mechanism (trusted contacts can collectively deactivate a DID)?

**Recommendation:** Define the full DID deactivation lifecycle, including edge cases (lost key, court order, deceased user).

---

### SEC-010 — One-instance-one-DID enforcement depends on trusted attestation providers
**Severity:** Major
**Section:** Reputation evasion and sybil resistance

"The protocol enforces that a given instance can only be linked to one DID at a time." But this enforcement relies on attestation providers honestly reporting associations. If a compromised or colluding attestation provider issues credentials for the same instance to two different DIDs, the protocol has no independent way to detect this — the on-chain data only contains hashes, not the instance-DID mapping.

**Recommendation:** Specify a cryptographic mechanism to enforce one-instance-one-DID at the protocol level, independent of attestation provider honesty. Consider a commitment scheme where the instance identifier is committed on-chain in a way that prevents double-linking without revealing the instance.

---

### SEC-011 — Resolver network availability and DDoS resilience unaddressed
**Severity:** Major
**Section:** Resolver layer

The resolver network must deliver responses within 500ms for incoming call scenarios. This creates a hard availability requirement. The document says nothing about:

1. DDoS protection for cache nodes
2. Failover and redundancy requirements
3. What happens to the user experience when the resolver is unavailable (fail-open = no trust signals; fail-closed = all calls marked unverified)
4. Geographic distribution requirements for latency

A targeted DDoS against the resolver network would disable trust signals for all users — a high-value target for organised crime that profits from impersonation.

**Recommendation:** Define availability requirements (SLA), DDoS mitigation strategy, failover behaviour, and fail-open vs. fail-closed policy. This is a security requirement, not just an operations one.

---

### SEC-012 — Telecom-side CAMARA API trust is assumed
**Severity:** Minor
**Section:** Continuous validation

The protocol integrates CAMARA APIs (Number Verification, SIM Swap) for continuous validation. The security of these API calls is entirely assumed — the document treats CAMARA as a trusted oracle. But:

1. CAMARA APIs are accessed through operator gateways. The security of the API call (authentication, transport, integrity) depends on each operator's implementation.
2. A compromised CAMARA endpoint could return false positives ("yes, the number is verified") for attackers.
3. The document does not discuss API authentication between the TARIDE resolver and CAMARA endpoints.

**Recommendation:** Specify minimum security requirements for CAMARA API integration (mutual TLS, signed responses, error handling for unavailable APIs).

---

### SEC-013 — Smart contract security not addressed
**Severity:** Minor
**Section:** Development roadmap

The protocol uses on-chain smart contracts for DID registration, credential commitments, and reputation anchoring. Smart contract vulnerabilities (re-entrancy, logic errors, upgrade risks) have caused billions in losses in the blockchain ecosystem. The document does not mention:

1. Smart contract audit requirements
2. Upgrade mechanisms (immutable vs. upgradeable contracts)
3. Access control for contract administration
4. Emergency pause mechanisms

**Recommendation:** Require independent smart contract audits before each deployment phase. Define the upgrade and governance model for on-chain contracts.

---

### SEC-014 — No incident response framework
**Severity:** Minor
**Section:** Throughout

The document describes a complex multi-stakeholder system with attestation providers, cache nodes, applications, and a blockchain layer. No incident response framework is defined:

1. Who coordinates security incidents?
2. What is the escalation path for a compromised attestation provider, cache node, or application?
3. How are emergency credential revocations handled?
4. Is there a security advisory mechanism for ecosystem participants?

**Recommendation:** Define an incident response framework as part of the foundation governance. Include communication channels, escalation paths, and emergency procedures.

---

## Open Questions

1. **What cryptographic algorithms are specified for the PoC?** (Key generation, signatures, hashing, commitment schemes)
2. **What is the key recovery path for end users who lose their device?** This will affect millions of users at scale.
3. **Can the Foundation unilaterally deactivate a DID?** If yes, under what circumstances? If no, what happens when a court orders it?
4. **What is the security model for the blockchain layer itself?** The document says "Ethereum testnet" for PoC and "L2 or EBSI" for production. These have vastly different security properties (public vs. permissioned, validator sets, finality guarantees).
5. **How is the attestation provider registry itself secured?** If an attacker can modify the registry to claim authority over number ranges, they can issue credentials for any number.
6. **What prevents a rogue cache node from logging all queries and building a surveillance dataset?** (Cross-reference with PRIV-013)
7. **Is there a protocol-level mechanism to detect split-brain scenarios?** (Different cache nodes serving different data for the same DID)
8. **How are cryptographic commitments on-chain verified in practice?** Does every application need blockchain access, or do they trust cache nodes?

---

## Spec Readiness Verdict: Security

**Not ready for specification.**

Three blockers must be resolved first:
- **SEC-001**: A threat model must exist before security properties can be specified
- **SEC-002**: Key lifecycle (storage, recovery, rotation, compromise response) must be defined — this is the foundation of all other security claims
- **SEC-003**: Attestation provider compromise mitigation must be designed — the document's own problem statement (Odido breach) demonstrates this is a real and current threat

Eight major findings require design decisions:
- SEC-004 (on-chain correlation), SEC-005 (cache node integrity), SEC-006 (post-quantum), SEC-007 (UUID security), SEC-008 (reputation manipulation), SEC-009 (DID deactivation), SEC-010 (one-instance-one-DID enforcement), SEC-011 (availability/DDoS)

The document demonstrates good security intuition (time-as-trust is genuinely clever, the SIM swap analysis is honest about limitations). But there is a gap between describing security properties and specifying how they are achieved. The specification needs the cryptographic details, the threat model, and the failure modes.
