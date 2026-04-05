# Review 8: Supportability & Governance

**Reviewer perspective:** Senior programme director with expertise in open-source foundation operations, multi-stakeholder governance, ecosystem onboarding, and long-term maintainability of protocol infrastructure.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-05

---

## Findings

### SUP-001 — Governance structure is aspirational, not operational
**Severity:** Blocker
**Section:** Governance

The governance section is 15 lines long. For a foundation that claims multi-stakeholder governance as a core differentiator, this is insufficient. Missing:

1. **Decision-making process.** How are protocol changes proposed, debated, and approved? Rough consensus (IETF-style)? Formal voting? Weighted votes by stakeholder category? Veto rights?
2. **Board mechanics.** How many board members? Term limits? How are they selected — appointed, elected, nominated by constituency? What happens if a board member represents a conflicting interest (e.g., a telecom board member whose company is accused of non-compliance)?
3. **Protocol change governance.** Who can propose a change to the protocol specification? What is the review and approval process? How are breaking changes handled?
4. **Conflict resolution.** When stakeholders disagree (telecom wants feature X, civil society opposes it), who arbitrates? What is the appeals process?
5. **Transparency.** Are board meetings public? Are decisions published? Is there an open process for community input?

The document cites Signal Foundation, Mozilla, and Ecosia as precedents. All three have detailed, published governance structures. TARIDE has bullet points.

**Recommendation:** Draft operational governance bylaws before the PoC phase. At minimum: board composition (number, selection, terms), decision-making process (proposals, voting, quorum), conflict resolution, transparency requirements, and the protocol change process. These can be lightweight initially but must exist.

---

### SUP-002 — No onboarding path defined for any ecosystem participant
**Severity:** Blocker
**Section:** Throughout

The document describes five categories of ecosystem participants (DID holders, attestation providers, cache node operators, credential issuers, applications) but provides no onboarding specification for any of them:

**Attestation providers:**
- What technical integration is required? (API endpoints, credential issuance flow, metadata exposure)
- What legal agreements must be signed? (Joint controller arrangement, SLA commitment)
- What infrastructure must they operate? (Or can they use Foundation-hosted tools?)
- What is the testing/certification process before going live?
- What ongoing obligations do they have? (Uptime, data freshness, revocation propagation)

**Cache node operators:**
- What software do they run? (Foundation reference implementation? Custom?)
- What hardware/infrastructure requirements exist?
- How do they connect to attestation providers and the blockchain?
- What is the acceptance process for joining the network?
- What monitoring must they provide to the Foundation?

**Applications:**
- What is the API integration guide?
- What are the protocol terms of use?
- How do they test against the resolver network?
- What support is available during integration?

**Credential issuers (KvK, regulators):**
- What credential format must they issue?
- What trust framework governs their participation?
- What liability do they assume for issued credentials?

Without onboarding paths, the document is asking stakeholders to commit to something they can't evaluate technically or operationally.

**Recommendation:** Define onboarding requirements per participant type. This doesn't need to be the full specification — a "participation brief" per stakeholder type covering: prerequisites, integration steps, legal requirements, testing process, and ongoing obligations. Publish these alongside the foundation document for partnership conversations.

---

### SUP-003 — Dispute resolution and enforcement are undefined
**Severity:** Major
**Section:** Application registration, governance

The document describes enforcement mechanisms: "the foundation can revoke [a UUID]" and "the foundation retains the right to audit." But the process around these powers is absent:

1. **Due process.** What happens before revocation? Warning? Suspension? Right to respond? Timeline?
2. **Evidence standard.** What constitutes "violating protocol terms"? Who investigates? Who decides?
3. **Appeals.** Can a revoked application appeal? To whom? Within what timeframe?
4. **Proportionality.** Revocation is the nuclear option (instant disconnection). Are there graduated sanctions (warning, rate limiting, probation, suspension, revocation)?
5. **Attestation provider disputes.** What if an attestation provider issues credentials incorrectly? What if they refuse to revoke a credential for a reported scammer? Who adjudicates?
6. **Cross-stakeholder disputes.** What if an application accuses a cache node of serving stale data? What if an attestation provider disputes the Foundation's audit findings?

For an open protocol, enforcement without due process will deter participation. For a trust protocol, weak enforcement will enable abuse.

**Recommendation:** Define a graduated enforcement framework with due process: notice → investigation → response period → decision → appeal. Specify who holds each power and the checks on that power. Consider an independent compliance committee separate from the board.

---

### SUP-004 — Documentation strategy is absent
**Severity:** Major
**Section:** Throughout

The document says "the foundation develops and maintains open-source specifications and reference implementations." But there is no documentation strategy:

1. **Specification documents.** Protocol specification, API reference, credential schemas, data models — who writes these? In what format? How are they versioned?
2. **Implementation guides.** Integration guides per participant type, code examples, SDKs — are these Foundation responsibility or community-contributed?
3. **Operational documentation.** Runbooks for attestation providers, cache node operators. Incident response procedures. Monitoring guides.
4. **End-user documentation.** How does a user understand their trust profile? How do they exercise data subject rights? How do they report abuse?
5. **Governance documentation.** Published bylaws, meeting minutes, decision records, protocol change proposals.

Open-source projects live or die by documentation quality. The W3C DID specification is 150+ pages. The Verifiable Credentials specification is 100+ pages. TARIDE's spec will be equally complex. Who writes and maintains it?

**Recommendation:** Define a documentation roadmap: what documents are needed, who is responsible, what format and tooling, and when each must be ready. Budget documentation as a first-class work item, not an afterthought.

---

### SUP-005 — Foundation staffing and capability is unaddressed
**Severity:** Major
**Section:** Governance, how to get involved

The document lists what the Foundation does: develops protocol, maintains specifications, governs ecosystem, coordinates partners, handles GDPR obligations, audits applications, manages finances. This requires:

- Protocol engineers (cryptography, distributed systems)
- Legal counsel (GDPR, foundation law, telecom regulation)
- Programme management (partnerships, ecosystem coordination)
- DevOps/SRE (infrastructure, PoC operations)
- Community management (open-source, documentation)

The document names two individuals (Martin Voorzanger & Leonard Wolters) and says "TARIDE Foundation (in formation)." The gap between two founders and the operational requirements is significant. Key questions:

1. How many FTEs does the Foundation need per phase?
2. What is the hiring plan and budget?
3. Are any roles filled by Calmido staff (creating the dependency flagged in BIZ-002)?
4. Are advisory board members expected to contribute substantive work, or only guidance?

**Recommendation:** Include a staffing plan (role, FTE count, phase) in the operational planning. Be explicit about which Foundation functions are performed by Calmido staff in the early phase, and the plan to transition to Foundation-employed staff.

---

### SUP-006 — Protocol evolution governance is missing
**Severity:** Major
**Section:** Development roadmap, governance

The protocol will evolve across four phases over five years. Phase 1 is telephony-only on a testnet. Phase 4 is pan-European multi-channel on production blockchain. This requires:

1. **Breaking changes.** When the DID method changes from PoC to production (TECH-001), what happens to PoC DIDs? Migrated? Abandoned? Dual-support period?
2. **Feature additions.** When email support is added (Phase 3), how are new credential types introduced? Do existing attestation providers need to upgrade?
3. **Deprecation.** When early-stage design decisions are superseded, how are they deprecated? What notice period do ecosystem participants get?
4. **Backwards compatibility.** For how long must the protocol support old versions? Who decides when support ends?

The IETF uses RFCs with status tracks (Proposed Standard → Internet Standard → Historic). W3C uses Recommendation tracks. TARIDE needs an equivalent process or its specification will become a moving target that participants can't rely on.

**Recommendation:** Define a protocol evolution process: proposal format, review process, versioning scheme, deprecation timeline, and backwards compatibility policy. Consider adopting an existing framework (e.g., IETF-style RFCs or W3C-style working group process).

---

### SUP-007 — Audit right is a power without a process
**Severity:** Major
**Section:** Application registration

"The foundation retains the right to audit any registered application for compliance with the protocol terms." This is stated as a deterrent ("the knowledge that an audit can occur at any time is often sufficient"). But:

1. **Scope.** What can the Foundation inspect? Source code? Server logs? Database contents? Financial records? The scope of an audit right determines whether applications will accept it.
2. **Process.** How is an audit initiated? Scheduled? Random? Complaint-driven? Who conducts it — Foundation staff, external auditors?
3. **Cost.** Who pays for the audit? If the Foundation bears the cost, audits will be rare. If the application bears the cost, it's a penalty.
4. **Confidentiality.** Audit findings may include competitive or proprietary information. What confidentiality protections apply?
5. **Consequences.** What happens when an audit finds non-compliance? Remediation period? Immediate sanctions?

Without these answers, the audit right is unenforceable — a paper tiger that sophisticated actors will ignore and risk-averse actors will fear.

**Recommendation:** Define audit scope, triggers, process, cost allocation, confidentiality, and consequence framework. Distinguish between routine compliance checks (lightweight, automated) and full audits (heavyweight, triggered by evidence).

---

### SUP-008 — No ecosystem health metrics defined
**Severity:** Minor
**Section:** Measurable objectives for 2031

