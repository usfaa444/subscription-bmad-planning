---
title: Input reconciliation — PRD → architecture spine
input: _bmad-output/planning-artifacts/prds/prd-bf-recurring-payments-2026-09-17/prd.md
target: ARCHITECTURE-SPINE.md
created: 2026-09-17
verdict: partial-bind
---

# Reconcile: `prd.md` → architecture spine

## Method

Read both files in full. The spine is a build substrate: it should fix only the calls two independently-built units could choose incompatibly. This review flags (1) PRD requirements that never became an `AD` Rule, convention, or Deferred item — especially quiet tone, constraint, and FR/NFR areas the AD structure dropped — and (2) contradictions between the two documents.

**Landed** means an enforceable Rule, convention, or named Deferred item would keep two builders aligned. A Capability Map row that cites an FR but points at an AD whose Rule does not state that FR’s consequence is **map-only**, not landed.

Frontmatter claims `binds: [FR-1–FR-56, NFR-1–NFR-12]`. That claim is overstated. The money/consent/rail core is bound. The SMS OS, merchant sitting, tone, audit completeness, and several domain edges are not.

---

## Well retained (no action required)

These PRD load-bearing calls actually have Rules.

| PRD element | Spine anchor |
| --- | --- |
| Dual-channel OS, one product, hexagonal modular monolith | AD-1; paradigm; UJ dual-channel in map |
| One Consent State Machine, two skins; no Customer passwords; no platform shortcode | AD-3 (FR-10, FR-12, FR-49) |
| Reminder-then-pay only; no auto-debit copy; vacant Direct Debit; new SA text later | AD-4, AD-2 (FR-11, FR-20, FR-48) |
| Pay-time method; live Orange; Moov/Wave stubs; cash is not a Rail | AD-5 (FR-8, FR-19, FR-21, FR-22, FR-23) |
| CountryPack; BF-only production; one Merchant account; ULID + `country_code` | AD-6 (FR-3, FR-45–FR-47, NFR-1, NFR-9) |
| Ledger events; no Customer wallet; pass-through; T+1 target; cash = already received; `refunded` reserved | AD-7 (FR-34–FR-39) |
| Attempt idempotency token; Session `Idempotency-Key`; signed at-least-once webhooks; replay must not re-debit | AD-8 (FR-16, FR-24, NFR-3, NFR-10) |
| Obligation minted only after SA confirm; reminder budget 1 + 48h; Cancel ≠ Session `cancelled` | AD-9 (FR-9, FR-53, FR-51) |
| Take-rate 0% first live MoMo then 2.5% half-up; 0% fail/cash/Sandbox; reversal claws fee, does not reset first-success | AD-10 (FR-40–FR-42) |
| KYC gate blocks **payouts** not invoicing/SMS; 500k + dest-change; informal start | AD-10 (FR-43, FR-44) |
| Customer Magic Link/OTP 15m; Merchant MSISDN OTP; Staff PIN privileged set; hashed keys shown once; sandbox isolated; MSISDN last-4 | AD-11 (FR-4, FR-5, FR-12, FR-14, FR-17, FR-50, NFR-6 partial, NFR-12 partial) |
| `/v1`; stable statuses including `settled`; French Merchant `message`; Session TTL 24h | AD-12 (FR-14, FR-16, §16) |
| BullMQ queue names; integer money; UTC + CountryPack TZ; SMS verb list accent-insensitive; SOLDE ≠ platform balance | AD-13 (FR-29, FR-30 verbs, FR-35–FR-36, FR-47, NFR-2 partial) |
| Fly.io `ams`; `local` / `sandbox` / `production`; no in-country-compute assumption; operator Rail visibility | AD-14 (NFR-2, NFR-4, NFR-9, §14, §17 residency) |
| French-first checkout/SMS; no `$`; no card-network marks | Conventions + AD-12 (FR-15, NFR-1) |
| Glossary entity names in code | Consistency Conventions |
| Deferred: live Moov/Wave, second-country pack, SMS vendor, Orange counterparty, legal float, KYC ops list, email backup, late-fee compounding, USSD renderer, PI-SPI live, microservices, e-money license | Matches PRD §5–6, §8.1–8.6 |

---

## Gaps — what the PRD said that the spine did not land

### Gap 1 — SMS OS behavioral contract (FR-18, FR-29–FR-33, FR-31 safety)

