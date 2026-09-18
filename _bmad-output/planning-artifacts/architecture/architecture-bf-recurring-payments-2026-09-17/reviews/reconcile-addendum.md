---
title: Reconcile architecture spine against PRD addendum
date: 2026-09-17
spine: _bmad-output/planning-artifacts/architecture/architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md
addendum: _bmad-output/planning-artifacts/prds/prd-bf-recurring-payments-2026-09-17/addendum.md
verdict: PARTIAL — core bet landed; transport / versioning close / rails landscape / ledger catalog / NEXT-LATER catalog did not fully land
---

# Reconcile — Architecture Spine vs PRD Addendum

Read both artifacts dated 2026-09-17. Addendum is **not** a second FR source (`addendum.md` § header). This review asks only: which addendum architecture-how, rejected-alt, and NEXT/LATER material is missing from the spine, and where the two texts disagree.

## Verdict

**PARTIAL.** The spine absorbed the dual-channel / reminder-then-pay / hexagonal bet, vacant `DirectDebitPort`, CountryPack, Orange-live + Moov/Wave stubs, signed webhooks, `/v1` + no-second-Session-type versioning *rules*, and the pass-through ledger posture. It did **not** finish the addendum’s draft versioning, the Country → scheme → Rail → Adapter layering, rail *transport* (operator USSD / in-app confirm, Moov USSD-push, Wave Checkout shape), the addendum ledger catalog as written, MSISDN+country as identity key, or most of the NEXT/LATER catalog in `Deferred`.

No FR from the addendum was required to land (addendum forbids that). Gaps below are architecture-how and deferred-scope hygiene.

---

## What landed (baseline)

| Addendum § | Spine landing |
| --- | --- |
| §1 Magic Link/OTP; no Customer portal home; pay-time method | AD-3, AD-5, AD-11 |
| §1 / §2 automated debit = vacant Direct Debit Port, not marketed | AD-2, AD-4, AD-11, Deferred |
| §2 Dual-channel + reminder-then-pay + one Consent SM | AD-1, AD-3, AD-4 |
| §2 Rail adapters + CountryPack; no `if-Burkina` | AD-2, AD-5, AD-6 |
| §2 Collection = confirm then debit; SA + Obligation | AD-4, AD-9 |
| §2 No platform shortcode | AD-3 (FR-49) |
| §2 Orange counterparty not domain-locked | AD-5 + Deferred |
| §2 Moov/Wave interfaces now; live NEXT | AD-5 stubs + Deferred |
| §2 Session → hosted URL → signed webhook; Sandbox ≠ live Orange | AD-8, AD-11, AD-12, AD-14 |
| §2 Status vocabulary | AD-12 (exact match) |
| §2 Versioning *rules* (no second Session type; dated version on breaking rename; prefer additive) | AD-12 |
| §2 Currency = CountryPack attribute | AD-6, AD-13 |
| §3 Two products / auto-debit / cards-English-$ / CI prod / live PI-SPI / Customer wallets | AD-1, AD-4, Locale/Money, AD-6, Deferred, AD-7 |
| §4 Moov+Wave+failover, USSD renderer, WhatsApp/IVR, collector, vertical templates, payout-to-own-MoMo, Laravel/Flutter, PI-SPI | Deferred (partial list) |
| §5 Second-country paper shadow; license posture; float; auto-debit marketing | Deferred + AD-4/AD-7 |

---

## 1. Transport — did not land

Addendum transport is implicit across §2 (collection primitive, Orange/Moov/Wave how, DX) and §6 (SMS / PI-SPI / payer-approved collect). The spine locked *ports* and *HTTP prefix*, not *how a collect actually travels*.

