---
title: Input reconciliation — brief addendum
created: 2026-09-17
input: /workspace/projects/bf-recurring-payments/_bmad-output/planning-artifacts/briefs/brief-bf-recurring-payments-2026-09-17/addendum.md
against:
  - prd.md
  - addendum.md
---

# Reconciliation: Product Brief Addendum (`brief-bf-recurring-payments-2026-09-17/addendum.md`)

## 1. Source extract (binding for this pass)

| Block | Brief addendum role |
| --- | --- |
| §1 Spine & directions | Dual-channel receivables OS; rail adapters + vacant DirectDebitPort + CountryPack; reminder-then-pay; never market auto-debit; GTM wedge; deferred moat (on-time collection, mandates sandbox). |
| §2 Theme map | Vocabulary only — not a backlog. |
| §3 Opportunity matrix | **MVP MUST** defers to brief Scope (not this file). **NEXT SHOULD** table (19 rows). **LATER / WON’T** prose list. USSD renderer note: first shipped paths = web + SMS; live shortcode NEXT. |
| §4–§5 Personas & landscape | Depth for PRD/addendum; light competitive/rail scan 2026-09-17. |
| §6 Economics & legal | Open table (take-rate %, SMS bundling, cash fees, settlement, entity). |
| §7–§8 Identity, consent, failure | MSISDN PK; household NEXT; French-first; failure inbox; SMS verbs; idempotency. |
| §9 Rejected alternatives | Why-not-now table. |
| §10 Critical insights | Same loop two skins; literacy = one protocol; cash on-ramp; trust/settlement over features. |
| §11 Open questions for PRD | Eight prompts (take-rate, Orange path, pass-through legality, KYC threshold, SMS/STOP, USSD timing, payout SLA, portal+API same increment). |

## 2. Mandatory checks

### 2.1 NEXT → MVP FR promotion (silent promotion scan)

| Brief addendum NEXT item | MVP FR? | Disposition |
| --- | --- | --- |
| Moov + Wave + in-session failover | No live collect; FR-22 design + §5/§6.2 Out of Scope | **OK** |
| WhatsApp receipts + IVR | §6.2 Out of Scope | **OK** |
| Escalating dunning + payday windows (~26th–5th) | Basic retry inbox FR-26, NFR-11 only; no payday windows | **OK** |
| Collector offline cash + cash-day calendar | Cash Mark-paid FR-23 only; no collector/calendar | **OK** |
| Family / group pay + share-pay-link | §5/§6.2 Out of Scope | **OK** |
| Consumer home (all platform subscriptions) | §6.2; §22 IA notes NEXT | **OK** |
| One-off invoices as named product | FR-9 NON-GOAL | **OK** |
| Vertical plan templates | FR-6 `[ASSUMPTION]` clone-a-plan / N-day; no template marketplace | **OK** |
| PWA + shop QR + paper membership card | §6.2, §11 NEXT | **OK** |
| Payout to own MoMo / Wave merchant code (productized) | FR-38 pass-through Orange MSISDN only; §6.2 | **OK** (baseline payout ≠ NEXT productization) |
| MRR / churn / failed-collection analytics | §6.2 Out of Scope | **OK** |
| Laravel / Flutter samples; Ouaga / Abidjan docs | §6.2 Out of Scope; PRD addendum §4 NEXT | **OK** |
| Dispute freeze | §6.2 Out of Scope | **OK** |
| Mooré / Dioula keywords | §6.2 Out of Scope | **OK** |
| Guild / association launch | §5 non-goal; addendum §3 rejected | **OK** |
| **One merchant account spanning portal and API** | **FR-3, §6.1** | **Allowed PO override** (brief addendum NEXT row 85; `.memlog.md` + PRD addendum §10 headless locks) |
| Thermal / printed PAID receipt | §6.2 Out of Scope | **OK** |
| MNO daily-limit display before confirm | §6.2 Out of Scope | **OK** |
| Known-customer whitelist | §6.2 Out of Scope | **OK** |
| Live USSD `*shortcode#` (§3 renderer note) | FR-49 model only; §6.2 Out of Scope | **OK** |
| Household payer ≠ beneficiary (§7) | Not an FR | **OK** |

**Pause / Skip / Staff PIN:** Not listed in brief addendum NEXT table; brief Scope + `.memlog.md` keep them in MVP → FR-27, FR-28, FR-4. **Not** treated as silent NEXT promotion.

**Verdict:** No disallowed silent NEXT→FR promotions. One explicit override: portal + API on one account (FR-3).

### 2.2 LATER / WON’T → MVP FR promotion