**PRD:** SMS is the OS. Invoice fields are specified (what / to whom / how often / how much / by when / how to pay). STOP is **not** Cancel; transactional billing SMS continue; consent discloses the mapping; a second STOP within 24h replies with ANNULER instructions only; after STOP, unsolicited AIDE is not sent; unknown text gets an AIDE-equivalent (except after STOP, only if the Customer originated). Invite is transactional and not re-sent after STOP unless a new consent path. Success produces **only** the SMS Receipt (FR-32). Failure SMS must not claim the Subscription is cancelled. Named shop chrome on invite/invoice/receipt/failure/AIDE/SOLDE (FR-13, FR-33). Cash Mark-paid still sends a Receipt after STOP (FR-23).

**§12 Safety:** “STOP cannot silently strand an owed Obligation without transactional notice (FR-31).”

**Spine:** AD-13 lists the seven verbs and says SOLDE never shows a platform-held balance. AD-3 calls SMS a renderer. AD-9 caps invoice/reminder count. Capability map assigns FR-18, FR-29–FR-33 to `SmsPort` + `sms.send` under AD-2/AD-13.

**Dropped:** Every STOP mapping rule. Silence-after-success. Invoice field contract. Named-SMS requirement. Unknown-text → AIDE. Invite-vs-STOP. Failure-does-not-mean-cancelled. Receipt-still-sends-after-STOP. The §12 STOP safety guardrail.

**Why it matters for a spine:** Two builders will invent incompatible STOP machines, dunning volume, and receipt rules. That is exactly the “could they choose incompatibly?” test. AD-4 binds “safety guardrail” but only the debit-without-confirm half.

**Severity:** High.

**Suggested carry-forward:** New AD (or extend AD-13) for the SMS OS: transactional set vs STOP, STOP ≠ ANNULER, 24h second-STOP, no unsolicited AIDE after STOP, silence on success, invoice six-field contract, shop-name on every transactional SMS, fail-closed copy (failure ≠ cancelled). Bind FR-18, FR-29–FR-33, §12 STOP sentence.

---

### Gap 2 — Tone, aesthetic, and shame-free constraints (NFR-11, §19, SM-C4, emotional JTBD)

**PRD §19:** French, official-enough receipt, named SMS, neighbourhood-level trust — not neobank English, not card-checkout skeuomorphism. Silence after success; ping on failure or change. Shame-free private reminder. Cash is an on-ramp, not an apology. Developer docs should feel like Stripe, not a bank PDF — French examples in XOF.

**NFR-11:** Dunning and failure copy is private and factual. No public defaulter wall, no shaming ranking in MVP.

**UJ-2 / §2.1:** Shame-free private reminders; a receipt that looks official; Customer can show one SMS to a spouse or parent; Merchant looks organized.

**SM-C4:** Do not treat Cash Mark-paid share as failure.

**Spine:** Conventions lock French-first, no `$`, no card marks. Capability map says NFR-11 → “copy in CountryPack; no ranking tables” governed by **AD-4**. AD-4’s Rule is reminder-then-pay and auto-debit vocabulary only. No AD Rule forbids a defaulter wall, ranking table, celebratory SMS mesh, or cash-shaming copy. Neighbourhood as trust chrome (FR-13) is absent. Official-enough receipt is absent. Stripe-shaped DX voice is absent (Deferred lists Laravel/Flutter samples, not docs tone).

**Dropped:** The qualitative product that the PRD treated as binding: private factual dunning, silence, official receipt, neighbourhood match, cash-without-apology, Stripe-like French DX.

**Severity:** High for quiet requirements. The AD structure is the exact mechanism that drops tone.

**Suggested carry-forward:** Either a tone/copy AD (CountryPack strings: no ranking, no defaulter wall, silence on success, cash-neutral labels, shop+neighbourhood chrome, official receipt) or an explicit Deferred “UX owns copy; spine forbids ranking/defaulter surfaces in `apps/portal`.” Do not leave NFR-11 as a map line pointing at AD-4.

---

### Gap 3 — Merchant sitting, portal IA, and “no forced extra modules” (FR-2, FR-7, §20, UJ-1)

**PRD §20:** First sitting is Signup (FR-1) → Plan (FR-6) → Payout Destination (FR-38) → Client Record + Invite (FR-7, FR-18); Staff PIN offered before leaving the machine. Portal top-level: Accueil (Receivables Book) · Clients · Plans · Encaissements · Paramètres · Développeurs. Action homes are specified (cash/retry on Encaissements **and** the open Obligation row; lifecycle on Client Record; KYC hold banner on Accueil **and** Paramètres). French nav labels. Receivables Book shows **pending consent**, not an owed amount, until confirm.

