# TARIDE — Threat Model

*Extracted from the [foundation document](../taride_foundation.md) v0.5, March 2026. For technical review by security researchers.*

## Protocol overview

TARIDE is an open protocol for verified digital communications. It combines decentralised identifiers (DIDs), verifiable credentials, a distributed resolver network, and a reputation layer to provide real-time trust profiles for phone numbers, email addresses, and messaging handles.

The protocol explicitly does NOT:
- Sit in the telecom signalling path
- Block or filter communications
- Store personal data on-chain
- Prevent SIM swaps or number fraud at the telecom layer
- Guarantee that all participants are honest

## Trust assumptions

The protocol relies on the following assumptions. Each is a potential attack surface.

| Assumption | Depends on | Failure mode |
|---|---|---|
| Telecom providers honestly attest number ownership | Provider operational integrity | False credentials issued for numbers the subscriber doesn't own |
| Private keys in mobile secure enclaves are not extractable | Device hardware security | DID impersonation, credential theft |
| Blockchain timestamps are unforgeable | Chain consensus mechanism | Fake DID age, undermining time-as-trust |
| Cache nodes serve authentic data | On-chain commitment verification | Stale or manipulated trust profiles served to applications |
| Credential revocation propagates in real time | Resolver network availability | Revoked credentials still appearing valid |
| One instance can only be linked to one DID at a time | Protocol enforcement at attestation provider level | Parallel identity building, sybil attacks |

## Attack taxonomy

### 1. Sybil attacks and reputation evasion

**Attack:** Create many DIDs to spam, build fake reputation, or abandon identities with poor reputation.

**Protocol defenses:**
- *Time-as-trust:* Two independent timestamps — DID age (on-chain, immutable) and instance-DID association age (tracked by attestation provider). Both reset to zero on a new identity. No attacker can fake time.
- *One-instance-one-DID:* A phone number can only be linked to one DID. To associate it with a new DID, the old link must be severed first. No parallel identity building.
- *Optional provider trust signal:* Telecom providers can add an endorsement to the credential (comparable to a blue checkmark). Adds weight without revealing identity.

**Open questions:**
- Is time-as-trust sufficient as the *primary* sybil resistance mechanism? What threshold values (DID age, association age) meaningfully deter attackers?
- How expensive is it for an attacker to maintain aged identities in reserve? What is the economic break-even point?
- The one-instance-one-DID constraint is enforced by attestation providers. What prevents a colluding or compromised provider from violating it?

### 2. SIM swap attacks

**Attack:** Attacker convinces telecom provider to transfer victim's number to a new SIM.

**What does NOT change after a SIM swap:**
- DID remains the same
- Instance (phone number) remains the same
- Attestation provider remains the same
- On-chain verification credential remains the same

**What does change:**
- Private key and SIM are now on different devices
- Victim has the private key but no SIM
- Attacker has the SIM but no private key

**Protocol defenses:**
- *Fundamental defense:* Attacker cannot use the DID. All protocol actions require the private key (stored in device secure enclave). The attacker gains the phone number for off-protocol use (calls, SMS) but cannot authenticate on-protocol.
- *Continuous validation (optional):* App on victim's device periodically re-authenticates with the mobile network. After SIM swap, re-authentication fails. Instance flagged as "verification degraded."
- *Credential metadata (optional):* If the telecom provider exposes `last_sim_swap` timestamp, applications can detect recent swaps.

**Open questions:**
- Both optional detection mechanisms depend on telecom integration. What is the realistic adoption path?
- If the victim's app is not running (device off, app uninstalled), continuous validation cannot detect the swap. How large is this gap?
- What happens when an attacker performs a SIM swap AND obtains physical access to the victim's device (and thus the secure enclave)?

### 3. Key management and device compromise

**Attack:** Extract or compromise the private key from the device.

**Architecture:** Private keys are generated and stored in the mobile device's secure enclave. The user never sees or manages keys directly. The app handles all cryptographic operations.

**Attack surfaces:**
- Secure enclave vulnerabilities (hardware-specific)
- Malware with elevated privileges
- Physical device access with biometric bypass
- Backup/restore flows that may export key material
- App vulnerabilities that expose signing operations

**Open questions:**
- What is the key recovery model? If a user loses their device, how do they regain control of their DID? The foundation document mentions the EU Digital Identity Wallet as a future portable container — what is the interim solution?
- What is the key rotation model? Can a DID holder rotate their key pair without creating a new DID?
- How are keys handled in multi-device scenarios (phone + tablet, dual-SIM)?

### 4. Credential fraud

