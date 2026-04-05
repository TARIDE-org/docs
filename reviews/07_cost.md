# Review 7: Cost

**Reviewer perspective:** Senior financial analyst / CFO with expertise in infrastructure economics, blockchain cost modelling, non-profit sustainability, and technology platform unit economics.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-05

---

## Findings

### COST-001 — No financial model exists
**Severity:** Blocker
**Section:** Foundation revenue model, throughout

The document contains exactly one financial number: "€0.001 per query × 10 million lookups/day = €3.65M annually." This is a napkin calculation, not a financial model. Missing:

1. **Cost side** — no estimate of what it costs to run the protocol at any scale
2. **Revenue timeline** — no projection of when each revenue stream activates
3. **Break-even analysis** — when does the Foundation become self-sustaining?
4. **Sensitivity analysis** — what if adoption is 50% slower? What if gas prices 10x?
5. **Cash flow** — when does money come in vs. when must it go out?

A foundation asking telecom partners, government agencies, and EU grant bodies to invest cannot present a document with one revenue line and zero cost lines. This is not a specification issue per se, but the specification will encode architectural decisions that have direct cost implications. Without a model, those decisions are uninformed.

**Recommendation:** Build a three-scenario financial model (conservative, base, optimistic) covering 2026-2031. Include all cost categories below. Present it alongside the foundation document to any stakeholder being asked to commit resources.

---

### COST-002 — Blockchain costs are unquantified and potentially prohibitive
**Severity:** Major
**Section:** Registration layer, development roadmap, incentives

On-chain operations (DID registration, credential commitments, revocations, reputation anchoring) incur gas costs on public networks. The document says "transaction costs remain low enough for the protocol to scale" without quantifying:

**Ethereum L2 costs (2026 estimates):**
| Operation | Frequency (NL scale, 10M DIDs) | Cost per tx (L2) | Daily cost |
|---|---|---|---|
| DID registration | ~5,000/day (growth phase) | $0.01-0.05 | $50-250 |
| Credential commits | ~10,000/day | $0.01-0.05 | $100-500 |
| Credential revocations | ~1,000/day | $0.01-0.05 | $10-50 |
| Reputation anchoring | 1/day (global hash) | $0.01-0.05 | $0.01-0.05 |
| **NL daily total** | | | **$160-800** |
| **EU scale (100x)** | | | **$16,000-80,000/day** |

At EU scale, on-chain costs alone could reach $5M-30M/year. The €3.65M annual lookup revenue doesn't cover it.

**EBSI alternative:** Zero gas costs (permissioned), but EBSI is pre-production with no commitment to supporting TARIDE's transaction volumes.

**Who pays gas costs?** The document is silent. Options: the Foundation (from revenue), the attestation provider (passed to telecoms), the application (passed to users). Each has different economic implications.

**Recommendation:** Model on-chain costs per scenario. Evaluate batching strategies (Merkle tree aggregation, rollups within rollups) to reduce per-operation costs. Specify who bears gas costs. Include gas cost volatility as a risk factor — L2 costs have varied 10x within single quarters.

---

### COST-003 — Infrastructure costs for resolver network not estimated
**Severity:** Major
**Section:** Resolver layer, incentives

Cache nodes must: store all trust profiles, serve sub-500ms lookups, replicate from attestation providers, verify against on-chain commitments, and handle reputation ingestion. What does this cost?

**Rough cache node cost model:**
| Component | Estimate |
|---|---|
| Compute (high-availability, multi-AZ) | $2,000-5,000/month |
| Storage (trust profiles for 10M DIDs, ~5KB each = 50GB, with redundancy) | $50-200/month |
| Bandwidth (10M lookups/day × 1KB response = 10GB/day egress) | $200-500/month |
| Blockchain node or RPC access | $200-1,000/month |
| Monitoring, logging, security | $500-1,000/month |
| Operations staff (part-time SRE) | $3,000-8,000/month |
| **Total per cache node** | **$6,000-16,000/month** |

