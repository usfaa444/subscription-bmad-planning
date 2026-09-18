---
title: BF Recurring Payments — PRD addendum
created: 2026-09-17
updated: 2026-09-17
---

# Addendum — BF Recurring Payments PRD

Depth that does not belong in `prd.md`: technical-how, rejected alternatives, NEXT/LATER catalogs, landscape citations, and persona color. **MVP floor remains the PRD §6 and brief Scope.** This file is not a second source of truth for in-scope FRs.

## 1. System idea — superseded interpretations

Verbatim source: `docs/system-idea.md`. PRD wins on product law.

- “Customer logs in” = Magic Link or OTP, not a password account.
- “Access a portal” ≠ Customer account home. Method is chosen at pay time (Hosted Checkout Page or no-browser PAY path).
- “Automated debits later” = LATER vacant Direct Debit Port — not live, not marketed, not bundled into MVP consent.

## 2. Technical-how (architecture owns)

**Spine.** Dual-channel receivables + reminder-then-pay on one live Rail; channel-agnostic Consent State Machine.

**Architecture bet (not a design).** Rail Adapters + vacant Direct Debit Port + CountryPack. Flow: Country → scheme → Rail → Adapter. No `if-Burkina` in product logic. PI-SPI is one future scheme, accessed via a licensed participant, not a raw public debit API for this startup.

**Subscription = five primitives.** Promise, schedule, payment instruction, record, reminder. Everything else is packaging.

**Collection Primitive.** Invoice → SMS → Customer confirms → Rail debit after Customer confirm → Merchant credit. Recurring = Standing Authorization + scheduled Obligation — not merchant-initiated pull. Operator USSD may appear as the **Rail-wallet** confirm, not as a platform shortcode.

**Orange Adapter.** Product forbids hard-locking the commercial counterparty. Practical paths (2026 landscape): local Orange Money merchant APIs (sales-led; Orange Developer Web Payment country list has historically omitted BF — conflict with Orange BF “paiement en ligne” pages) and/or a BF aggregator. Customer confirm is typically USSD OTP / in-app approve. Architecture chooses; PRD FR-21 is the outcome.

**Moov / Wave.** Design Adapter interfaces now. Moov collect in-market is aggregator/USSD-push, not easy self-serve. Wave has a documented Business Checkout API; still not a receivables OS. Live collect NEXT.

**Ledger events.** Gateway-agnostic: intent, authorized, collected, settled, failed, refunded, written-off.

**Identity.** MSISDN + country code as primary key. Currency as CountryPack attribute.

**DX.** Session → hosted URL → signed Webhook. Sandbox without live Orange. Laravel/Flutter samples and Ouaga/Abidjan field docs are NEXT.

**Illustrative API names (not contract).** `createSession`, retrieve Session, Webhook topics for status. Exact routes, auth, and idempotency header names are architecture. PRD locks status vocabulary: `authorized`, `collected`, `failed`, `expired`, `cancelled`, `settled`.

**Versioning (draft for architecture).** Additive Rail and Direct Debit Port must not mint a second Session type. Breaking status renames require a dated version; prefer additive statuses.

## 3. Rejected alternatives

| Alternative | Why rejected |
| --- | --- |
| Two products (API company + portal company) | Duplicates Consent State Machine and event bus |
| Market auto-debit / “set and forget” | Dishonest on BF Rails; trust and ARCEP/BCEAO-hostile |
| Cards / English / `$` checkout as default | Wrong market; kills conversion |
| Build Côte d’Ivoire production in v1 | CountryPack sockets yes; production pack no |
| Live PI-SPI debit now | Scheme moving; not a licensed participant; Merchant API must stay stable |
| Native Merchant apps | Informal Merchants will not install; mobile web first |
| Platform-held Customer wallets | License + trust; prefer Pass-through |
| Demand-side Customer-invites-Merchant | Only after volume through the platform |
| Tontine / Super-App / credit bureau | Different products; steal focus from five primitives |
| Guilds as a built product surface | Distribution motion, not an MVP feature |
| Treating Flutterwave/Paystack “subscriptions” as the model | Those products tokenize cards; BF MoMo is payer-approved each time |

## 4. NEXT (out of MVP FRs)

| Item | One-line reason |
| --- | --- |
| Moov + Wave live + in-session failover | Trust Orange first |
| Live `*shortcode#` USSD renderer | Cost/availability; model already USSD-capable |
| WhatsApp receipts + IVR | After SMS delivery is reliable |
| Escalating dunning + payday windows (~26th–5th) | Learn from retry inbox |
| Collector offline cash + cash-day calendar | Cash Mark-paid is the wedge |
| Family / group pay + share-pay-link | Payer ≠ beneficiary is trust/UX risk |
| Consumer home of all platform Subscriptions | Demand-side after volume |
| One-off invoices as a **named product** | Primitive already in MVP |
| Vertical Plan templates (maid, school term, mosque, gym) | Clone-a-plan first |
| PWA + shop QR + paper membership card | Mobile web first |
| Payout to Merchant’s own MoMo / Wave merchant code as a productized feature | Settlement/legal design beyond Pass-through MSISDN |
| MRR / churn / failed-collection analytics suite | Receivables book first |
| Laravel and Flutter samples; Ouaga-only docs (no Abidjan production pack) | API + Sandbox first |
| Dispute freeze (“I did not get service”) | After happy path |
| Mooré / Dioula keywords | French-first; Merchants should not compose bilingual SMS |
| Guild / association launch | GTM, not a product surface |
| Thermal / printed PAID receipt | SMS Receipt first |
| MNO daily-limit display before confirm | Needs per-Rail metadata |
| Known-Customer whitelist | Fraud later |

