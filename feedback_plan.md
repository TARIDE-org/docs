# TARIDE — Feedback & Validation Plan

## Objective

Validate the foundation document (v0.51) with key stakeholders before committing to protocol specification and PoC development. Goal: identify architectural blind spots, dealbreakers, and missing requirements before they become expensive to fix.

## Feedback groups

### Tier 1 — Can block or reshape the architecture

These groups must be consulted before finalising specs. Their input directly affects protocol design.

| # | Group | Why critical | What to send |
|---|-------|-------------|-------------|
| 1 | **Telecom providers / COIN** | Only party that can issue verification credentials. Without them, no protocol. COIN is the single coordination point for all Dutch operators. | Telecom brief (3-4 pages): attestation provider role, CAMARA integration points, resolver architecture, credential lifecycle |
| 2 | **Security researchers** | Protocol must survive adversarial analysis. Sybil resistance, key management, and SIM swap defenses are untested claims. | Threat model (5-6 pages): attack surfaces, trust assumptions, cryptographic choices, what the protocol explicitly does NOT protect against |
| 3 | **Privacy / digital rights (Bits of Freedom, EDPS, Dutch DPA / AP)** | A trust protocol that violates privacy expectations is dead on arrival in Europe. Early engagement prevents costly redesigns and builds credibility. | Privacy analysis (3-4 pages): complete data flow mapping, on-chain vs off-chain, what's linkable, GDPR basis for each processing activity, comparison to Truecaller/Hiya data model |
| 4 | **DID/SSI community (DIF, W3C CCG, Rebooting the Web of Trust)** | These people have built and broken decentralised identity systems for a decade. They know which DID methods work, which credential formats survive interop, and which governance models fail. | Technical protocol summary (4-5 pages): DID method choice, credential lifecycle, resolver architecture, relationship to existing standards |

### Questions per group

Each outreach asks specific technical questions. Adoption and partnership questions are handled separately, outside the technical review process.

**Telecom providers / COIN**
- What subscriber metadata (account age, registration type, SIM swap timestamp) can be exposed through existing APIs without infrastructure changes?
- What are the technical constraints for credential issuance and revocation latency in your current systems?
- How does the proposed attestation provider model interact with existing CAMARA API implementations and COIN's shared infrastructure?
- What are the integration requirements for number portability event propagation?

**Security researchers**
- What attacks does the threat model miss?
- Where are the weakest trust assumptions?
- Is time-as-trust (DID age + association age) sufficient as the primary sybil resistance mechanism?
- What key management risks exist when private keys are stored in mobile device secure enclaves?
- What are the attack surfaces in the credential revocation propagation path?

**Privacy / digital rights**
- What privacy risks do you see in the on-chain footprint (DID registration, credential hashes)?
- Is pseudonymity-by-default credible given that DID activity patterns could be correlated across transactions?
- What is the linkability risk when a single DID holds multiple instances across channels?
- What GDPR legal basis applies to each data processing activity in the protocol?
- How does the data model compare to Truecaller/Hiya from a privacy perspective?

**DID/SSI community**
- Does `did:ethr` fit the requirements or does the protocol need a custom DID method?
- What credential format (JSON-LD, JWT, SD-JWT) best supports selective disclosure for this use case?
- What interoperability traps should we avoid given planned eIDAS 2.0 wallet integration?
- How should the resolver network handle credential status checks at scale (revocation lists vs status registries)?

### Tier 2 — Important but can follow the first round

These groups provide valuable input but won't fundamentally change the architecture. Consult after Tier 1 feedback is incorporated.

| # | Group | Why important | When to engage |
|---|-------|--------------|---------------|
| 5 | **KvK (Chamber of Commerce)** | Key credential issuer for business identity. Need to confirm they can/will issue VCs. | After telecom conversations confirm the attestation model works |
| 6 | **Legal counsel** | Foundation structure (Dutch stichting), liability for incorrect credentials, GDPR controller/processor roles | After privacy review clarifies the data flow questions |
| 7 | **Financial sector (NVB, individual banks)** | Major beneficiaries (fraud reduction) and potential early adopters with full identity credentials | After PoC demo exists — banks respond to working software, not whitepapers |
| 8 | **Consumer organisations (Consumentenbond)** | End-user perspective, plain-language validation, potential advocacy partner | After PoC demo — need something tangible to evaluate |
| 9 | **Regulators (ACM, Ministry of BZK)** | Regulatory alignment, potential policy support, eIDAS 2.0 coordination | After Tier 1 feedback is incorporated and the protocol is more concrete |
| 10 | **L2 / EBSI teams** | Chain selection affects cost, latency, and governance. Need feasibility confirmation. | After DID method and credential format are decided |

### Not yet — too early for these