**FR-2:** After signup, happy path is Plan → Client Record → send invite, with **no forced extra modules**. Time-to-first-invite is SM-1.

**FR-7:** Name + MSISDN only; duplicate MSISDN for the same Merchant rejected with an explicit message (**no silent merge**); Invite SMS only after Merchant **confirms send**.

**FR-1:** Signup is display name + MSISDN + **neighbourhood**, without NINEA/RCCM/tax ID.

**UJ-5 / §20:** KYC hold is a banner she understands — not a silent freeze.

**Spine:** AD-6/AD-11 cover one account, informal KYC, Staff PIN action list. AD-9 covers pending-consent vs Obligation. Structural seed has `apps/portal`. No sitting sequence. No IA. No “no forced extra modules.” No neighbourhood signup field. No duplicate-MSISDN rule. No confirm-before-invite. No KYC banner (only “blocks payouts”). Staff PIN “recommended during first sitting” / “before leaving the machine” not stated (AD-11: optional-to-set, mandatory-when-set).

**Why it matters:** Two portal builders can ship a billing-admin IA (forced Developers, KYC, or analytics before first invite), silently merge Client Records, or freeze the book without a hold explanation. FR-2 is a product constraint with architectural teeth: module gating after signup.

**Severity:** High for the offline wedge (Awa). Medium if UX is accepted as the IA owner — then the spine must **Deferred** the sitting/IA and still lock the domain edges (duplicate MSISDN, confirm-send, neighbourhood on Merchant, no extra-module gate).

**Suggested carry-forward:** Domain AD or convention: Client Record unique `(merchant_id, msisdn)`; invite requires explicit send; Merchant signup fields = name + MSISDN + neighbourhood; no post-signup capability gate before first invite. IA/nav/banner → UX companion or a short surface-map AD. UJ-1 edge (PIN before leaving the machine) is UX unless the spine wants a first-sitting prompt invariant.

---

### Gap 4 — Domain edges two builders will implement differently

These are testable PRD consequences that never became Rules. “Follow PRD semantics” (AD-9) is not a Rule.

