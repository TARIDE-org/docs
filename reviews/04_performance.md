# Review 4: Performance & Scalability

**Reviewer perspective:** Senior platform engineer with expertise in high-throughput distributed systems, real-time APIs, blockchain scalability, and telecom-grade infrastructure.

**Document reviewed:** TARIDE Foundation v0.51 (taride_foundation.md)

**Date:** 2026-04-04

---

## Findings

### PERF-001 — No capacity model or sizing estimates
**Severity:** Blocker
**Section:** Throughout

The document targets "pan-European scale" by 2031 but contains no capacity estimates. Without these, infrastructure costs, blockchain selection, and resolver network design cannot be evaluated. Basic questions:

1. **How many DIDs?** Netherlands alone: ~22 million mobile subscriptions, ~10 million email addresses. EU: ~450 million mobile subscriptions. At full penetration, hundreds of millions of DIDs.
2. **How many lookups per second?** The Netherlands had ~2.8 billion voice call minutes per quarter (ACM, 2024). At one lookup per call, that's ~360 lookups/second sustained for NL alone. EU-scale: ~10,000-50,000 lookups/second. Peaks (weekday mornings) are 3-5x average.
3. **How many credential operations per day?** New registrations, revocations, portability events, metadata updates.
4. **How much on-chain storage?** Per DID: public key + creation timestamp + credential commitments. At 200 bytes per DID and 100 million DIDs, that's 20 GB of on-chain state — significant for any blockchain.
5. **How much resolver network storage?** Full trust profiles with reputation, consent, metadata, logos.

The revenue model assumes "10 million lookups per day" — but this is a Netherlands-only pilot number. EU scale is 100x that.

**Recommendation:** Build a capacity model with three scenarios (PoC: 10K DIDs, Pilot: 100K DIDs, EU scale: 100M DIDs). Derive lookup rates, storage requirements, on-chain transaction volumes, and bandwidth needs for each. Use these to validate architectural decisions.

---

### PERF-002 — 500ms SLA has no error budget or degradation strategy
**Severity:** Major
**Section:** Resolver layer

The sub-500ms target is stated but not decomposed. A realistic latency budget for an incoming call lookup:

| Component | Estimate |
|---|---|
| App detects incoming call, initiates lookup | 10-50ms |
| DNS resolution to cache node | 0ms (cached) to 50ms |
| TLS handshake (if new connection) | 50-100ms |
| App → cache node network RTT | 10-80ms (geography-dependent) |
| Cache node processing | 5-20ms |
| Cache node → app network RTT | 10-80ms |
| App processes and renders trust profile | 10-50ms |
| **Total (warm path, cache hit)** | **~50-200ms** |
| **Total (cold path, TLS setup, cache miss)** | **~300-600ms** |

The cold path already threatens the 500ms target. A cache miss requiring attestation provider fetch or on-chain verification would blow it entirely.

Missing definitions:
- Is 500ms measured end-to-end (app-to-app) or cache-node-only?
- Is it P50, P95, or P99?
- What happens when the SLA is missed? (Fail-open showing "loading..." or fail-closed showing "unverified"?)
- Is there a timeout-and-retry strategy?

**Recommendation:** Define the latency SLA precisely (measurement point, percentile, timeout behaviour). Design a warm-path-only lookup for the incoming call scenario (persistent connections, pre-cached profiles). Accept that cold-path lookups will miss the target and define the degraded UX.

---

### PERF-003 — Blockchain transaction throughput will bottleneck at scale
**Severity:** Major
**Section:** Registration layer, development roadmap

On-chain operations include: DID registration, credential commitment writes, credential revocations, and daily reputation commitment hashes.

Throughput estimates:

| Scenario | Daily on-chain transactions |
|---|---|
| PoC (10K DIDs, low churn) | ~100-500 |
| Pilot (100K DIDs) | ~1,000-5,000 |
| NL production (10M DIDs) | ~50,000-200,000 |
| EU scale (100M DIDs) | ~500,000-2,000,000 |

Ethereum L2 throughput (Arbitrum/Optimism): ~40-100 TPS sustained = 3.5M-8.6M transactions/day. This is sufficient for EU scale in aggregate, but:

1. **Cost**: At $0.01-0.10 per L2 transaction, EU-scale daily costs are $5,000-$200,000/day. Who pays?
2. **Burst capacity**: Number portability campaigns (a telecom migrating users) could generate thousands of credential operations in minutes.
3. **EBSI**: EBSI's throughput is much lower (permissioned, fewer validators). The document lists EBSI as a production option but it may not handle the transaction volume.

**Recommendation:** Estimate transaction volumes per phase. Evaluate whether batch transactions (aggregating multiple DID operations into a single on-chain transaction via Merkle trees) can reduce cost and throughput requirements. This may be necessary for L2 viability and essential for EBSI.

---

### PERF-004 — Cache coherence problem is unaddressed
**Severity:** Major
**Section:** Resolver layer

Multiple independent cache nodes serve trust profiles. When data changes (credential revocation, reputation update, consent change):

1. **How quickly must all cache nodes reflect the change?** The document says revocation propagates "in real time" — but in a distributed cache with independent operators, "real time" requires a propagation mechanism.
2. **What is the consistency model?** Strong consistency (all nodes see the same data at the same time) is impractical across independent operators. Eventual consistency means different apps querying different cache nodes may get different answers for the same DID.
3. **Stale data risk**: A revoked credential could still appear valid on a cache node that hasn't received the update. This is a security issue disguised as a performance issue — a scammer could race the revocation propagation.