| Group | Why wait |
|-------|---------|
| **General public / end-users** | Cannot evaluate a protocol spec. Wait until Calmido has a working demo. |
| **International telecoms** | Solve the Netherlands first. Cross-border adds complexity that shouldn't influence the initial design. |
| **Media** | Nothing to show yet. Premature coverage creates expectations without substance. |

## Derivative documents to produce

Each document is extracted from the foundation doc, tailored to its audience. Not new content — restructured existing content with audience-specific framing.

| Document | Audience | Pages | Key sections |
|----------|----------|-------|-------------|
| `briefs/telecom_brief.md` | Telecom / COIN | 3-4 | Problem (their perspective), attestation provider role, CAMARA integration points, revenue model, what we need from them, timeline |
| `briefs/threat_model.md` | Security researchers | 5-6 | Trust assumptions, attack taxonomy (sybil, SIM swap, credential fraud, privacy attacks, resolver attacks), mitigations, open questions |
| `briefs/privacy_analysis.md` | Privacy / digital rights | 3-4 | Complete data flow diagram, on-chain footprint analysis, linkability assessment, GDPR basis per processing activity, comparison matrix vs existing solutions |
| `briefs/technical_summary.md` | DID/SSI community | 4-5 | DID method, credential format & lifecycle, resolver architecture, standards alignment, open design questions |
| `briefs/explainer.md` | Non-technical stakeholders | 1 | Plain-language "what this means when your phone rings" — for later use |

## Feedback process

### Structure

Use this repo as the single source of truth. All feedback is tracked here — not scattered across email threads, LinkedIn messages, and meeting notes.

```
docs/
  feedback/
    tracker.md              # Central status tracker (table below)
    templates/
      feedback_request.md   # Standard outreach template
      feedback_form.md      # Structured response template
    responses/
      coin_2026-04.md       # One file per respondent
      bitsofffreedom_2026-04.md
      ...
```

### Tracker format

A single table in `tracker.md` tracks every feedback engagement:

| Stakeholder | Tier | Contact | Document sent | Date sent | Response due | Status | Key findings | Action taken |
|---|---|---|---|---|---|---|---|---|
| COIN | 1 | [name] | telecom_brief.md | 2026-04-07 | 2026-04-21 | Awaiting response | — | — |
| Bits of Freedom | 1 | [name] | privacy_analysis.md | 2026-04-07 | 2026-04-21 | — | — | — |

### Feedback request template

Each outreach follows the same structure:
1. **Context** — one paragraph on what TARIDE is (not the full vision, just enough)
2. **Why you** — specific reason this person/org's technical expertise matters
3. **The document** — attached, with reading guidance ("focus on sections 2 and 4")
4. **What we're asking** — 3-5 specific technical questions (not "what do you think?")
5. **How to respond** — structured form or annotated document, their choice
6. **Timeline** — specific date, typically 2 weeks
7. **What happens next** — how their feedback will be used, offer to share updated version

See `feedback/templates/feedback_request.md` for the full template.

### Structured feedback form

See `feedback/templates/feedback_form.md`. The form is scoped to technical assessment: what works, technical concerns, what's missing, and suggested changes.

### Principles

- **Technical questions only.** Every outreach asks specific technical questions. "Can your systems propagate credential revocation within 60 seconds?" produces actionable input. Adoption and partnership discussions are handled separately.
- **Respond to every response.** Send a brief acknowledgment within 48 hours. Share what changed as a result within 2 weeks. People who see their input reflected come back for round 2.
- **Track disagreements explicitly.** When two stakeholders contradict (e.g., "more data on-chain" vs "less data on-chain"), document both positions and the tradeoff. Don't silently pick one.
- **Two-week response windows.** Shorter gets ignored, longer loses momentum. Follow up once at the midpoint.

## Timeline

```
Week 1-2   Write derivative documents (briefs + threat model + privacy analysis)
Week 3     Internal review of derivative documents
Week 4     Send Tier 1 outreach (telecom, security, privacy, DID/SSI)
Week 5-6   Response window (follow up at midpoint)
Week 7-8   Synthesise feedback, update foundation doc to v0.6
Week 9     Send Tier 2 outreach where relevant
Week 10-12 Incorporate Tier 2 feedback
           (Track 2 — spec + PoC work runs in parallel from week 3 onward)
```

## Success criteria

Tier 1 feedback round is successful if:
- [ ] At least one telecom contact (ideally COIN) confirms the attestation provider model is technically feasible
- [ ] Security review identifies no fundamental protocol-breaking flaw (fixable issues are expected and welcome)
- [ ] Privacy review confirms pseudonymity-by-default is credible, or identifies specific changes needed
- [ ] DID/SSI community validates the standards choices or recommends concrete alternatives
- [ ] All Tier 1 feedback is documented in `feedback/responses/` and reflected in an updated foundation doc
