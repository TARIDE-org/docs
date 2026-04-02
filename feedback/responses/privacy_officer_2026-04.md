# Privacy Officer Feedback

**Date:** April 2026
**Stakeholder:** Privacy officer (informal review)
**Document reviewed:** taride_foundation.md v0.5
**Tier:** 1 (Privacy / digital rights)

## Summary

The entire processing constitutes processing of personal data and falls under the GDPR. The data is not anonymous (in the legal sense), but pseudonymous. All measures described appear to be security measures to ensure that actual identification is only reasonably possible for those who are authorised (including telcos and law enforcement). It is not anonymisation, because identification (by someone) is not excluded and inherently cannot be excluded. This means that DIDs linked to an E.164 address at a telco constitute personal data.

## Key findings

### 1. GDPR applicability — pseudonymisation, not anonymisation

The processing falls under GDPR. The protocol produces pseudonymous data, not anonymous data. A DID coupled to an E.164 number at a telco is a personal data point because the telco can identify the natural person behind it. Identification by *someone* is not excluded and cannot inherently be excluded. The measures described (cryptographic separation, no personal data on-chain) are security measures, not anonymisation measures.

### 2. Data controller responsibility

A data controller (verwerkingsverantwoordelijke) or joint controllers must be designated. Per GDPR definition, the controller is the organisation that "determines the purposes and means of processing." That is the Foundation in relation to the DIDs. The fact that the Foundation does not itself process or have access to the data is irrelevant. Precedent: IAB Europe was found to be a controller by the Belgian DPA in relation to the TCF string, despite not processing data itself.

### 3. Compliance obligations

The Foundation must organise and facilitate that it and others can comply with GDPR. This includes arrangements with all involved parties and ensuring that data subject rights can be honoured:
- Right to information (art. 13/14)
- Right of access (art. 15)
- Right to rectification (art. 16)
- Right to erasure (art. 17)
- Right to object (art. 21)

This has significant operational implications.

### 4. Value proposition and scalability concerns

The reviewer questions what problems this proposal solves and whether the solution has such high entry barriers that it will not achieve the scale necessary for effectiveness. No clear network effect is identified.

## Original feedback (Dutch)

> Kort samengevat heb ik de volgende punten. De gehele verwerking lijkt mij een verwerking van persoonsgegevens en valt onder de GDPR. Het is niet anoniem (in juridische zin), maar pseudo-anoniem. Alle maatregelen lijken me beveiligingsmaatregelen om daadwerkelijke identificatie alleen redelijkerwijs mogelijk te maken voor wie dat mag of moet (waaronder telco's en politie/justitie bijv via hen). Het is geen anonimisering, want identificatie (door iemand) is niet uitgesloten en kan inherent ook niet uitgesloten worden, zo lijkt mij. Dat betekent dus bijvoorbeeld dat de DIDs die bij de Telco aan een E.164 adres gekoppeld zijn, een persoonsgegeven vormen.
>
> De consequenties dat er een verwerkingsverantwoordelijke of gezamenlijke verwerkingsverantwoordelijken aangewezen moeten/zullen worden. Een verwerkingsverantwoordelijke is, per definitie in GDPR, de organisatie die "doelen en middelen van de verwerking vaststelt". Dat is de Foundation in relatie tot de DIDs. Dat de Foundation niet zelf verwerkt of bij de gegevens kan maakt niet uit/is irrelevant. IAB Europe is hierop gestuit bij de Belgische privacy autoriteit ihkv de TCF string.
>
> Dat betekent dus dat de Foundation moet organiseren/faciliteren dat zij en andere aan GDPR kunnen voldoen. Dwz regelingen maken met betrokken partijen en organiseren dat rechten van betrokkenen gehonoreerd kunnen worden. Waaronder recht op informatie, inzage, correctie, verwijdering en verzet. Heeft nogal wat voeten in de aarde.
>
> Ik zie niet goed welk problemen dit voorstel oplost en/of de oplossing niet dusdanig veel entry barriers heeft dat het daardoor geen schaalgrootte zal gaan krijgen. Ik zie bijvoorbeeld geen echt netwerk effect.
>
> Dus: wel degelijk vallend onder GDPR (en wereldwijd soortgelijke wetten), dat heeft consequenties, en ik vraag me af of er behoefte aan zal zijn.

## Actions required

- [x] Correct terminology throughout foundation document: "anonymous" → "pseudonymous" where legally inaccurate (v0.51)
- [x] Add GDPR section: controller responsibility, data subject rights, processing agreements, DPIA (v0.51)
- [x] Address right-to-erasure vs blockchain immutability tension (v0.51, in GDPR section under art. 17)
- [ ] Strengthen value proposition and network effect argument