| Addendum claim | Spine | Status |
| --- | --- | --- |
| Flow **Country → scheme → Rail → Adapter**. PI-SPI is a future *scheme*, reached via a **licensed participant**, not a raw public debit API | CountryPack → `PaymentRailPort` / vacant `DirectDebitPort` → `*Rail`. No scheme layer, no licensed-participant adapter path | **Missing** |
| Operator USSD may appear as **Rail-wallet confirm**, not a platform shortcode | AD-3: no platform shortcode; later USSD *renderer* may attach. Does not say Hosted Checkout / PAY waits on **operator** USSD OTP or in-app approve, then rail callback | **Missing** (risk: builders treat Orange’s own USSD as the deferred platform renderer) |
| Orange Customer confirm is typically **USSD OTP / in-app approve** | AD-11: API accepts no Rail PIN/OTP (correct). No adapter rule that Orange collect is async rail-owned confirm | **Missing** |
| Moov in-market collect is **aggregator / USSD-push**, not easy self-serve | Stub `MoovMoneyRail` only | **Missing** |
| Wave has a documented **Business Checkout API** (checkout + webhooks + payouts); still not a receivables OS | Stub `WaveRail` only | **Missing** |
| DX: create Session, **retrieve Session**, webhook topics | `POST /v1/checkout/sessions` + `Idempotency-Key` only. No `GET` retrieve. Events table is internal types, not a public topic catalog | **Partial** |
| §6 SMS: ARCEP SVA shortcode *declaration* **or** Orange in-country SMS API **without** vanity shortcode | SmsPort + vendor deferred. Fork not recorded | **Missing** |
| §6 Every current MoMo collect is **payer-approved in-session**; silent merchant-initiated debit is not a live BF primitive | AD-4 philosophically (no collect without Customer confirm). Not an adapter invariant on `PaymentRailPort.collect` | **Partial** |

**Landed transport:** HMAC-SHA256 `X-Webhook-Signature` (`t=`, `v1=`), outbox + at-least-once, Session create header names, SMS verbs, no platform shortcode, Hosted Checkout + SMS as renderers.

---

## 2. Versioning — did not land

Addendum §2: *“Versioning (draft for architecture).”* Architecture was supposed to finish the draft. AD-12 copied the two product rules and stopped.

| Draft item | Landed? |
| --- | --- |
| Additive Rail / Direct Debit Port must not mint a second Session type | **Yes** — AD-12 |
| Breaking status rename requires a **dated version**; prefer additive statuses | **Rule yes; mechanism no** |
| What “dated version” is (`/v2`, `/v2026-09-17`, `Api-Version` header, sunset of `/v1`) | **No** |
| Webhook payload version vs signature `v1=` | **No** (HMAC `v1=` is not API version) |
| How additive statuses are advertised without a bump | **No** |
| Retrieve / list / cancel Session under the same `/v1` contract | **No retrieve** (see transport) |
| Sandbox vs production contract version | **No** (separate env/key namespace only) |

Consent *text* version on `StandingAuthorization` (AD-11) is a different axis and **did** land. Do not confuse it with public API versioning.

---

## 3. Rails — did not land

Adapter *names* landed. Landscape *constraints* and the scheme layer did not.

| Addendum §2 / §6 | Spine | Status |
| --- | --- | --- |
| Design Moov/Wave adapters now; live collect NEXT | AD-5 stubs + Deferred | **Landed** |
| Orange: do not hard-lock local merchant API vs BF aggregator; FR-21 is the outcome | AD-5 + Deferred | **Landed** |
| 2026 conflict: Orange Developer WebPay listings historically **omit BF** vs Orange BF “paiement en ligne” | Not in spine (only in `.memlog.md` assumption) | **Missing from spine** |
| Moov: no easy self-serve public API; aggregator/USSD-push | Not recorded | **Missing** |
| Wave: Checkout + webhooks + payouts; PayFac-style invite-only; not an OS | Not recorded | **Missing** |
| PI-SPI launched 30 Sep 2025; participant connection still completing; Business API / Request-to-Pay for **connected participants**; no public third-party standing-mandate debit; **sockets urgent**, live debit out | Vacant `DirectDebitPort` + Deferred “Live PI-SPI / Request-to-Pay — vacant port only” | **Partial** — port yes; participant/socket/scheme no |
| PSP gap (CinetPay / PayDunya / … are checkout, not roster+consent+SMS+cash ledger) | Not required as an AD | Out of scope for spine |
| Practical launch: software + **agent of a licensed collect counterparty**; do not issue e-money | Float/license Deferred; Orange as adapter config. “Agent of licensed counterparty” not named | **Partial** |