**Attack:** Forge, replay, or misuse verification credentials.

**Protocol defenses:**
- Credentials are cryptographically signed by the attestation provider
- Credentials are time-bound (expiration)
- Credentials are revocable (real-time propagation)
- Credential status is verifiable against on-chain anchors

**Attack surfaces:**
- Compromised attestation provider signing key — can issue arbitrary credentials
- Colluding attestation provider — issues credentials for numbers the subscriber doesn't own
- Credential replay within validity window after revocation but before propagation
- Forged credential metadata (account age, registration type) if the provider is the sole source of truth

**Open questions:**
- What is the trust model for attestation providers? How are they vetted, monitored, and sanctioned?
- What is the maximum acceptable window between credential revocation and full propagation across all cache nodes?
- How does the protocol handle a compromised attestation provider signing key? Is there a key rotation or emergency revocation mechanism?

### 5. Privacy attacks

**Attack:** De-anonymise users or correlate activity despite anonymity-by-default claims.

**On-chain footprint:**
- DID registration (public key, creation timestamp)
- Credential anchors (hashes)
- Periodic reputation commitment hashes

**Off-chain data (resolver network):**
- Instance-DID associations
- Consent preferences
- Reputation scores

**Linkability risks:**
- A DID that holds multiple instances (phone + email + messaging handle) links those identities at the protocol level
- DID transaction patterns on-chain may be correlatable
- Cache node query patterns could reveal who is calling whom (traffic analysis)
- Reputation feedback submission patterns could de-anonymise contributors

**Open questions:**
- What is the realistic anonymity set? If a DID is linked to a phone number and an email address, how many users share that combination?
- The foundation document mentions zero-knowledge proofs for the pilot phase. Which properties need ZK protection? Credential verification without revealing the DID? Reputation queries without revealing the queried instance?
- What privacy guarantees do cache node operators have? Can a cache node operator perform traffic analysis on queries?
- How does the protocol prevent a "reverse lookup" — given a DID, enumerate all linked instances?

### 6. Resolver network attacks

**Attack:** Disrupt or manipulate the resolver network that serves trust profiles.

**Attack surfaces:**
- Cache node serves stale data (fails to propagate revocations)
- Cache node serves manipulated data (altered reputation scores, false credential status)
- Denial of service against cache nodes (prevent trust lookups during incoming calls)
- Attestation provider goes offline (no new credentials, no revocations)
- Sybil attack on cache node network (operate many nodes to control data distribution)

**Protocol defenses:**
- Cache node data is verifiable against on-chain cryptographic commitments
- Multiple independent cache node operators (DNS model)
- Attestation provider registry prevents unauthorised credential issuance
- Rate limiting per application UUID

**Open questions:**
- What is the consistency model? How does the protocol handle split-brain scenarios where different cache nodes serve different data?
- What is the minimum number of independent cache node operators for meaningful resilience?
- How are cache node operators vetted? What prevents a malicious actor from operating a cache node to perform traffic analysis?

### 7. Application-layer attacks

**Attack:** Exploit the application registration system or manipulate reputation through registered applications.

**Attack surfaces:**
- Register an application under false identity to submit manipulated reputation data
- Coordinate fake reputation submissions across multiple registered applications
- Exploit the revenue-sharing mechanism with inflated or fabricated lookup volumes
- Application UUID theft or spoofing

**Protocol defenses:**
- Application registration requires verifiable business identity
- Reputation data quality monitoring (consistency, volume, manipulation signals)
- Foundation can revoke application UUIDs
- Revenue sharing weighted by data quality score

**Open questions:**
- How robust is the reputation manipulation detection? What is the minimum detectable manipulation?
- What prevents a well-resourced attacker from registering many legitimate-looking businesses and coordinating reputation attacks?
- How is "data quality" defined and measured? What are the specific metrics?

## What the protocol explicitly does NOT protect against

- Telecom-layer attacks (the protocol does not sit in the signalling path)
- Content-based fraud (the protocol verifies identity, not intent)
- Social engineering that doesn't depend on caller identity
- Attacks that occur entirely off-protocol (traditional phone scams against non-users)
- A fully compromised attestation provider acting maliciously within its declared authority
- Nation-state adversaries with the ability to compromise device hardware at scale

## Technical questions for review

1. What attacks does this threat model miss?
2. Where are the weakest trust assumptions?
3. Is time-as-trust (DID age + association age) sufficient as the primary sybil resistance mechanism?
4. What key management risks exist when private keys are stored in mobile device secure enclaves?
5. What are the attack surfaces in the credential revocation propagation path?