| PRD rule | Why a spine should bind it | Spine today |
| --- | --- | --- |
| **FR-6** Plan: weekly + monthly (+ optional N-day); late-fee none / fixed XOF / %; overdue fee appears on next SMS Invoice; collection preferences do **not** lock a Rail; reminder cadence is FR-53, not a Merchant-authored schedule | Fee and schedule policies will fork | AD-5 prevents Plan-locked Orange only. Frequencies, late-fee enum, “no free-form reminder cadence” absent. Late-fee compounding is Deferred (UX) — the **enum itself** is not. |
| **FR-9** No separate one-off invoice product IA | Second product surface | Not prevented |
| **FR-12** Shared-phone risk disclosed in consent; enrolled MSISDN is the consenting party; **Rail-wallet MSISDN may differ**; no share-pay-link / second-payer / beneficiary fields in MVP | Payment identity vs enrolled identity; family-pay creep | AD-11: no password, 15m TTL. Second-payer non-goal and wallet-MSISDN-may-differ **absent** |
| **FR-14** Session create requires Plan ID; **no free-form line items** | Second invoice model | Idempotency + TTL only |
| **FR-16** Merchant can **rotate** the webhook signing secret from Developers | Secret lifecycle | AD-8: HMAC + replay. Rotation absent |
| **FR-23** Cash Mark-paid cannot apply to an already `collected` Obligation | Double-close | Not stated |
| **FR-25** Failed Attempt does not cancel Obligation or Standing Authorization; notify failure, not cancellation | Honest book | Folded into AD-9 “PRD semantics” |
| **FR-26** Retry Inbox fields + retry = new Attempt **and** new reminder/PAY path | Surface + side effect | New token in AD-8; inbox contract absent |
| **FR-27 / FR-28** Pause: open Obligation remains; Resume = REPRENDRE or Client Record. Skip closes as skipped, 0% Take-rate, next cycle still mints | Lifecycle | AD-9 names Pause/Skip; details not ruled |
| **FR-40** Partial Rail success: fee on collected XOF only; remainder stays open on a **new** Attempt. Late fee is inside collected amount if collected. Failed settlement after `collected` does **not** void Take-rate unless FR-56. Policy documented as configurable later; MVP sole live value 2.5% | Fee/ledger fork | AD-10 has 2.5% and first-success; partials, late-fee-in-collected, settlement-does-not-void **absent**. “Configurable later” absent (mild tension with hardcoded 2.5%) |
| **FR-42 / §4.5 / §8.7** No per-SMS invoice; SMS bundled into MoMo Take-rate; cash-only books are an accepted subsidy until first Orange | A builder can add SMS billing | AD-10: 0% on cash. Bundling / subsidy not stated; **§8.7 not in Deferred** |
| **FR-51** After Cancel: no new Obligations; SA **revoked**; billing SMS **stop**; open Obligations remain due | Cancel machine | Event name landed; revoke + SMS-stop + open-remain are “PRD semantics” |
| **FR-52** Remove Client Record **Cancels every Subscription** with that Customer for this Merchant; historical receipts/ledger remain; SOLDE after removal = no open Subscription | Remove vs Cancel collapse | AD-11 lists remove as a PIN action only |
| **FR-53** A Session that collects now may mint **and** collect the current Obligation in the same Session | Birth vs collect race | AD-9 mint-after-confirm; same-session collect-now not stated |
| **FR-54** Equal no-browser path: PAY → OTP → Rail-wallet confirm (operator USSD/in-app) | Path treated as degraded | AD-3 “SMS verbs are renderers”; NFR-8 “equal, not superior” not in any Rule |
| **FR-55** Customer cannot Write-off; Customer gets a transactional notice; cannot apply to `collected` | Actor + notice | PIN on Write-off only |
| **FR-56** Obligation re-opens for reversed XOF; Customer **and** Merchant notified; Payout Clock **must not** show Customer-paid for the reversed Attempt | Clock lie after reversal | Fee claw + `reversed` event; clock-clear and notices absent |
| **FR-5** Live keys may collect only after Orange live credentials/onboarding; Developers surface itself is **not** gated on first success | Key vs Rail onboarding | Sandbox isolation landed; live-key-onboarding gate and “Developers always on” not explicit |
| **NFR-6** Staff PIN cannot be recovered in plaintext | PIN storage | API keys hashed/shown-once; Staff PIN recoverability absent |
| **NFR-2** Hosted Checkout first contentful interaction **< 8 seconds** on mid-3G QA | Weight budget two checkout apps will miss | Map: “checkout weight”; no number, no budget |
| **NFR-8** SMS verbs are not a degraded mode; hosted web is equal, not superior | Channel hierarchy | Map only |
| **§16** Retrieve Session / Obligation / Collection Attempt; **expire** Session | Public contract | Only `POST /v1/checkout/sessions` |
| **FR-50** Sandbox can simulate `failed` for Retry Inbox testing | Sandbox completeness | `SandboxRail` named; `failed` simulation not required |
| **Counter-metrics SM-C1–C3** Do not maximize SMS volume, month-one Take-rate, or Pause/STOP as vanity | Optimization pressure | Reminder cap only (SM-C1 partial). Others absent (acceptable if treated as product, not architecture) |

**Severity:** High as a cluster. Highest individual items: FR-12 second-payer / wallet-MSISDN split, FR-40 partials, FR-52 remove=cancel-all, FR-56 clock-clear, FR-6 late-fee enum, FR-7 uniqueness.

**Suggested carry-forward:** Promote the table’s High rows into AD-9 / AD-10 / AD-11 / AD-12 Rules. Stop using “follow PRD semantics” as a bind.

---

### Gap 5 — Privacy, audit, data governance, compliance (NFR-7, NFR-12, §12–§13, §17)

**NFR-7:** No Customer **email harvest** in MVP. Retention of **SMS bodies** and consent text sufficient for dispute/audit. Documented in a **French privacy notice**.

**NFR-12:** Standing Authorization text version, timestamps, Collection Attempt tokens, **Cash Mark-paid actor**, and **payout-destination changes** retained for audit.

**§12 Privacy:** Shared-phone disclosure. No Customer wallet dossier beyond receivables need.

**§13:** WAEMU/BCEAO — do not issue e-money; prefer **agent / technical integrator** posture via the Orange Adapter’s licensed counterparty. Platform KYC gate is a **payout-risk control, not an EME-compliance claim**. ARCEP transactional vs SVA (STOP is the product expression). **BF entity first; per-country entity on expand.** Copy law = honest reminder-then-pay.

**§17:** MSISDN, consent text, and ledger are **sensitive**. Retention = life of the Merchant relationship **plus a documented minimum**. **No sale of Customer ledgers.**