---

## 4. Ledger — did not land / drifted

Addendum §2 gateway-agnostic events:

`intent`, `authorized`, `collected`, `settled`, `failed`, `refunded`, `written-off`

Spine AD-7:

`intent`, `authorized`, `collected`, `settled`, `failed`, `written-off`, **`reversed`**. **`refunded` reserved**; no Merchant-initiated refund in MVP.

| Item | Status |
| --- | --- |
| `refunded` as a peer event in the catalog | **Did not land** — demoted to reserved |
| `reversed` | **Spine addition** — not in addendum list (PRD Rail Reversal justifies it; catalog still drifted) |
| Five primitives: Promise, schedule, payment instruction, record, reminder | **Did not land** — no map onto Plan / StandingAuthorization / Obligation / CollectionAttempt / LedgerEntry / `reminder.dispatch` |
| Identity: **MSISDN + country code as primary key** | **Did not land** — see contradictions |
| Currency as CountryPack attribute; pass-through; no Customer wallet | **Landed** |
| Tax / e-invoicing **empty hooks only** (LATER) | **Did not land** — no reserved ledger/tax hook |

Payout Clock, integer XOF, `pending settlement` are spine/PRD closes, not addendum misses.

---

## 5. NEXT / LATER — did not land in `Deferred`

Addendum §4–§5 exist so later work does not quietly pull them into MVP. Spine `Deferred` is a **subset**.

### NEXT (§4) absent from Deferred

- Escalating dunning + payday windows (~26th–5th) — spine’s “late-fee compounding copy” is a different item
- Family / group pay + share-pay-link
- Consumer home of all platform Subscriptions
- One-off invoices as a **named product**
- PWA + shop QR + paper membership card
- MRR / churn / failed-collection analytics suite
- Ouaga-only field docs (Laravel/Flutter samples *are* listed; docs are not)
- Dispute freeze (“I did not get service”)
- Mooré / Dioula keywords
- Guild / association launch
- Thermal / printed PAID receipt
- MNO daily-limit display before confirm
- Known-Customer whitelist

### NEXT present in Deferred

Moov+Wave live + in-session failover · platform USSD renderer · WhatsApp/IVR · collector mode · vertical templates · productized payout-to-own-MoMo-code · Laravel/Flutter samples.

### LATER / WON’T (§5) absent from Deferred

- Tontine / e-susu
- B2B supplier credit
- NGO / government / institutional payers
- Customer-invites-Merchant growth
- Split / franchise payouts
- Credit scoring
- Recurring payouts / salary-like out
- Savings-toward-due envelopes
- Prepaid auto-refill Merchant wallets
- Tax / e-invoicing (empty hooks only)
- White-label domains
- Per-country aggregators **as a product** (Orange counterparty-as-config is not this)
- Voice-mandate as a legal instrument
- Women-support line as a standalone product
- Super-App Customer wallet (Customer wallets prevented; Super-App not named)
- Native apps as a won’t (Flutter *samples* are Deferred; native Merchant apps from §3 are not)
- Card PAN checkout as an explicit won’t (Locale forbids card-network *marks* only)
- Compliance calendar (BF license vs UEMOA passporting) as an MVP artifact
- `bmad-deep-recon` TAM study (not an architecture item)

### LATER present

Live PI-SPI / Request-to-Pay · second-country production pack · payment-institution / e-money license posture · legal float · auto-debit marketing (as AD-4 Prevents, stronger than defer).

### Rejected alternatives (§3) not encoded as Prevents / Deferred

- Demand-side Customer-invites-Merchant
- Tontine / Super-App / credit bureau
- Guilds as a built product surface
- Treating Flutterwave/Paystack “subscriptions” (card token) as the model
- Native Merchant apps (mobile web first)

---

## Contradictions

These are text-level disagreements, not mere omissions.

### C1 — Identity primary key (HIGH)

- **Addendum §2:** “MSISDN + country code as primary key.”
- **Spine AD-6 / Conventions:** “Public IDs are ULID; country is an attribute, not an ID prefix.”