The 2031 objectives table includes outcome metrics (90% reduction in unwanted communications, 10,000+ DIDs). But there are no ecosystem health metrics:

1. **Attestation provider availability** — % uptime per provider
2. **Cache node performance** — lookup latency P50/P95/P99, cache hit rates
3. **Credential freshness** — average time between credential issuance and cache availability
4. **Revocation propagation** — time from revocation to network-wide visibility
5. **Reputation data quality** — manipulation detection rates, contribution volumes
6. **Application ecosystem** — active applications, new registrations, churn

These are the vital signs of the protocol. Without tracking them, the Foundation cannot detect degradation or demonstrate health to stakeholders.

**Recommendation:** Define ecosystem health metrics with targets per phase. Publish them transparently (dashboard or periodic report) to build stakeholder confidence.

---

### SUP-009 — Open-source community model undefined
**Severity:** Minor
**Section:** Governance, how to get involved

"Technical contribution: review protocol specifications, contribute to the open-source codebase, or run a cache node." But:

1. Where is the code hosted? (GitHub? GitLab? Self-hosted?)
2. What license governs contributions? (Apache 2.0 is stated for specs — what about reference implementations?)
3. Is there a contributor guide?
4. What is the contribution review process? (PR review, CLA, CI/CD requirements?)
5. Is there a community forum, mailing list, or chat?
6. How are external contributors credited and recognised?

An open-source protocol without a functioning open-source community is a foundation-controlled project with a permissive license — structurally different from what "open-source" implies.

**Recommendation:** Define the open-source community model before the PoC launches. At minimum: code hosting, contribution guide, CLA or DCO policy, review process, communication channels, and governance participation rights for contributors.

---

### SUP-010 — Localisation and internationalisation not considered
**Severity:** Minor
**Section:** European positioning, measurable objectives

The document targets pan-European scale by 2031 across 27 Member States. This implies:

1. **Language.** Protocol documentation, error messages, trust profile labels, privacy notices — in how many languages? The GDPR requires privacy information in the language of the data subject.
2. **Legal variation.** Each Member State has local data protection authority, local telecom regulation, local business registration (not every country has a KvK equivalent). The credential issuer framework must be country-specific.
3. **Number format.** E.164 is universal, but number portability databases are country-specific. Each country's COIN equivalent has different APIs and data formats.
4. **Cultural expectations.** Trust signals may be interpreted differently across cultures. A "reputation score" may be acceptable in the Netherlands but controversial in Germany.

**Recommendation:** Include an internationalisation section that identifies country-specific dependencies (portability databases, credential issuers, legal requirements) and the approach for scaling from NL to EU.

---

## Open Questions

1. **Who governs the Foundation until the board is formed?** During "in formation" status, who makes decisions? The founders? Under what authority?
2. **What legal form is the Foundation?** Dutch "stichting"? If so, stichtingen have specific legal constraints (no members, board-governed) that differ from a "vereniging" or other forms.
3. **How are attestation providers removed from the network?** If a telecom exits the market, goes bankrupt, or becomes non-compliant — what is the process and who inherits their responsibilities?
4. **What is the support model for end users?** If a user's DID is incorrectly marked or their trust profile is wrong, who do they contact? The Foundation? The app developer? The attestation provider?
5. **How is the Foundation accountable to the public?** Published financials? Annual reports? Public board meetings? For an entity claiming to build public infrastructure, transparency is a governance requirement.
6. **What is the process for adding a new communication channel?** When email or WhatsApp support is added, who defines the attestation provider requirements for that channel? How are new channel-specific credential types approved?
7. **Is there a formal relationship between Foundation governance and Calmido's product roadmap?** If Calmido decides to implement a protocol feature before it's formally specified, does that create a de facto standard?

---

## Spec Readiness Verdict: Supportability & Governance

**Not ready for specification.**

Two blockers:
- **SUP-001**: Governance structure is aspirational — no decision-making process, conflict resolution, or protocol change governance exists. A specification requires a process for its own evolution.
- **SUP-002**: No onboarding path for any participant type. The specification will define technical requirements, but without onboarding frameworks, no one can evaluate whether those requirements are achievable.

Five major findings need resolution:
- SUP-003 (enforcement due process), SUP-004 (documentation strategy), SUP-005 (staffing), SUP-006 (protocol evolution), SUP-007 (audit process)

Governance is the document's weakest section relative to its ambition. The structural protections (cannot be sold, mission-protected, open-source) are necessary but not sufficient. They prevent bad outcomes without enabling good ones. The specification needs a governance framework that can evolve the protocol, resolve disputes, onboard participants, and maintain quality — all while remaining transparent and accountable. That framework must be designed now, not after the spec is written.