**Spine:** AD-11 stores consented text version and masks MSISDN in prod logs. Conventions: consent bodies not in debug logs. AD-7 no Customer wallet. AD-14 CountryPack must not assume BF compute. Deferred: legal float, e-money license posture.

**Dropped:** Email-harvest ban. French privacy notice. SMS-body retention. Cash Mark-paid **actor**. Payout-destination change audit. Shared-phone disclosure in consent (FR-12). Data classification. Retention minimum. **No sale of ledgers.** Agent/integrator posture (vs accidental PI/EME marketing in ops copy). KYC-is-not-EME-claim. Legal-entity-per-country. ARCEP/SVA named as the reason STOP exists.

**Severity:** High for NFR-7 email, NFR-12 actor/payout-dest audit, §17 no-sale. Medium for regulatory posture (some is Deferred license; the **disclaimer** that FR-43 is not EME KYC is a product-architecture claim).

**Suggested carry-forward:** Extend AD-11: no Customer email field in MVP; audit columns (consent version, attempt token, cash actor, payout-dest changes); SMS body retention sufficient for NFR-12; Staff PIN unrecoverable. Convention or AD: no sale/export of Customer ledgers as a product. One Deferred or AD sentence: operate as software + licensed-Rail integrator; FR-43 is platform payout-risk, not EME tiers; legal entity is BF-first.

---

### Gap 6 — Operational product constraints (§14, NFR-4 already partial)

**PRD §14:** Publish Payout Clock targets in **Merchant-facing help**. Support = Merchant Portal + AIDE SMS; **no voice support line as a product**. Orange/aggregator outage: **reminders may send; collects fail closed; Obligations remain**.

**Spine:** AD-14 is deploy/observability. AD-4/AD-9 imply fail-closed debit but do not state rail-outage behavior. No support-channel constraint. No “publish the T+1 target in help.”

**Severity:** Medium. Fail-closed on Rail outage belongs in AD-4 or AD-9. Voice-line non-goal and help-copy can be Deferred to UX/ops if named.

---

### Gap 7 — User journeys the spine under-binds

AD-3 binds **UJ-1–UJ-3** only.

- **UJ-4** (failed Friday collect): Retry Inbox, last Rail status, cash, Write-off, honest book — scattered across AD-8/AD-9/AD-11 without the journey or FR-26 surface contract.
- **UJ-5** (soft KYC): “she understands why money is held” — banner, not silent freeze — dropped (see Gap 3).

Non-users (§2.2) are mostly in Deferred or AD Prevents (cards, English-first, live Moov/Wave, native apps, PI/EME, NGO/B2B/tontine, Super-App wallet, second-country production). **Not named as non-users / Prevents:** demand-side “Customer invites Merchant,” consumer subscription home, credit scoring (NFR-11 ranking is the nearest hook).

Success metrics SM-1–SM-10 are product, not spine — acceptable. SM-4/SM-8/SM-9 are already implied by AD-8/conventions/AD-10.

---

## Contradictions

### C1 — Dangling `AD-18`

AD-4 Rule: “Enabling it later requires a **new** `StandingAuthorization` text version **(AD-18)**.”

The spine has AD-1–AD-14 only. AD-18 does not exist. The requirement itself is in AD-4 and FR-11; the citation is a broken bind.

### C2 — Frontmatter and Capability Map overclaim

`binds: [FR-1–FR-56, NFR-1–NFR-12]` plus map rows for every FR/NFR. Several map governances are false:

| Map claim | Actual Rule |
| --- | --- |
| NFR-11 → AD-4 | AD-4 is auto-debit copy, not shame-free / no ranking |
| NFR-8 → AD-13 | AD-13 lists verbs and queues, not “equal not degraded” |
| NFR-2 → AD-3, AD-14 | Neither states 3G weight or the 8s target |
| FR-18, FR-29–FR-33 → AD-2, AD-13 | Verbs listed; STOP/silence/invoice fields not ruled |
| FR-27, FR-28, FR-51–FR-56 → AD-4 **and** AD-9 | AD-4 is the wrong governor for Pause/Skip/Write-off |
| FR-1–FR-5 → AD-6, AD-11, AD-14 | Neighbourhood signup, sitting, Developers-always-on, live-key-onboarding not ruled |

This is a contradiction of **coverage claim vs content**, not of two Rules fighting.

### C3 — No true Rule-vs-PRD inversions on money/consent

