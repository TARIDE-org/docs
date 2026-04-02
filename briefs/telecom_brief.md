# TARIDE — Technical Brief for Telecom Providers

*Extracted from the [foundation document](../taride_foundation.md) v0.51, April 2026*

## What TARIDE is

TARIDE is an open protocol for verified digital communications. When someone receives a call, the protocol confirms that the caller's phone number is registered to an active account at a telecom provider — without revealing the caller's identity.

The protocol does not replace telecom infrastructure. It adds a verification layer on top of existing networks. Phone calls are established as usual. TARIDE is not in the signalling path.

## The role of the telecom provider

Telecom providers are **attestation providers** in the TARIDE protocol. They are the only parties that can authoritatively confirm that a specific phone number is registered to a specific account. This makes telecom participation foundational — without attestation providers, the protocol cannot issue credentials.

### What an attestation provider does

1. **Confirms number ownership.** When a subscriber registers on the protocol (through an app like Calmido), the attestation provider issues a verification credential: a cryptographically signed, time-bound statement that the phone number is registered to an active account. No personal data is disclosed.

2. **Provides credential metadata.** The verification credential can include metadata that the provider already holds:
   - Registration type (prepaid or subscription)
   - Account age
   - Identification level (anonymous activation vs verified subscriber)
   - Last SIM swap timestamp
   - Last number reassignment

3. **Revokes credentials.** When a number is disconnected, ported out, or flagged for abuse, the attestation provider revokes the credential. Revocation propagates through the resolver network in real time — all connected applications immediately see the change.

4. **Handles number portability.** When a subscriber ports to a new provider, the old provider revokes its credential. The new provider issues a fresh one. The subscriber's DID (decentralised identifier) and reputation are unchanged — only the attestation provider changes.

### What an attestation provider does NOT do

- Does not own, control, or store the subscriber's DID or private key
- Does not see or process reputation data
- Does not enforce consent preferences (that is the application's responsibility)
- Does not need to modify call routing or signalling infrastructure

## Integration with existing infrastructure

### CAMARA APIs

The protocol is designed to consume existing CAMARA APIs as verification inputs:

| CAMARA API | Role in TARIDE |
|---|---|
| **Number Verification** | Continuous silent re-authentication — confirms the number is still active and on the same SIM without user interaction |
| **SIM Swap** | Populates the `last_sim_swap` metadata field in the verification credential |
| **Number Recycling** | Populates the `last_number_reassignment` metadata field; triggers credential lifecycle events |
| **KYC / Identity** | Optional input for enhanced identity credentials (subscriber-consented) |

TARIDE does not duplicate these APIs. It consumes their outputs and adds a persistent, cross-operator trust layer that individual API calls cannot provide: DID-based identity, time-based reputation, and recipient consent.

### COIN shared infrastructure

In the Netherlands, COIN already operates the infrastructure that maps closely to the TARIDE resolver network:

| COIN function | TARIDE equivalent |
|---|---|
| Number portability database | Automatic attestation provider detection — routing verification requests to the correct operator |
| CAMARA Discovery Service | Attestation provider registry — declaring which provider is authoritative for which number ranges |
| SMS Sender ID Register | Analogous verification concept, extended to all channels |
| Cross-operator fraud prevention | Credential revocation propagation; reputation layer |

The resolver network could potentially operate on top of existing COIN shared infrastructure, reducing integration effort for individual operators.

### Attestation provider registry

Each attestation provider declares which channels and number ranges it serves. This registry ensures that:
- Verification requests are routed to the correct operator
- A provider can only issue credentials for numbers in its declared range
- The protocol rejects credentials from unauthorised providers

This is analogous to DNS nameserver authority for specific zones.

## Credential lifecycle

```
Subscriber registers    → Provider issues verification credential
Number active           → Credential valid, metadata updated periodically
SIM swap occurs         → last_sim_swap metadata updated (if exposed)
Number ported out       → Provider revokes credential
Number ported in        → New provider issues fresh credential
Number disconnected     → Provider revokes credential
Number recycled         → New subscriber, new DID, fresh credential
Abuse reported          → Provider can revoke credential
```

The DID (subscriber's protocol identity) persists across all these events. Reputation stays with the DID and number combination, not with the attestation provider.

## Credential types

The protocol adopts the GSMA's proposed distinction for mobile numbers as verifiable credentials:

**Contract owner credential** — confirms the DID holder is the registered contract owner. Relevant for lifecycle events (SIM replacement, porting, contract changes).

**End user credential** — confirms the DID holder is the actual user of the device and number. May differ from the contract owner (family plans, corporate subscriptions). Can be silently re-authenticated via the mobile network.

A single DID can hold multiple credentials (dual-SIM, multiple subscriptions). Each is independently issued and revocable.

## Architecture overview

```
┌─────────────┐     ┌──────────────────┐     ┌────────────┐
│  Subscriber  │────▶│  Application     │────▶│ Cache node │
│  (DID holder)│     │  (e.g. Calmido)  │     │            │
└─────────────┘     └──────────────────┘     └─────┬──────┘
                                                    │
                           ┌────────────────────────┤
                           │                        │
                    ┌──────▼──────┐          ┌──────▼──────┐
                    │ Attestation │          │  Blockchain  │
                    │  provider   │          │  (L2/EBSI)   │
                    │  (telecom)  │          └─────────────┘
                    └─────────────┘
```

- **Attestation providers** issue and revoke credentials, maintain instance-DID associations
- **Cache nodes** serve trust profiles to applications in real time (<500ms)
- **Blockchain** stores DIDs, credential anchors, and cryptographic commitments (not personal data)

These are separate roles. A telecom provider operates as an attestation provider. Cache nodes can be operated by any party meeting technical and governance requirements.

## Technical questions for review

1. What subscriber metadata (account age, registration type, SIM swap timestamp) can be exposed through existing APIs without infrastructure changes?
2. What are the technical constraints for credential issuance and revocation latency in your current systems?
3. How does the proposed attestation provider model interact with existing CAMARA API implementations and COIN's shared infrastructure?
4. What are the integration requirements for number portability event propagation?

## Timeline

| Phase | Scope | Timeline |
|---|---|---|
| Proof of concept | Core protocol on Ethereum testnet, first attestation provider integration | Q2-Q3 2026 |
| Pilot | Real-world credential issuance, expanded resolver network | Q4 2026-Q2 2027 |
| Production | L2 or EBSI deployment, multi-operator support | Q3 2027-Q4 2028 |
