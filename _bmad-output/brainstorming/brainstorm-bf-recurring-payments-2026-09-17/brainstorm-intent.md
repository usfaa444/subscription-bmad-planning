# Brainstorm intent: BF recurring payments

**Spine:** Build the receivables-and-consent OS for Burkina Faso, render it as Stripe-like checkout and as SMS/USSD, collect via reminder-pay on Orange first, and leave the sockets open for every other rail, language, country, and future PI-SPI debit.

This file is the only downstream handoff seed from the 2026-09-17 brainstorm. It is not a product brief, PRD, or architecture.

---

## Product stance (binding)

- **Market:** Burkina Faso first. West Africa expansion is later, not this build.
- **Channels:** One dual-channel product — hosted checkout API **and** merchant portal/SMS — not two systems.
- **Rails:** Design for Orange Money, Moov Money, and Wave as interchangeable backends. Only Orange must go live in MVP. Payment-gateway flexibility is assumed; do not hard-lock a single MNO.
- **What “recurring” means here:** Customer-authorized pay-on-reminder, not merchant-initiated wallet pull. True A2A debit may come later via PI-SPI. Never market auto-debit.
- **Identity / locale:** Primary customer key is MSISDN (+ country code), not email. French-first UX. XOF first currency. No English, `$`, or card-network defaults on first-run screens.
- **Who we serve:** Offline providers (cleaning, tutoring, rent, school, gym, mosque) are as important as app merchants. Cash is a first-class collection method, equal in standing to MoMo.
- **Currency / country:** Currency is a tenant/country attribute, not a hardcoded XOF constant — even while launching BF-only.

---

## The two-sided job and the unified primitive

**Merchant bedrock job:** Stop chasing weekly/monthly money without hiring a collector.

**Customer bedrock job:** Stay in good standing with a trusted provider without walking to a kiosk every cycle.

These are the same loop: a shame-free receivables book plus a reminder-then-pay action.

**What recurring actually is:** A standing customer authorization plus a scheduled obligation — not a merchant-initiated wallet pull (those barely exist on BF MoMo).

**The collection primitive:** Invoice → SMS/USSD → customer confirms → wallet debit → merchant credit.

**The unified consent/checkout primitive:** One channel-agnostic state machine — identify → consent → method → confirm — rendered as hosted web, USSD, SMS verbs, or later IVR. Literacy constraints force that unification. Hosted web, USSD, SMS, and IVR are renderers of **one consent protocol**, not separate products.

**API vs portal:** One product, two skins, over the same checkout state machine and event bus. Splitting them would duplicate the real work. One merchant account can start on the portal and later add API access.

**Plan vs collection:** Separate *what is owed* from *how it is collected* (cash, MoMo, future bank debit) so they can change independently.

**Consent today, debit later:** Enroll first; collect on the agreed date. Consent copy written now must also cover future A2A debit. Show the next due date before it happens so future debit is never a surprise.

---

## Theme map (14)

Use as a vocabulary map only — not a backlog.

1. **Collection Primitive** — reminder-then-pay, standing authorization, scheduled obligation
2. **Dual-channel Access** — API checkout + portal/SMS as two skins of one product
3. **Identity & Household** — MSISDN primary key; shared phones; payer ≠ beneficiary
4. **Merchant Receivables OS** — living who-owes-what book, not generic subscription SaaS
5. **Rail Abstraction & PI-SPI** — MNO adapters, vacant DirectDebitPort, PI-SPI as future scheme
6. **Notifications Mesh** — SMS as OS; WhatsApp/IVR/flash as failover, not a single channel
7. **Trust/Fraud/Proof** — named receipts, matching shop name, payout clock, guilds
8. **Cash Culture Hybrid** — cash mark-paid as first-class; graduate cash payers to MoMo
9. **Developer Experience** — Stripe-like session + webhooks + sandbox + local samples
10. **Informal Onboarding** — name + phone in under 60s; sub-15-minute merchant sitting
11. **Vertical Templates** — weekly clean, school term, mosque due, gym month, rent
12. **Platform Economics & Growth** — take-rate after first success; guilds; developers as channel
13. **Multi-country Architecture** — CountryPack sockets; BF is the first pack, not the schema
14. **Literacy-first UX** — USSD/SMS/numeric verbs primary; web is a fallback, not the default

---

## Chosen directions

1. **Spine:** Dual-channel receivables + reminder-pay on one live rail, with a channel-agnostic consent state machine.
2. **Architecture bet to carry (not a design doc):** Rail adapters + vacant DirectDebitPort + CountryPack. Do not encode `if-Burkina`. Country → scheme → rail → adapter; PI-SPI is one future scheme.
3. **Go-to-market wedge:** Offline SMS merchants plus send-a-bill, then subscriptions. Guilds over ads. Developers as the online-merchant channel.
4. **Deferred moat:** Consented recurring mandates + PI-SPI-shaped sandbox. Market as **on-time collection**, never as auto-debit.

---

## Prioritized opportunity space

### MVP MUST

- Hosted Checkout Session API + webhooks (create session → hosted URL → event on clear)
- French MoMo checkout; no cards
- Merchant portal: plans, clients by phone, SMS invite
- Reminder-then-pay as the live collection path
- Ledger that separates plan (what is owed) from collection method
- Magic-link / OTP identity — no customer passwords
- One live rail (Orange Money) behind a rail adapter
- SMS invoice that answers what / to whom / how often / how much / by when
- Cash mark-paid so the ledger stays complete
- Idempotent collection tokens (retries, double-SMS, webhook replay cannot double-charge)
- CountryPack skeleton + XOF + MSISDN as customer keys
- Sandbox (no live Orange Money for QA)
- Basic pause / skip
- SMS receipt
- Staff PIN on shared devices
- Future-debit-capable consent copy (covers later A2A without rewriting merchant APIs)
- Take-rate after first successful collection — merchants free until then
- Sub-15-minute merchant onboarding (client registerable in under 60 seconds: name + phone)
- Failed-collection retry inbox
- Zero English / `$` / card-logo defaults