At minimum 3 cache nodes for redundancy: $18,000-48,000/month = $216,000-576,000/year — just for the NL pilot. EU scale with geographic distribution might require 10-20 cache nodes: $720,000-3.8M/year.

The attestation provider role has additional costs: credential issuance infrastructure, CAMARA API integration, metadata management, compliance with joint controller obligations. These are borne by telecoms — but the document doesn't quantify them, making the telecom business case (BIZ-003) unsubstantiated.

**Recommendation:** Estimate infrastructure costs per component at each scale tier. Present the cache node cost model to potential operators so they can evaluate the business case. Estimate attestation provider integration costs to support telecom partnership conversations.

---

### COST-004 — Compliance and legal costs are invisible
**Severity:** Major
**Section:** Data protection and GDPR, governance

The GDPR section creates substantial compliance obligations:
- DPIA (Article 35): €20,000-80,000 for a thorough assessment with external counsel
- Joint controller arrangements: legal drafting per attestation provider, €10,000-30,000 each
- Data processing agreements: per cache node operator, €5,000-15,000 each
- Privacy notices: drafting and maintenance, €5,000-15,000
- Data subject request handling: operational process, potentially dedicated staff
- Data breach response: incident response capability, potentially 72-hour notification requirement
- Supervisory authority interactions: legal representation

Estimate for GDPR compliance setup: €100,000-300,000. Annual maintenance: €50,000-150,000.

Additional legal costs:
- Foundation establishment and governance documents: €20,000-50,000
- Intellectual property (patent FTO, trademark): €15,000-40,000
- Telecom partnership agreements: €20,000-50,000 per partner
- EU grant applications (specialist grant writers): €10,000-30,000 per application

None of this appears in the document.

**Recommendation:** Include a legal and compliance budget in the financial model. These costs are front-loaded (occur before revenue) and represent a significant portion of early-stage spending.

---

### COST-005 — Revenue-sharing economics don't work at small scale
**Severity:** Major
**Section:** Foundation revenue model, application registration

The revenue-sharing model: applications that contribute reputation data receive a share of lookup fees. At the stated rate (€0.001/lookup, 10M lookups/day):

- Gross lookup revenue: €10,000/day = €3.65M/year
- Foundation takes a cut (unspecified — assume 30%): €1.1M
- Cache node operators take a cut (unspecified): assume €1M
- Remaining for revenue share: ~€1.5M/year
- Split among 50+ contributing applications: ~€30,000/year per app maximum

€30K/year is not a meaningful incentive for a serious application developer to build and maintain a reputation data pipeline. And this is the "at scale" scenario. During the pilot with 1-2 applications and 100K lookups/day, revenue share per app is ~€100/year.

**Recommendation:** Acknowledge that revenue sharing is a scale-stage incentive, not a bootstrap incentive. Design early-stage incentives differently: free API access for contributing apps, co-marketing, data reciprocity (contributors get enriched trust data). Revisit the revenue split when the actual numbers are known.

---

### COST-006 — EU grant dependency is high-risk
**Severity:** Major
**Section:** Foundation revenue model

The early-stage funding plan relies on "EU grants and subsidies" from Digital Europe, Horizon Europe, and the European Digital Identity framework. Risks:

1. **Timelines.** Digital Europe Programme calls have 6-12 month evaluation cycles. Horizon Europe is even longer. Funding disbursement often lags 3-6 months after award. A grant applied for in Q2 2026 may not deliver funds until Q1-Q2 2027.
2. **Success rates.** Horizon Europe success rates are 15-17%. Digital Europe is slightly higher but still competitive. Relying on grants as a primary funding source means a >50% chance of not receiving any given grant.
3. **Conditionality.** EU grants come with reporting requirements, audit obligations, and co-funding requirements (typically 25-50% of project cost must come from the applicant). The Foundation needs matching funds it doesn't yet have.
4. **Scope constraints.** Grant funding is tied to specific work packages. It cannot fund general operations, salaries, or legal costs unless those are in the work plan.

