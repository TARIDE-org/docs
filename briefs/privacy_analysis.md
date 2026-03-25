# TARIDE — Privacy Analysis

*Extracted from the [foundation document](../taride_foundation.md) v0.5, March 2026. For review by privacy advocates and digital rights organisations.*

## Core principle

**Anonymity by default, identification by choice.** A phone number can be verified as registered without revealing who the holder is. Identity disclosure is always optional, always under the holder's control.

This is the fundamental differentiator from existing solutions. Truecaller and Hiya require or incentivise identification as a precondition for trust. STIR/SHAKEN verifies the network path, not the holder. TARIDE inverts the model: verification is the default, identification is optional.

## Protocol overview

The protocol enables any party to verify that a communication (call, email, message) comes from a registered source. It operates through five layers: registration (DIDs), verification (credentials from telecom providers), reputation (community feedback on opted-in organisations), consent (recipient-defined reachability rules), and a resolver network (real-time lookup infrastructure).

## Data flow mapping

### What is stored where

| Data | Location | Who controls it | Visibility |
|---|---|---|---|
| **DID (public key + creation timestamp)** | On-chain | DID holder created it; immutable once registered | Public — anyone can see that a DID exists and when it was created |
| **Private key** | Device secure enclave | DID holder exclusively | Never leaves the device |
| **Instance-DID association** (e.g., phone number X belongs to DID Y) | Resolver network (off-chain) | Attestation provider maintains it; DID holder initiated it | Queryable by connected applications via cache nodes |
| **Verification credential** (number is registered to an active account) | On-chain (anchor/hash), full credential off-chain | Issued by attestation provider, held by DID holder | Credential status is public; full credential details served by cache nodes |
| **Credential metadata** (account age, registration type, SIM swap timestamp) | Part of verification credential | Attestation provider provides it | Served to querying applications as part of trust profile |
| **Identity credentials** (KvK registration, banking licence, verified logo) | On-chain (anchor/hash), full credential off-chain | Issued by credential issuer, held by DID holder — **opt-in only** | Only present if the DID holder chose to add them |
| **Organisation affiliation** | Off-chain credential | Issued by organisation's DID to employee's DID — revocable | Visible to querying applications if the employee consents |
| **Reputation scores** | Resolver network (off-chain), periodic commitment hash on-chain | Aggregated from contributing applications | Only for DIDs that opted into visibility (typically businesses) |
| **Individual user feedback** | Originating application only | The application | Never leaves the app — only aggregated scores are submitted to the resolver network |
| **Consent preferences** | Resolver network (off-chain) | DID holder sets and modifies them | Queryable by applications to determine if communication is welcome |
| **Application UUID** | Foundation registry | Foundation issues and can revoke | Infrastructure-level; not visible to end users |

### What is NOT stored

- Names, addresses, or any personal information — not on-chain, not in the resolver network
- Phone numbers are not stored on-chain — only the cryptographic association between a DID and an instance exists in the resolver network
- Individual user feedback — stays in the originating app
- Call records, message content, communication history — the protocol verifies identity, not content

## On-chain footprint analysis

The blockchain stores:
1. DID registrations (public key + creation timestamp)
2. Credential anchors (hashes of issued/revoked credentials)
3. Periodic reputation commitment hashes

### What can be inferred from on-chain data alone

- That a DID exists and when it was created
- That credentials were issued or revoked for a DID (but not which phone number or instance)
- Transaction patterns: how frequently a DID's credentials change
- If the chain is public (Ethereum L2), all of the above is visible to any observer

### What CANNOT be inferred from on-chain data alone

- Which phone number, email, or handle is associated with a DID
- The identity of the DID holder
- Who is calling whom
- Reputation scores (only commitment hashes are on-chain)

## Linkability assessment

### Within the protocol

A single DID can hold multiple instances across channels (phone number + email + messaging handle). **This links those identities at the protocol level.** Anyone who can query the resolver network for a DID can see all associated instances.