All brief addendum §3 LATER bullets (PI-SPI live debit, second-country production, tontine, B2B credit, NGO/gov, demand-side growth, split payouts, credit scoring, payment-institution license, recurring payouts, savings envelopes, prepaid merchant wallets, tax/e-invoicing hooks, white-label, per-country aggregators product, voice-mandate, women-support line, Super-App wallet, native apps, card PAN, platform-held customer balances, auto-debit marketing/copy) appear in PRD §5 Non-Goals and/or §6.2 Out of Scope and/or PRD addendum §5 LATER. No live FR implements them.

**Verdict:** **OK**

### 2.3 Technical-how placement (brief addendum → PRD addendum)

| Brief addendum depth | Expected home | Actual |
| --- | --- | --- |
| Architecture bet, five primitives, ledger events, adapter counterparty paths | PRD addendum §2 | **Present** |
| Rejected alternatives | PRD addendum §3 | **Present** (+ one extra row on card-token “recurring”) |
| NEXT / LATER catalogs | PRD addendum §4–§5 | **Present** (portal+API row omitted — promoted to MVP) |
| Economics / legal open notes | PRD addendum §6 + PRD §12 locked policy | **Mostly present** |
| Landscape / PI-SPI / PSP citations | PRD addendum §7 | **Present** |
| Persona color | PRD addendum §8 + PRD UJs | **Present** |
| Theme map | PRD addendum §9 | **Present** |

**Leak into `prd.md` (capabilities doc):** §10 Why Now and §12 monetization competitive band (~3.5–4.5%) repeat landscape-level claims that brief addendum §5 and PRD addendum §7 already own. Glossary/FR-21 Rail Adapter wording is outcome-level (acceptable). §17 dependencies stay capability-level.

**Verdict:** **Mostly OK** with one placement gap (see GAP-2).

### 2.4 PO override registry (allowed)

| Brief addendum | PO lock | PRD |
| --- | --- | --- |
| One merchant account portal + API — NEXT (“shipping both keys can trail”) | `.memlog.md` decision; addendum §10 inputs | FR-3, §6.1, UJ-1/UJ-3 |

Documented; not a reconciliation failure.

## 3. Coverage summary (spine & insights)

| Brief addendum element | PRD / addendum | Status |
| --- | --- | --- |
| Dual-channel, one consent machine | Vision §1; FR-10; addendum §2 spine | **Captured** |
| Reminder-then-pay; no auto-debit marketing | FR-11, FR-20; §5 | **Captured** |
| Orange live; Moov/Wave designed | FR-21–22; §6.2 | **Captured** |
| Vacant Direct Debit Port / PI-SPI socket | FR-48; addendum §7 | **Captured** (live debit out) |
| Cash mark-paid wedge | FR-23; §6.1 | **Captured** |
| CountryPack; BF first; CI shadow only | FR-45–47; §5 | **Captured** |
| SMS as OS; verbs PAY/PAUSE/SOLDE/AIDE/STOP | FR-29–31 | **Captured** |
| Settlement / payout clock | FR-35–37, FR-39; NFR-5 | **Captured** |
| Informal onboarding + soft KYC | FR-1, FR-43–44; UJ-5 | **Captured** (threshold locked in PRD) |
| Trust: named SMS, shop match | FR-13, FR-33; UJ-2 | **Captured** |
| Failure default; retry inbox | FR-25–26 | **Captured** |
| Open questions §11 | PRD §8 + memlog locks | **Mostly closed** (see GAP-5) |

## 4. Gaps and refinements

### GAP-1 — NEXT/LATER discipline and override (check outcome)

**Expectation:** Every NEXT/LATER item stays out of MVP FRs except explicit Out of Scope; no silent NEXT→FR; portal+API override allowed.

**Finding:** Traceability table (§2.1–§2.2) passes. FR-3 is the only brief-addendum NEXT item promoted, with PO lock recorded.

**Impact:** None for scope gate.

**Recommendation:** Keep FR-3; optionally add one line in PRD addendum §4 footnote that brief-addendum NEXT row “one account” was superseded by PO lock (audit trail only).

---

### GAP-2 — Competitive / PI-SPI landscape duplicated in main PRD

**Source:** Brief addendum §5 (light scan, PI-SPI dates, aggregator names, Orange WebPay omission).

**PRD:** `prd.md` §10 Why Now names CinetPay, PayDunya, LigdiCash, Flutterwave, PI-SPI launch timeline; §12 cites BF aggregator fee band. Same material is consolidated in PRD addendum §7.

**Impact:** Low for product truth (consistent story). Medium for document contract: “technical-how / landscape stays in addendum” — polish may re-trim §10 to a short pointer.

