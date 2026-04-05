# Review 1: Privacy & Data Protection

**Reviewer perspective:** Senior Data Protection Officer / Privacy Officer with deep GDPR, eIDAS 2.0, and EU digital identity expertise.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-04

---

## Findings

### PRIV-001 — Legal basis not specified
**Severity:** Blocker
**Section:** Data protection and GDPR

The document identifies the Foundation as a (joint) controller and maps GDPR obligations, but never states the **legal basis** for processing under Article 6(1). Is it consent (6(1)(a))? Legitimate interest (6(1)(f))? Public interest (6(1)(e))? This is not a detail — it determines the entire compliance architecture. Legitimate interest requires a balancing test and LIA documentation. Consent requires opt-in mechanics and withdrawal rights that affect the protocol design. Public interest requires a legal mandate or public task designation.

The GDPR governance matrix has a row for "legal basis determination" but simply says "Defines protocol-level legal basis" without defining it.

**Recommendation:** Specify the legal basis per processing activity (DID registration, credential issuance, reputation aggregation, consent storage, lookups). Different activities may have different bases. Document the reasoning in the DPIA.

---

### PRIV-002 — Right to erasure vs. blockchain immutability insufficiently resolved
**Severity:** Blocker
**Section:** Data protection and GDPR, right to erasure

The document acknowledges the tension and proposes storing "only cryptographic commitments on-chain" while keeping linkable data off-chain. It claims that "on-chain hashes become meaningless once the off-chain data they reference is removed." This is a common argument but it has **not been tested by a supervisory authority or court** in the context of a system that deliberately maintains the hash as a verifiable anchor.

Key problems:
1. The hash is a **fingerprint** of the data. Under CNIL guidance (2024) and EDPB opinions, a hash derived from personal data can itself constitute personal data if re-identification is possible by any party. If the off-chain data is deleted but any party (attestation provider, cache node, application) retains a copy, the on-chain hash becomes re-linkable.
2. The document does not describe a **deletion propagation mechanism** — how does the Foundation ensure all off-chain copies across attestation providers, cache nodes, and applications are actually purged?
3. DID age is described as "immutable" and "recorded on-chain at the moment of registration." If a data subject exercises erasure, the DID creation timestamp remains on-chain forever. Is an orphaned timestamp personal data? It depends on whether anyone can re-link it.

**Recommendation:** Commission a dedicated legal analysis of the erasure/blockchain tension before the pilot. Define a concrete deletion protocol that covers all ecosystem participants. Consider whether the DPIA should explicitly conclude that the residual risk of on-chain hashes is acceptable under Article 35(7)(d), or whether additional safeguards (e.g., key rotation, commitment scheme redesign) are needed.

---

### PRIV-003 — Cache node operators classified as processors — questionable
**Severity:** Major
**Section:** GDPR governance responsibility matrix

Cache nodes are classified as processors (Article 28). But cache nodes "replicate data from attestation providers and verify it against on-chain commitments" and serve trust profiles to any connected application. If cache node operators have any discretion over what data to cache, how long to retain it, or how to respond to queries, they may be **independent controllers or joint controllers**, not processors. The EDPB has consistently held that a party exercising discretion over means of processing is a controller, not a processor.

If cache nodes are independently operated (the document says "any party meeting the technical and governance requirements can operate a cache node"), they have operational discretion. The DNS analogy in the document actually undermines the processor classification — DNS resolvers are generally considered independent controllers.

**Recommendation:** Re-evaluate the controller/processor classification for cache node operators. If they exercise operational discretion, classify them as joint controllers and extend the Article 26 arrangement accordingly.

---

### PRIV-004 — Reputation layer creates pseudonymous profiling
**Severity:** Major
**Section:** Reputation layer

Aggregated reputation scores per DID-instance combination constitute **profiling** under GDPR Article 4(4): "any form of automated processing of personal data consisting of the use of personal data to evaluate certain personal aspects." Even though the document says "no reputation data is collected about individuals," a reputation score attached to a DID that is linked to a phone number that is linked to a natural person **is** a profile of that person.

The document states reputation only applies to DIDs with identity credentials (businesses), but the architecture does not enforce this at the protocol level — it's described as a policy choice. What prevents a future governance decision from extending reputation to pseudonymous DIDs?

**Recommendation:** 
1. Explicitly address profiling under Article 22 (automated decision-making) — does the reputation score lead to decisions that "significantly affect" data subjects (e.g., being filtered out by consent rules)?
2. Build the business-only restriction into the protocol specification, not just policy.
3. Document the safeguards required under Article 22(3) if profiling is acknowledged.

---

### PRIV-005 — Consent layer is not GDPR consent
**Severity:** Major
**Section:** Consent layer