**Recommendation:** Diversify early-stage funding beyond grants. Explore: Dutch national innovation funding (RVO, NWO), strategic investment from a committed telecom partner, philanthropic funding aligned with digital rights, or a recoverable loan from Calmido B.V. Don't treat grants as a certainty — build a funding plan that survives zero grant awards.

---

### COST-007 — Cost to end users is claimed as zero but isn't
**Severity:** Minor
**Section:** Incentives (end users)

"Participation costs the user nothing as the app handles everything." But:

1. The app (Calmido) presumably charges a subscription or has a freemium model. The user pays for the app, which includes TARIDE protocol access.
2. The DID registration requires an on-chain transaction. Someone pays the gas. If the app absorbs this, it's an acquisition cost per user.
3. Continuous validation requires CAMARA API calls. These have per-call costs charged to the operator or the app.

The user doesn't pay the Foundation directly, but the cost is embedded in the application layer. If Calmido charges €2/month and 50% of its costs are TARIDE-related, the user effectively pays €1/month for the protocol.

**Recommendation:** Be transparent about the economic chain. "Free to the user" is true at the protocol level but potentially misleading. Articulate who ultimately bears each cost component.

---

### COST-008 — Premium credential fees have no market benchmark
**Severity:** Minor
**Section:** Foundation revenue model

"Businesses that want enhanced trust profiles pay an annual verification fee. This is comparable to how SSL certificate authorities and domain registrars operate."

The SSL/domain analogy is dated. Let's Encrypt destroyed the paid SSL certificate market by offering free certificates. Domain registration is a regulated market with fixed wholesale prices. Neither is a reliable revenue model analogy.

More relevant benchmarks:
- Google Verified Calls: free to businesses (Google monetises through ecosystem control)
- Apple Business Connect: free
- BIMI/VMC certificates: $1,000-1,500/year (DigiCert) — but adoption is low because of cost
- Truecaller for Business: $500-5,000/month depending on volume

If TARIDE prices like BIMI ($1,000/year), it needs 3,650 paying businesses to hit €3.65M. If it prices like Google (free), it needs alternative revenue. The document doesn't take a position.

**Recommendation:** Research willingness-to-pay for verified business trust profiles. Define a pricing range, not just an analogy. Consider whether premium credentials should be free initially to drive adoption (BIZ-001 cold start) and monetised later.

---

## Open Questions

1. **What is the Foundation's annual operating budget?** Staff, infrastructure, legal, travel, governance — what does it cost to run the Foundation per year, before any protocol infrastructure?
2. **Who pays for the PoC?** Infrastructure, development, legal setup. Is this Calmido's investment, founder capital, or committed external funding?
3. **What is Calmido's unit economics?** If the Foundation's early viability depends on Calmido's success, the business case for Calmido is implicitly the business case for TARIDE.
4. **What is the cost per DID per year to the Foundation?** On-chain storage, resolver network hosting, credential renewal overhead, support — what is the all-in cost to maintain one DID?
5. **How does the Foundation handle currency risk?** Gas costs are denominated in ETH/crypto. Revenue is in euros. Crypto volatility creates budget uncertainty.
6. **What happens if a cache node operator determines the economics don't work?** Does the Foundation run a cache node of last resort? What's the cost?
7. **What is the cost of a DPIA, legal framework, and governance structure?** These are pre-revenue, pre-grant expenses that must be funded from somewhere.

---

## Spec Readiness Verdict: Cost

**Not ready for specification.**

One blocker:
- **COST-001**: No financial model. Architectural decisions encoded in the specification (blockchain choice, resolver design, credential format, caching strategy) all have cost implications that cannot be evaluated without a model.

Five major findings need resolution:
- COST-002 (blockchain costs), COST-003 (infrastructure costs), COST-004 (compliance costs), COST-005 (revenue-sharing viability), COST-006 (grant dependency risk)

The document is a technical and strategic vision — it is not a business plan and perhaps shouldn't be. But the specification will lock in cost structures. The choice between L2 and EBSI alone could represent a $5M/year cost difference at EU scale. A financial model isn't a nice-to-have; it's a prerequisite for informed specification decisions.