The number recycling section acknowledges this: "cache nodes must actively propagate credential revocations rather than relying on cache expiry." But no mechanism is specified.

**Recommendation:** Define the consistency model explicitly. Specify maximum acceptable staleness per data type (credentials: seconds, reputation: minutes, consent: minutes). Design the propagation mechanism (event bus, webhooks, polling with short intervals). Accept that strong consistency across independent operators is impractical and design around eventual consistency with bounded staleness.

---

### PERF-005 — Reputation commitment cycle creates write amplification
**Severity:** Major
**Section:** Reputation layer

"The resolver network writes a cryptographic commitment (a hash of the current reputation state) to the chain" with a "target of daily for the production network." At scale:

1. What is "the current reputation state"? A single hash of all reputation data? Per-DID hashes? Per-application hashes?
2. A single global hash means any reputation change invalidates the entire commitment — any auditor must re-verify everything.
3. Per-DID hashes scale linearly with DID count — at 100M DIDs, daily commitment of 100M hashes is infeasible on any blockchain.
4. A Merkle tree commitment reduces this to a single root hash, but requires the full tree to be reconstructable for verification.

The integrity guarantee ("any party can verify that the reputation data served by a cache node matches the committed state") requires the verifier to have access to the full reputation dataset — defeating the purpose of off-chain storage.

**Recommendation:** Specify the commitment scheme (likely a Merkle tree with root on-chain). Define what "any party can verify" means in practice — is it spot-checking or full verification? Specify who maintains the full Merkle tree and how verifiers access proofs.

---

### PERF-006 — Logo delivery at scale is a CDN problem, not a protocol problem
**Severity:** Minor
**Section:** Verified logo

If trust profiles include logos, cache nodes become image servers. At 50,000 lookups/second EU-scale, with even 10% including a logo, that's 5,000 logo deliveries/second. Logo sizes vary (1KB SVG to 50KB PNG). This is a classic CDN workload that doesn't belong in a protocol lookup path.

**Recommendation:** Deliver logos via URL reference (pointing to a CDN), not inline in the trust profile response. Cache nodes return the logo URL; the application fetches and caches the image separately, outside the 500ms lookup budget.

---

### PERF-007 — Reputation feedback aggregation load is underestimated
**Severity:** Minor
**Section:** Reputation layer, application registration

"Applications submit aggregated feedback to the resolver network via the protocol API." At scale:

- 50+ applications submitting reputation data
- Each covering potentially millions of DID-instance combinations
- Submission frequency unspecified (real-time? Hourly? Daily?)

The resolver network must ingest, validate, cross-reference for manipulation, aggregate across applications, compute quality scores, and update cached profiles. This is a data pipeline, not a simple API write.

**Recommendation:** Specify submission frequency limits, batch sizes, and the expected processing pipeline. Consider whether reputation updates should be asynchronous (queued) rather than synchronous.

---

### PERF-008 — On-chain verification during lookups creates a hot path dependency
**Severity:** Minor
**Section:** Resolver layer

"Cache nodes replicate data from attestation providers and verify it against on-chain commitments." If this verification happens on every lookup, every lookup depends on blockchain query latency. If verification is periodic (background sync), there's a window where cache nodes serve unverified data.

The document doesn't clarify when on-chain verification occurs: at data ingestion (attestation provider → cache node), at query time (cache node → application), or both.

**Recommendation:** Specify that on-chain verification occurs at data ingestion, not at query time. Cache nodes should pre-verify and serve verified data from local storage. This keeps the blockchain out of the hot lookup path.

---

## Open Questions

1. **What is the expected adoption curve?** The capacity model depends on how many users register per phase. "10,000+ verified DIDs" by 2031 is conservative — but if adoption is faster, does the architecture hold?
2. **What is the cost per DID per year?** On-chain storage, resolver network hosting, credential renewals — what is the all-in cost to support one DID?
3. **How does the system handle flash crowds?** A news event mentioning TARIDE could drive millions of registrations in hours. Can the PoC infrastructure handle 100x expected load?
4. **What is the cache node hardware/hosting profile?** What does it take to run one? This determines who can operate a cache node and whether the "any party" claim is realistic.
5. **What is the bandwidth requirement per cache node?** At 50,000 lookups/second, with 1KB average response size, that's 50MB/s sustained — modest for a data centre, significant for a smaller operator.
6. **Are there geographic locality requirements?** Must cache nodes be co-located with attestation providers? With users? With blockchain nodes?
7. **What happens during blockchain downtime?** L2 sequencer outages have occurred (Arbitrum, 2023). If the chain is down, can new DIDs be registered? Can credentials be revoked?

---

## Spec Readiness Verdict: Performance & Scalability

**Not ready for specification.**

One blocker:
- **PERF-001**: Without a capacity model, no infrastructure decision can be validated. The difference between 10K and 100M DIDs is not a scaling question — it's a different architecture.

Five major findings need resolution:
- PERF-002 (SLA definition), PERF-003 (blockchain throughput/cost), PERF-004 (cache coherence), PERF-005 (reputation commitments), and the interaction between PERF-003 and TECH-006 (blockchain choice constrains throughput)

The document's performance claims are plausible at PoC scale but unvalidated at production or EU scale. The 500ms target is achievable for cache hits but needs a concrete design. The blockchain scalability question is real but solvable with batching and L2 selection. The biggest gap is that the document never quantifies the problem it's solving — how many lookups, how many DIDs, how much data — which makes all other performance analysis speculative.