If the addendum meant *resource IDs*, the spine contradicts it. If it meant *Customer uniqueness* `(country_code, MSISDN)`, the spine never states that uniqueness — ClientRecord identity is unspecified. Either reading is a hole: builders can mint ULID-only customers with duplicate MSISDNs, or reintroduce country-prefixed IDs the spine forbade.

### C2 — Ledger event catalog (MED)

- **Addendum:** `refunded` is a first-class gateway-agnostic event; no `reversed`.
- **Spine:** `reversed` is live; `refunded` is reserved / no Merchant refund.

Compatible with PRD Rail Reversal and no-refund MVP, **not** compatible with “land the addendum catalog as written.” Events table also emits `rail.reversed` without a `*.refunded` topic.

### C3 — Scheme layer collapsed (MED)

- **Addendum architecture bet:** Country → **scheme** → Rail → Adapter. PI-SPI is a scheme behind a licensed participant.
- **Spine:** two ports (`PaymentRailPort`, `DirectDebitPort`). Scheme is not a type. Request-to-Pay is a Deferred bullet on the same vacant port.

Risk: PI-SPI / RtP gets implemented as “just another Rail” or as a toggle on reminder-then-pay consent (AD-4/AD-11 try to prevent the consent reuse; they do not prevent the layering mistake).

### C4 — Five primitives vs entity graph (LOW–MED)

Addendum: Subscription = Promise + schedule + payment instruction + record + reminder; everything else is packaging. Spine ERD: Plan, StandingAuthorization, Obligation, CollectionAttempt, LedgerEntry — no primitive map. Not a hard conflict, but a builder can invent a sixth aggregate or collapse reminder into Obligation.

### C5 — USSD renderer vs rail-wallet USSD (LOW, implementation trap)

Addendum allows operator USSD **now** as the wallet confirm. Spine defers “Live platform USSD shortcode renderer” and forbids platform shortcode. Texts do not formally contradict, but without the rail-wallet sentence, “no USSD in MVP” is an easy misread of AD-3/Deferred.

### Non-contradictions (do not treat as conflicts)

- Addendum illustrative names (`createSession`) vs spine `POST /v1/checkout/sessions` — addendum says names are not contract.
- Spine stack, Fly.io `ams`, BullMQ queue names, take-rate, KYC, Payout Clock — PRD/architecture-owned; addendum does not specify them.
- `reversed` vs PRD — PRD wins; this is addendum-catalog drift, not a PRD break.
- Addendum “not a second source of truth for in-scope FRs” — missing NEXT/LATER lines are hygiene, not missing MVP scope.

---

## Recommended spine patches (not applied)

1. **Transport AD:** Orange/Moov collect is rail-owned payer approve (USSD OTP / in-app / USSD-push); platform never hosts that USSD; Hosted Checkout and SMS PAY await rail callback. Record SMS fork (SVA declaration vs Orange SMS API). Add `GET /v1/checkout/sessions/:id`.
2. **Versioning AD:** define dated-version mechanism (recommend stay on `/v1` + additive statuses; breaking rename → `/v{date}` + sunset). Version webhook envelopes separately from HMAC `v1=`.
3. **Rails:** one paragraph of 2026 landscape (WebPay-omit-BF, Moov aggregator, Wave Checkout invite-only). Name scheme as a type or explicitly reject it: “PI-SPI/RtP attaches through `DirectDebitPort` via a licensed participant; never a third Session type.”
4. **Ledger:** publish the closed catalog (`reversed` live, `refunded` reserved) and map five primitives → aggregates. State ClientRecord uniqueness `(merchant_id, country_code, MSISDN)` and keep public IDs as ULID (resolves C1 without undoing AD-6).
5. **Deferred:** append the missing NEXT/LATER / rejected-alt rows so they cannot sneak into MVP stories.

---

## Sources

- `_bmad-output/planning-artifacts/prds/prd-bf-recurring-payments-2026-09-17/addendum.md`
- `_bmad-output/planning-artifacts/architecture/architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md`
- Spine `.memlog.md` consulted only to confirm WebPay-omit-BF lives in memlog, not in the spine (not used as a third source of product law).
