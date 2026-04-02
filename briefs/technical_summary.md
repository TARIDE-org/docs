# TARIDE — Technical Protocol Summary

*Extracted from the [foundation document](../taride_foundation.md) v0.51, April 2026. For review by the DID/SSI community.*

## What TARIDE is

An open protocol for verified digital communications. Combines decentralised identifiers, verifiable credentials, a distributed resolver network, and a reputation layer to provide real-time trust profiles for communication instances (phone numbers, email addresses, messaging handles).

**Core principle:** Pseudonymity by default, identification by choice. Verification credentials confirm that an instance is registered to an active account without revealing the holder's identity.

## Core concepts

**Communication channel.** A type of communication: telephony, email, WhatsApp, Signal, SMS.

**Instance.** A specific contact point within a channel. `+31 6 1234 5678` (telephony), `user@example.com` (email), `@user` (WhatsApp).

**Attestation provider.** The provider that confirms the coupling between a DID and a specific instance. For telephony: the telecom operator. For email: the email provider. For messaging: the platform.

**Cache node.** An operator in the resolver network that serves trust profiles to applications in real time. Replicates data from attestation providers and verifies against on-chain commitments.

## Protocol layers

### 1. Registration

DID holders generate a cryptographic key pair. Public key registered on-chain as a DID. Private key stored in device secure enclave. The DID is pseudonymous — no inherent link to any personal information.

In practice, an application (e.g., Calmido) or the future EU Digital Identity Wallet handles key generation, DID registration, and instance linking. No wallets, seed phrases, or blockchain interactions visible to the user.

### 2. Verification

Two credential categories:

**Mandatory — instance verification:**
- Attestation provider confirms the instance is registered to the DID holder
- No personal information disclosed
- Two types for telephony (per GSMA proposal for eIDAS 2.0):
  - *Contract owner credential* — DID holder is the paying customer
  - *End user credential* — DID holder is the actual device user (may differ from contract owner)
- A single DID can hold multiple credentials (dual-SIM, multi-provider)
- Each credential independently issued and revocable

**Optional — identity and attestations:**
- KvK (chamber of commerce) registration
- Regulatory credentials (banking licence, healthcare registration)
- Verified logo (inspired by BIMI, tied to trademark verification via VMC or EUIPO)
- Organisation affiliation (employer-to-employee credential, revocable)

**Continuous validation:**
- Optional silent re-authentication via mobile network (CAMARA Number Verification API)
- Each credential presentation can trigger real-time proof of SIM possession
- Defends against SIM swap — re-authentication fails if SIM was swapped

### 3. Reputation

- Opt-in for DIDs with identity credentials (businesses, institutions)
- Anonymous DIDs are not subject to reputation scoring
- Individual user feedback stays in the originating application
- Only aggregated scores submitted to the resolver network
- Scores stored off-chain; periodic cryptographic commitment hash written on-chain
- Revenue-sharing incentive for applications that contribute quality reputation data

### 4. Consent

Per-channel preferences set by the DID holder:
- Open (accept all — default)
- Verified (only from DIDs with active verification credential)
- Established (minimum DID age + association age threshold)
- Trusted (positive reputation or identity credentials)
- Contacts only (maximum restriction)

Stored in resolver network (off-chain). Enforcement is at the application layer — the protocol stores and serves preferences but does not block communication.

### 5. Resolver network

Distributed infrastructure with two distinct roles:
- **Attestation providers** — issue/revoke credentials, maintain instance-DID associations
- **Cache nodes** — serve trust profiles to applications (<500ms), verify against on-chain commitments

Follows the DNS model: multiple independent operators, no single point of failure. Any party meeting technical and governance requirements can operate a cache node.

**Attestation provider registry:** Each provider declares which channels and instance ranges it serves. The protocol validates that a provider is authorised before accepting credentials.

**Automatic attestation provider detection:** For telephony, queries existing number portability infrastructure (COIN in NL). For email, DNS MX lookup. For messaging, platform namespace.

## Standards alignment