The document uses the word "consent" for two fundamentally different concepts:
1. **Protocol consent** — the recipient's preference for who can reach them (open, verified, established, trusted, contacts only).
2. **GDPR consent** — a legal basis for data processing under Article 6(1)(a), which requires freely given, specific, informed, and unambiguous indication of wishes.

These are completely unrelated. The document never clarifies this distinction. A reader (or regulator) could conflate the two and conclude that the protocol's "consent layer" satisfies GDPR consent requirements — it does not.

If protocol consent preferences are stored in the resolver network as part of the DID profile, that storage itself requires a legal basis. The preferences reveal information about the user (e.g., "contacts only" implies the user is privacy-conscious or has experienced harassment).

**Recommendation:** Rename the protocol concept to avoid confusion (e.g., "reachability preferences," "contact policy," "access rules"). If that is too disruptive, add an explicit disclaimer that protocol consent is unrelated to GDPR consent. Address the legal basis for storing and serving these preferences.

---

### PRIV-006 — Credential metadata reveals sensitive information without purpose limitation
**Severity:** Major
**Section:** Prepaid numbers and credential metadata

The protocol allows attestation providers to include: registration type (prepaid/subscription), account age, identification level, last SIM swap timestamp, and last number reassignment date. This metadata, attached to a pseudonymous DID, constitutes personal data that reveals:
- Financial status indicators (prepaid vs. contract)
- Duration of customer relationship
- Whether someone has been identity-verified
- Security events (SIM swap)

The document says "the protocol delivers this context, it does not judge" and leaves application-layer interpretation open. But under GDPR, the **purpose** of processing must be specified at collection, not delegated downstream. What is the lawful purpose for exposing "registration type: prepaid" to any querying application? The document itself acknowledges potential stigma ("there are many legitimate prepaid users").

**Recommendation:** Apply data minimisation (Article 5(1)(c)). Define which metadata fields are necessary for which purposes. Consider whether some fields should be tiered — available only to specific application categories or consent levels, not broadcast to every lookup.

---

### PRIV-007 — No data retention policy
**Severity:** Major
**Section:** Throughout

The document does not specify retention periods for any data category:
- How long are DID-instance associations retained after deactivation?
- How long are revoked credentials visible in the resolver network?
- How long is reputation data retained after a DID is deactivated?
- How long do cache nodes retain cached trust profiles?
- How long does the Foundation retain application registration data?

GDPR Article 5(1)(e) requires storage limitation — data kept "no longer than is necessary for the purposes for which the personal data are processed."

**Recommendation:** Define retention periods per data category. Include in the specification.

---

### PRIV-008 — Transparency to data subjects is underspecified
**Severity:** Major
**Section:** Data protection and GDPR

The document acknowledges the need for privacy notices (Articles 13-14) and says the Foundation "publishes protocol-level notice." But:
1. Who informs the **caller** that their trust profile is being looked up by the recipient's app? The caller is a data subject whose data is being processed (DID, credentials, reputation score served to a third party) — Article 14 requires information to be provided even when data is not collected directly from the data subject.
2. How is the privacy notice delivered in a sub-500ms lookup scenario? The caller cannot be informed before the lookup happens.
3. The GDPR governance matrix assigns transparency obligations to multiple parties. Without a concrete coordination mechanism, data subjects will be bounced between entities.

**Recommendation:** Design a transparency architecture. Consider a protocol-level transparency page (accessible via DID) that explains to any data subject what data is held, who can access it, and how. Define which party is the first point of contact for data subject requests.

---

### PRIV-009 — Cross-border transfer mechanism absent
**Severity:** Major
**Section:** International applicability

The document mentions international expansion ("the UK GDPR, Brazil's LGPD, South Africa's POPIA") and pan-European scale by 2029-2031. But there is no mention of:
- Transfer mechanisms for personal data leaving the EU (Chapter V GDPR)
- Whether cache nodes can operate outside the EU
- Whether attestation providers in non-adequate countries can participate
- Standard contractual clauses or adequacy decisions

If a cache node in Brazil serves a trust profile containing a Dutch phone number's DID data, that is a cross-border transfer.

**Recommendation:** Specify that, at minimum, cache nodes and attestation providers must operate within the EU/EEA or in countries with adequacy decisions, unless appropriate safeguards (SCCs, BCRs) are in place. Include this in the specification.

---

### PRIV-010 — Silent re-authentication and continuous validation raise proportionality concerns
**Severity:** Major
**Section:** Continuous validation

"Each time the credential is presented or a communication occurs, the application can verify in real time with the mobile network that the number is still active and still associated with the same SIM card." This constitutes real-time location/presence tracking capability. Even if no location data is returned, the ability to ping a mobile network to confirm SIM presence is a surveillance-capable feature.

