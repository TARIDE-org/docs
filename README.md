# TARIDE

Trust and Authentication Registry for Integrity in Digital Europe

An open, European protocol for trust without identification.

Read the [foundation document](taride_foundation.md) (v0.51, April 2026).

## Technical review briefs

The foundation document describes the full protocol vision. For targeted technical review, we have extracted focused briefs for specific audiences:

- [Telecom brief](briefs/telecom_brief.md) — attestation provider role, CAMARA integration, credential lifecycle
- [Threat model](briefs/threat_model.md) — attack taxonomy, trust assumptions, open security questions
- [Privacy analysis](briefs/privacy_analysis.md) — data flows, on-chain footprint, linkability, GDPR considerations
- [Technical protocol summary](briefs/technical_summary.md) — DID method, credential format, resolver architecture, standards alignment

## Feedback process

We are actively seeking technical review from telecom providers, security researchers, privacy advocates, and the DID/SSI community. See the [feedback plan](feedback_plan.md) for the process, timeline, and success criteria.

If you have relevant expertise and would like to contribute a review, the [feedback form](feedback/templates/feedback_form.md) provides a lightweight structure for your response. All feedback is tracked in the [feedback tracker](feedback/tracker.md).

## Repository structure

```
taride_foundation.md              Foundation document (v0.51)
feedback_plan.md                  Feedback & validation strategy
briefs/
  telecom_brief.md                Technical brief for telecom providers / COIN
  threat_model.md                 Threat model for security researchers
  privacy_analysis.md             Privacy analysis for digital rights organisations
  technical_summary.md            Protocol summary for DID/SSI community
feedback/
  tracker.md                      Central feedback status tracker
  templates/
    feedback_request.md           Outreach email template
    feedback_form.md              Structured response template
  responses/                      One file per respondent (added as received)
```

Published under CC BY 4.0 · [taride.org](https://taride.org)