## 5. LATER / WON’T this version

Live PI-SPI direct debit · Second-country production pack (CI = paper shadow only) · Tontine / e-susu · B2B supplier credit · NGO / government / institutional payers · Customer-invites-Merchant growth · Split / franchise payouts · Credit scoring · Payment-institution or e-money license · Recurring payouts / salary-like out · Savings-toward-due envelopes · Prepaid auto-refill Merchant wallets · Tax / e-invoicing (empty hooks only) · White-label domains · Per-country aggregators as a product · Voice-mandate as a legal instrument · Women-support line as a standalone product · Super-App Customer wallet · Native apps · Card PAN checkout · Platform-held Customer balances if Pass-through is viable · Auto-debit marketing · Compliance calendar (BF license vs UEMOA passporting) as an MVP artifact · `bmad-deep-recon` TAM study (deliberately not run).

## 6. Landscape digest (Discovery research, 2026-09-17)

Confidence: high on official BCEAO/ARCEP texts and operator/PSP docs; medium on blog market-share; low on unpublished commercial contracts and PI-SPI “débit différé” merchant readiness.

**Rails.** Orange Money and Moov Money still dominate MNO-led wallets. Wave present since ~2021 (local claims >1M users / 10k+ points — treat as commercially present, not proven volume leader). Orange BF has a local merchant “paiement en ligne” path; pan-African Orange Developer WebPay listings have historically omitted BF. Moov: no easy self-serve public API. Wave: documented Checkout + webhooks + payouts; aggregated/PayFac-style merchants invite-only.

**Every current MoMo collect is payer-approved in-session.** Silent merchant-initiated debit is not a live BF primitive.

**PSPs.** CinetPay, PayDunya, LigdiCash, InTouch, Flutterwave connectors: checkout / payment-link / payout. Recurring on Flutterwave is card/token language. Gap: roster + schedule + consent record + SMS dunning + cash-equal ledger.

**PI-SPI.** Launched 30 Sep 2025. Participant connection still completing in 2026 (deadlines extended). Developer portal / Business API / Request-to-Pay scopes exist for **connected participants**. No evidence of a public third-party standing-mandate debit for this product class as of Sep 2026. Sockets urgent; live debit out.

**Regulatory (high level).** Do not issue e-money (EME capital bar). Payment-institution agrément is a different path (explicit consent, generally no fund holding for PIS). Practical launch: software + agent of a licensed collect counterparty. ARCEP SVA shortcode is a declaration path; Orange also sells in-country SMS API without vanity shortcode. 2018 SVA rules emphasize subscriber-initiative subscribe/stop — aligns with honest reminder-then-pay and STOP mapping.

Selected sources: [PI-SPI](https://pispi.bceao.int/accueil); [BCEAO connection communiqués 2026](https://bceao.int/fr/communique-presse/connexion-la-plateforme-interoperable-du-systeme-de-paiement-instantane-pi-spi-de); [PI-SPI developer](https://developer.pispi.bceao.int/); [Orange BF paiement en ligne](https://www.orange.bf/business/fr/orange-money-paiement-en-ligne.html); [Orange Developer WebPay](https://developer.orange.com/apis/om-webpay); [Wave Business](https://docs.wave.com/business); [CinetPay](https://cinetpay.com/pricing); [PayDunya DMP](https://developers.paydunya.com/doc/EN/dmp); [Instruction 001-01-2024](https://www.bceao.int/sites/default/files/2024-02/Instruction%20N%C2%B0001-01-2024%20relative%20aux%20services%20de%20paiement%20dans%20l%27UMOA%20%20ok.pdf); [ARCEP SVA](https://www.arcep.bf/services-a-valeur-ajoutee-2/).

## 7. Persona color (not lead UJs)

Awa, Fatou, and Adama live in PRD UJ-1–UJ-3. Extra color only:

- **Issa** — school bursar; term peaks; proof + late list; vertical template NEXT. Not MVP acceptance.
- **Mariam** — gym/salon; Pause/Skip; shame-free private reminder.
- **Imam / treasurer** — cash-heavy; named SMS matching the mosque matters more than the dashboard.

## 8. Inputs this addendum absorbed

- `_bmad-output/planning-artifacts/briefs/brief-bf-recurring-payments-2026-09-17/brief.md`
- `_bmad-output/planning-artifacts/briefs/brief-bf-recurring-payments-2026-09-17/addendum.md`
- `docs/system-idea.md`
- `_bmad-output/brainstorming/brainstorm-bf-recurring-payments-2026-09-17/brainstorm-intent.md`
- Headless product-owner locks (take-rate, Orange path, Pass-through, KYC gate, SMS/STOP, USSD, Payout Clock, one account)
- Discovery landscape scan (2026-09-17)