**Risk:** If an attacker knows one instance (e.g., a phone number), they can query the resolver network and discover all other instances linked to the same DID.

**Mitigation discussed in foundation doc:** Not explicitly addressed. The document states the resolver network serves this data to connected applications. The architecture does not currently include access controls on which instances are visible per query.

### Across protocol and external data

- DID creation timestamps + credential issuance patterns could be correlated with telecom provisioning records by an attestation provider
- Cache node query logs (who queried which instance, when) could reveal communication patterns
- Reputation feedback timing could correlate with specific calls if the feedback is submitted immediately after

### Cross-DID correlation

- The one-instance-one-DID constraint means that if an instance moves to a new DID, the transition is visible in the resolver network
- A sequence of DID creations followed by instance transfers could fingerprint a user across identity resets

## GDPR considerations

| Processing activity | Data subject | Likely legal basis | Controller |
|---|---|---|---|
| DID registration | DID holder | Consent or legitimate interest | DID holder (self-sovereign) |
| Credential issuance | DID holder | Contract performance (telecom contract) or consent | Attestation provider |
| Instance-DID association stored in resolver network | DID holder | Consent or legitimate interest | Attestation provider / cache node operator (to be determined) |
| Trust profile lookup by application | DID holder (queried) | Legitimate interest of the recipient | Application / cache node operator |
| Reputation feedback aggregation | DID holder (rated), feedback provider | Legitimate interest | Application (individual feedback), resolver network (aggregated) |
| Consent preference storage | DID holder | Consent | Cache node operator |

### Open GDPR questions

- **Controller/processor roles:** Who is the data controller for instance-DID associations? The DID holder initiated the association, the attestation provider confirmed it, the cache node serves it. The foundation document does not specify controller/processor relationships.
- **Right to erasure:** A DID holder can deactivate their DID. But on-chain data (DID registration, credential hashes) is immutable. How is the right to erasure reconciled with blockchain immutability?
- **Data minimisation:** Is the credential metadata (account age, registration type, SIM swap timestamp) the minimum necessary for the stated purpose? Could the protocol achieve its goals with less metadata?
- **Purpose limitation:** Trust profiles are served to any connected application. Is there a mechanism to ensure applications only use the data for the stated purpose (trust verification)?
- **Cross-border transfers:** If cache nodes operate in multiple jurisdictions, what governs data transfers? The resolver network is designed to be distributed.

## Comparison with existing solutions

| | TARIDE | Truecaller | Hiya | STIR/SHAKEN |
|---|---|---|---|---|
| **Identification required** | No — anonymity by default | Yes — name attached to number via crowdsourced data | Yes — caller ID database | No — verifies network path, not holder |
| **Personal data collected** | No PII stored by protocol | Contact books uploaded, names/numbers aggregated | Caller ID databases, user reports | Carrier-level data |
| **User consent for data sharing** | Opt-in for identity, opt-in for reputation visibility | Implicit — installing the app uploads your contacts | Implicit — carrier-side integration | No end-user involvement |
| **Who controls the identity** | DID holder (self-sovereign) | Truecaller (centralised database) | Hiya (centralised database) | Carrier |
| **Data monetisation** | No — foundation model, protocol fees only | Yes — core business model | Yes — B2B data licensing | No |
| **On-chain data** | DIDs, credential hashes, reputation commitments | N/A | N/A | N/A |
| **Decentralised** | Yes — multiple attestation providers and cache nodes | No — single company | No — single company | Partially — per-carrier implementation |

## Technical questions for review

1. What privacy risks do you see in the on-chain footprint (DID registration, credential hashes)?
2. Is anonymity-by-default credible given that DID activity patterns could be correlated across transactions?
3. What is the linkability risk when a single DID holds multiple instances across channels?
4. What GDPR legal basis applies to each data processing activity in the protocol?
5. How does the data model compare to Truecaller/Hiya from a privacy perspective?