| Standard | Relationship |
|---|---|
| **W3C DID** | DIDs follow the W3C Decentralized Identifiers specification |
| **W3C Verifiable Credentials** | Credentials follow VC data model |
| **eIDAS 2.0** | Protocol designed to complement; EU Digital Identity Wallet as future DID container |
| **EBSI** | Target deployment chain for production (alternative: Ethereum L2) |
| **GSMA CAMARA** | APIs consumed as verification inputs (Number Verification, SIM Swap, Number Recycling) |
| **GSMA mobile number as VC** | Contract owner / end user credential distinction adopted from GSMA proposal |
| **BIMI** | Verified logo concept inspired by BIMI, without email dependency |

## Open design decisions

The following decisions are not yet finalised. Input from the SSI community is specifically sought.

### DID method

The foundation document does not specify a DID method. Options under consideration:

| Option | Pros | Cons |
|---|---|---|
| `did:ethr` | Exists, works on Ethereum, mature tooling | Limited to Ethereum-compatible chains; may not fit EBSI deployment; resolution depends on a specific smart contract |
| `did:web` | Simple, no chain dependency | Centralised (depends on web server availability); not self-sovereign in the strict sense |
| `did:key` | No registration needed, pure cryptographic | No on-chain anchor; no updateability; no key rotation |
| Custom `did:taride` | Full control over resolution, features, governance | Requires writing and registering a DID method spec; ecosystem adoption starts from zero |

**Question:** Given the requirements (on-chain anchor for timestamps, key rotation support, planned EBSI migration, eIDAS 2.0 wallet compatibility), which DID method fits best?

### Credential format

| Option | Pros | Cons |
|---|---|---|
| **JSON-LD + LD-Proofs** | W3C reference format; rich semantics; interoperable with EU wallet ecosystem | Complex; large payloads; JSON-LD processing overhead |
| **JWT (JWS)** | Simple; compact; widely supported; fast verification | No selective disclosure; all-or-nothing presentation |
| **SD-JWT** | Selective disclosure built in; compact; growing adoption in eIDAS context | Newer standard; less tooling maturity |

**Question:** The protocol needs selective disclosure (present verification without revealing identity credentials). SD-JWT appears to be the natural fit. What are the interoperability risks given planned eIDAS 2.0 wallet integration?

### Credential status mechanism

The protocol requires real-time credential revocation visible to all cache nodes. Options:

| Option | Pros | Cons |
|---|---|---|
| **Revocation lists (CRL-style)** | Simple; well-understood | Scaling issues; cache nodes must periodically fetch full lists |
| **Status list (Bitstring StatusList)** | Compact; W3C draft standard; single bitfield per issuer | Requires credential index management; privacy concerns (which bit flipped?) |
| **On-chain status registry** | Real-time; verifiable; no polling | Gas costs per revocation; scaling concerns |
| **Accumulator-based (e.g., CL accumulator)** | Privacy-preserving; efficient membership proofs | Complex; less mature tooling |

**Question:** Given the requirement for real-time propagation and the resolver network architecture (cache nodes verifying against on-chain commitments), which mechanism balances performance, privacy, and complexity?

### On-chain vs off-chain boundary

Current design:
- **On-chain:** DIDs, credential anchors (hashes), periodic reputation commitment hashes
- **Off-chain (resolver network):** Instance-DID associations, full credentials, reputation scores, consent preferences

**Question:** Is this boundary correct? Should more data move off-chain (e.g., credential anchors) or on-chain (e.g., instance-DID association commitments)?

### Cryptographic agility

The foundation document mentions migration to post-quantum signature schemes. No specific approach is described.

**Question:** What is the recommended approach to cryptographic agility for a DID method that needs to support key rotation and long-lived identifiers?

## Technical questions for review

1. Does `did:ethr` fit the requirements or does the protocol need a custom DID method?
2. What credential format (JSON-LD, JWT, SD-JWT) best supports selective disclosure for this use case?
3. What interoperability traps should we avoid given planned eIDAS 2.0 wallet integration?
4. How should the resolver network handle credential status checks at scale (revocation lists vs status registries)?