The document positions this as "optional" and for "higher-assurance scenarios." But the specification must address:
- Who can trigger continuous validation? Only the DID holder's own app, or any querying party?
- Is the DID holder informed each time their SIM is re-authenticated?
- What prevents an abusive application from using continuous validation to track whether someone's phone is active?

**Recommendation:** Restrict continuous validation to self-initiated checks (the DID holder's own app verifying its own SIM). Require explicit opt-in. Document the proportionality assessment.

---

### PRIV-011 — Organisation affiliation credential creates employment surveillance risk
**Severity:** Minor
**Section:** Organisation affiliation

The organisation affiliation credential, issued by employer to employee, is "revocable at any time by the issuing organisation." The document says "for organisations with large numbers of employees, the protocol supports automated credential management." This creates an infrastructure for real-time employment status tracking: any party querying a DID can see whether the affiliation is active, and the moment it is revoked, the change is visible across the network "in real time."

While the use case is legitimate, the privacy impact is significant — a termination becomes visible to any querying party before the employee may have even been informed, depending on the revocation timing.

**Recommendation:** Add a grace period or notification requirement before affiliation revocation becomes visible. Ensure the employee (DID holder) is notified before or simultaneously with revocation propagation.

---

### PRIV-012 — DPIA timing is too late
**Severity:** Minor
**Section:** Data protection and GDPR, compliance obligations

The document says the DPIA "should be conducted during the pilot phase." Under Article 35, a DPIA must be conducted **before** processing begins. The pilot phase **is** processing. The DPIA should be conducted before the proof of concept goes live with real personal data.

**Recommendation:** Conduct the DPIA before the proof of concept phase if real phone numbers or real subscriber data are involved. Update the roadmap accordingly.

---

### PRIV-013 — Lookup fee data creates a commercial surveillance dataset
**Severity:** Minor
**Section:** Foundation revenue model, application registration

Lookup fees are charged per query, tracked per application UUID, with quality scoring based on "consistency, volume, and absence of manipulation signals." This tracking inherently creates a dataset of who looked up whom, when, and from which application. This is communications metadata — potentially more sensitive than the trust profiles themselves.

The document does not address:
- Whether lookup logs are retained and for how long
- Who has access to lookup metadata
- Whether lookup patterns could be used to infer relationships or surveillance interests

**Recommendation:** Apply data minimisation to lookup logs. Define retention limits. Consider privacy-preserving billing (aggregate counts rather than per-query logging). Address in the DPIA.

---

## Open Questions

The document should answer these before specification:

1. **What is the legal basis per processing activity?** (See PRIV-001)
2. **Can a DID holder see their own complete trust profile as served to querying parties?** Article 15 implies yes, but the mechanism is unspecified.
3. **What happens to reputation data when a DID is deactivated?** Is it deleted, anonymised, or retained?
4. **Who is the lead supervisory authority?** The Foundation is Dutch, but attestation providers may be in other EU countries. Under Article 56, the lead SA depends on the main establishment.
5. **How are data subject requests routed in practice?** If a Dutch citizen asks KPN to delete their TARIDE data, how does KPN coordinate with the Foundation, cache nodes, and applications?
6. **What is the legal classification of DID age?** If it is personal data (because it can be linked to a natural person via the attestation provider), the claim that it is "immutable" conflicts with erasure rights.
7. **Are there special category data implications?** Could trust profiles or reputation scores infer sensitive data (e.g., a healthcare credential revealing health status, a religious organisation affiliation)?
8. **What role does the eIDAS 2.0 wallet play in data protection?** If the wallet becomes a DID management tool, does the wallet provider become a controller?
9. **Is the "one instance, one DID" constraint privacy-limiting?** It prevents someone from maintaining separate pseudonymous identities for the same phone number — is this justified and proportionate?

---

## Spec Readiness Verdict: Privacy & Data Protection

**Not ready for specification.**

Two blockers must be resolved first:
- **PRIV-001**: Legal basis must be defined per processing activity
- **PRIV-002**: Right to erasure vs. blockchain must be resolved with a concrete architectural answer and legal opinion

Six major findings require resolution or explicit design decisions before the privacy aspects can be specified:
- PRIV-003 (cache node classification), PRIV-004 (profiling), PRIV-005 (consent terminology), PRIV-006 (metadata purpose limitation), PRIV-007 (retention), PRIV-008 (transparency), PRIV-009 (cross-border), PRIV-010 (continuous validation)

The v0.51 update significantly improved the GDPR section compared to earlier versions (the pseudonymity correction, the IAB Europe precedent, the governance matrix are all strong). But the document currently describes **what GDPR requires** more than it describes **how the protocol satisfies those requirements**. The specification needs the latter.