### NEXT SHOULD

- Moov + Wave + in-session failover (Orange timeout → offer Moov)
- WhatsApp receipts + IVR
- Escalating dunning + payday-aligned windows (~26th–5th)
- Collector offline cash + cash-day calendar
- Family / group pay + share-pay-link (payer who is not the beneficiary)
- Consumer home listing all subscriptions across merchants on this platform
- One-off invoices as the quieter digital-billing wedge
- Vertical plan templates (maid, school-fees term, mosque, gym)
- PWA + shop QR + paper membership card
- Payout to the merchant’s own MoMo / Wave merchant code
- MRR / churn / failed-collection analytics
- Laravel and Flutter samples; docs aimed at Ouaga / Abidjan developers
- Dispute freeze: “I did not get service” holds the next collection pending merchant reply
- Mooré / Dioula keywords (and bilingual messages merchants do not compose)
- Guild / association launch (coiffeuses, gyms, private schools) over cold ads
- One merchant account spanning portal and API
- Thermal / printed “PAID” receipt
- MNO daily-limit display before confirm
- Known-customer whitelist (existing WhatsApp relationship)

### LATER / WON’T this time

- Live PI-SPI direct debit
- Second-country pack in production (Côte d’Ivoire stays a paper shadow pack)
- Tontine / e-susu as a first-class product
- B2B supplier credit
- NGO / government / institutional payers (school canteen, mutuelle, church tithe)
- Customer-invites-merchant demand-side growth
- Split / franchise payouts
- Credit scoring / bureau-style default risk
- Payment-institution or e-money license
- Recurring payouts / salary-like out
- Savings-toward-due envelopes
- Prepaid auto-refill merchant wallets
- Tax / e-invoicing (leave empty hooks only)
- White-label domains
- Per-country aggregators as a product
- Voice-mandate as a legal instrument
- Women-support line as a standalone product
- Super-App consumer wallet

---

## Critical insights (would change a brief or PRD)

- **Same loop, two skins.** Merchant “stop chasing money” and customer “stay in good standing” are one product. API and portal share the checkout state machine and event bus; do not spec them as separate systems.
- **No PI-SPI is the wedge, not a hole.** Reminder-pay is the honest BF primitive. Consented mandates stockpiled now become the PI-SPI moat when the public API opens. The winner is whoever already has mandates when the switch flips.
- **Literacy forces one consent protocol.** Web, USSD, SMS verbs, and IVR are renderers. Feature-phone `*shortcode#` is equal to `https` checkout, not a degraded mode. SMS is the OS: every action has a verb (PAY, PAUSE, SOLDE, AIDE, STOP); one message must carry invoice + link + USSD.
- **Cash is the on-ramp.** Informal merchants enter via cash mark-paid. Graduating cash payers to MoMo is the growth loop, not a v2 apology. Do not shame cash payers.
- **CountryPack without building CI.** Isolate strings, rails, KYC, holidays, MSISDN parsing, and wallet prefixes behind CountryPack. BF is the first pack, not the schema. Do not write `if-Burkina`.
- **Two wedges, one account.** Stripe-like DX wins app merchants in a week; a 15-minute portal wins offline. Start portal, add API later, same account. One-off “send a bill” is a quieter first ask than “start a subscription.”
- **Trust artifacts beat features.** Named SMS, matching shop name and neighbourhood, regulated-looking receipt, guild distribution, and a visible payout clock (customer paid vs merchant received) will out-convert extra features in a cash culture.
- **Settlement finality is the product.** If the merchant cannot trust the money arrived, nothing above it matters. Failure default: the obligation remains; only the collection attempt failed.
- **Pick the rail at pay time.** Wallets do not interoperate cleanly. Customer chooses Orange vs Moov vs Wave based on which wallet has money *today*, not a locked-in rail at subscribe.
- **Demand-side is later.** Customer-invites-merchant and a cross-merchant consumer home are the same later bet — only after people already pay through us.
- **Trust-preserving defaults.** Silence means success (ping only on failure or change). Take-rate after first collect. Never market auto-debit. Shame-free private reminders by default.
- **A subscription is five primitives:** promise, schedule, payment instruction, record, reminder. Everything else is packaging.

---

## Non-goals for the next planning skill

The brainstorm did **not** produce a product brief, PRD, or architecture. This intent is the seed only — the next skill may write those documents, but must not treat this file as if it already is one.

**Do not carry forward as scope:**

- Auto-debit marketing or any copy that implies merchant-initiated wallet pull
- Second-country production build (CountryPack skeleton only; no live CI)
- Live PI-SPI / DirectDebitPort beyond a vacant adapter and sandbox-shaped simulation
- Native apps for offline merchants (mobile web + later PWA only)
- Card PAN checkout, English-default screens, or `$` amounts
- Platform-held customer balances if pass-through settlement is viable
- The entire LATER / WON’T list above (tontine, B2B credit, license, Super-App, etc.)

**Do carry forward:** the spine, the two-sided job, the unified consent/checkout primitive, the four chosen directions, and the MVP MUST list as the opportunity floor.

---

## Later naming note (2026-09-18)

After this brainstorm, the product was named **Sarati** (Dioula/Jula: agreement / contract / terms) with primary domain **sarati.net**. This brainstorm did **not** invent or select that name — the note is historical only so downstream readers connect this intent file to the branded planning docs.