**Recommendation:** Shorten `prd.md` §10 to strategic “why now” without vendor/communiqué detail; keep citations in addendum §7 only.

---

### GAP-3 — Low-amount identity proof posture (qualitative)

**Source:** Brief addendum §7 — “Identity proof for small amounts: this phone paid this merchant before,” not government KYC on day one. Related NEXT: known-customer whitelist.

**PRD:** MSISDN identity (FR-46), shared-phone disclosure (FR-12, NFR-7), merchant-named trust (FR-13). No FR/NFR for “returning payer” trust heuristics or repeat-relationship proof.

**Impact:** Low for MVP mechanics; medium for fraud/trust epics and Customer copy when amount or merchant risk rises.

**Recommendation:** Add a one-line NFR or addendum §2 note that MVP relies on MSISDN + merchant display-name match + consent audit; whitelist remains NEXT.

---

### GAP-4 — GTM / positioning color from spine (on-time collection; guilds as motion)

**Source:** Brief addendum §1 — “Market **on-time collection**”; “Guilds over ads”; guild launch NEXT; §9 guilds not MVP surface.

**PRD:** Auto-debit non-goals and shame-free reminders (NFR-11, §21) cover **never market auto-debit**. “On-time collection” as positive positioning and guild-led GTM are not stated in vision/§21 — only implied via offline wedge and addendum rejected-alternatives.

**Impact:** Low for FR set; medium for marketing, launch playbook, and epic acceptance (“do not optimize guild product surface”).

**Recommendation:** One sentence in PRD §1 or §21: lead with on-time, payer-confirmed collection; guilds = distribution motion (not FR scope).

---

### GAP-5 — Brief economics “% TBD” vs locked 2.5% take-rate

**Source:** Brief addendum §6 — merchant fee “modest % TBD”; open question §11.1.

**PRD:** FR-40 / §12 lock **2.5%** after first successful MoMo collect; memlog decision.

**Impact:** Documentation drift only — readers of brief addendum alone may think rate is still open.

**Recommendation:** No PRD change required. Optional cross-note in PRD addendum §6 that PO locked 2.5% (configurable later policy).

---

### GAP-6 — Imam / mosque persona: dashboard vs named SMS emphasis

**Source:** Brief addendum §4 — “Named SMS that matches the mosque name matters **more than a dashboard**.”

**PRD:** Named SMS FR-13/FR-33; Merchant Portal IA is substantial for all personas.

**Impact:** Low — qualitative priority for vertical epics (mosque treasurer may need thinner portal MVP).

**Recommendation:** Keep in addendum §8 persona color; UX spec can weight SMS/receipt over dashboard depth for cash-heavy treasurers.

---

### GAP-7 — Tax / e-invoicing “empty hooks only” (LATER)

**Source:** Brief addendum LATER — tax/e-invoicing empty hooks only.

**PRD:** Listed in addendum §5 LATER; not named in `prd.md` §5 bullet list (§6.2 defers full NEXT/LATER to addendum).

**Impact:** None if architecture reads addendum; low if teams only read §5.

**Recommendation:** Optional single Non-Goal bullet in `prd.md` §5: “tax/e-invoicing integration hooks only, no compliance product in MVP.”

---

## 5. Qualitative / tone items

| Brief addendum tone | PRD expression |
| --- | --- |
| Shame-free private reminders (Mariam) | NFR-11, UJ-2, §21 | **Captured** |
| Silence = success | FR-32 | **Captured** |
| Literacy-first; web equal not superior | FR-49, NFR-8 | **Captured** |
| Cash as on-ramp, not apology | §21, SM-C4 | **Captured** |
| Stripe-like docs for developers | UJ-3, §21 | **Captured** |
| Fraud/matching shop / payout clock beats features | Partial — payout clock FR-35–39; shop name FR-13; fraud mesh mostly NEXT | **Partial** (see GAP-3) |

## 6. Reconciliation verdict

**Overall:** Brief addendum is **substantially absorbed** into `prd.md` + PRD `addendum.md`. NEXT/LATER items are **not** smuggled into MVP FRs except the **documented PO override** (one Merchant account: portal + API, FR-3). Technical-how and catalogs live primarily in PRD addendum; main PRD carries some landscape duplication (GAP-2).

**Actionable gaps for polish (priority):** GAP-2 (trim §10 landscape), GAP-3 (identity proof posture), GAP-4 (GTM wording), GAP-5 (economics cross-reference).

**Non-blockers:** GAP-6, GAP-7 — persona weighting and Non-Goal bullet optional.