Checked and **not** contradictory (aligned or additive):

- AD-12 `settled` vs FR-16 “at least” five Session statuses — §16 already lists `settled`. Additive.
- AD-11 Staff PIN set includes Cancel / remove / Write-off — PRD specifies those on FR-51/52/55, not the FR-4 consequence list. Compatible extension.
- Conventions put `Idempotency-Key` on Attempt confirm as well as Session create — compatible with FR-24 tokens.
- AD-7 cash sets Customer-paid immediately — matches UJ-4 / FR-23+FR-35 together.
- Hosting on Fly.io `ams` vs §17 “may be outside BF” — allowed.
- ULID, Nest/Next/BullMQ, webhook header names — architecture filling §16 “exact paths are architecture.”
- Hardcoded 2.5% in AD-10 vs “configurable later” in FR-40 — MVP-correct; later configurability is a dropped note, not an inversion.

---

## Non-goals the spine never named

Acceptable omissions at spine altitude are marked. Omissions that let a builder add a surface are not.

| PRD §5 / §6.2 non-goal | In spine? | Risk if omitted |
| --- | --- | --- |
| Payment institution / e-money issuer | Deferred (posture) | Low if Deferred stays |
| Customer wallet / Super-App | AD-7 Prevents | OK |
| Auto-debit / merchant-initiated pull | AD-4 | OK |
| Live PI-SPI / R2P | AD-2, Deferred | OK |
| Merchant-initiated refund | AD-7 `refunded` reserved | OK |
| Live Moov/Wave / in-session failover | AD-5, Deferred | OK |
| Second-country production pack | AD-6, Deferred | OK |
| Native apps | Implicit (web apps only); not Prevented | Low |
| Card / English-default / `$` | Conventions | OK |
| Named one-off invoicing product | **Absent** | Medium — second IA |
| Family/group pay, collector mode | Family-pay **absent**; collector in Deferred | Medium (FR-12) |
| WhatsApp / IVR / live USSD | Deferred | OK |
| Demand-side Customer-invites-Merchant | **Absent** | Low |
| Tontine, credit scoring, B2B, NGO, split payouts, salary outbound, savings | **Absent** | Low–medium; credit scoring overlaps NFR-11 |
| Consumer home of all subscriptions | **Absent** (AD-3 no Customer password home is close) | Low |
| PWA / QR / paper card, thermal receipt, Mooré/Dioula, MRR suite | Some in Deferred | OK |
| Voice support line | **Absent** | Low (ops) |

---

## Open questions vs Deferred

| PRD §8 | Spine |
| --- | --- |
| 1 Legal float | Deferred — OK |
| 2 KYC checklist / review SLA | Deferred — OK |
| 3 Orange counterparty | Deferred + AD-5 — OK |
| 4 Late-fee compounding copy | Deferred — OK (base enum still missing; Gap 4) |
| 5 Webhook numeric SLA | AD-8 decided the schedule; extra SLA Deferred — OK |
| 6 Merchant email backup | Deferred — OK |
| 7 Cash-only SMS subsidy | **Not in Deferred** — Gap 4 |

---

## Verdict

**Partial bind.** The spine correctly locked the hexagonal monolith, reminder-then-pay, vacant Direct Debit, CountryPack, ledger/pass-through, take-rate/KYC payout gate, idempotent attempts, obligation birth, sandbox isolation, and `/v1` status vocabulary. That is the money-and-consent core.

It did **not** land the SMS OS (especially STOP and silence), shame-free/official/neighbourhood tone, merchant sitting and portal IA, or a set of domain edges (duplicate Client Record, second-payer ban, late-fee enum, remove=cancel-all, reversal clock-clear, fee partials, SMS bundling). It overclaims a full FR-1–56 / NFR-1–12 bind, and AD-4 cites a nonexistent AD-18.

Those dropped items are mostly quiet — tone, constraints, FR/NFR areas — which is the failure mode this reconcile is meant to catch: the AD skeleton kept the rails and dropped the OS around them.

**Recommended spine edits (for the parent Finalize, not this review):** fix C1 (drop or add AD-18); narrow frontmatter `binds` to what Rules actually state; add an SMS-OS AD; bind NFR-11 in a Rule (or Deferred to UX with portal Prevents); lift Gap 4 High rows into AD-9/10/11/12; extend AD-11 for audit/email/PIN; Deferred §8.7 SMS subsidy and §14 support/outage if not Ruled.
