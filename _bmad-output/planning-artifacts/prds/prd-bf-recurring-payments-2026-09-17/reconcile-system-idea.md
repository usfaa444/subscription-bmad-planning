---
title: Input reconciliation — docs/system-idea.md
created: 2026-09-17
input: /workspace/projects/bf-recurring-payments/docs/system-idea.md
against:
  - prd.md
  - addendum.md
---

# Reconciliation: `docs/system-idea.md`

## 1. Source extract (authoritative phrases)

| Theme | system-idea.md |
| --- | --- |
| Problem | No simple way in Burkina Faso to offer subscriptions/recurring payments; businesses would build from scratch. |
| Example | Loyalty program or monthly service subscription. |
| Channel A | Businesses with websites/apps: API → hosted checkout (Stripe-like); customer logs in, enters info, configures recurring payment; callback with status on completion. |
| Channel B | Businesses without web app: register on web portal, set business details, configure billing terms (frequency, due dates, penalties, payment collection preferences), register clients by phone; client gets SMS to access portal and choose preferred payment method. |
| Billing | Platform sends invoices and billing notifications per agreed terms; plan automated debits from customer to merchant accounts later. |
| Rails (now) | Orange Money, Moov Money, Wave — gateway flexibility essential. |
| Rails (future) | PI-SPI (BCEAO interoperable); public API not out yet; architecture ready to adopt ASAP for A2A. |
| Vision | BF first; architect day one for multiple countries/currencies; West Africa → continent → global. |

## 2. Coverage summary

| system-idea element | PRD / addendum | Status |
| --- | --- | --- |
| BF recurring/subscription platform | §1 Vision, Collection Primitive, Subscription model | **Captured** |
| Dual go-to-market (API vs portal) | One receivables OS “rendered twice”; UJ-1/UJ-3; FR-3; addendum §1 | **Captured** — not two products |
| Hosted checkout + merchant callback | Hosted Checkout Session, Hosted Checkout Page, Webhook FR-14–16 | **Captured** (callback = signed Webhook) |
| Portal: billing terms | Plan FR-6: frequency, due dates, late-fee (penalties), collection preferences | **Captured** |
| Portal: clients by phone | FR-7 Client Record by MSISDN | **Captured** |
| SMS + pay path | Invite, SMS Invoice, verbs, Magic Link/OTP FR-18–19, FR-29–30 | **Captured** |
| Invoices & billing notifications | SMS Invoice, reminders (FR-6, FR-29); transactional set | **Captured** (SMS-first; no parallel email product) |
| Automated debits later | Direct Debit Port FR-48; future-debit-capable consent FR-11; addendum §1 “vacant, not live” | **Captured with intentional MVP posture**: reminder-then-pay only live; not merchant-initiated MoMo pull |
| Orange Money (live) | FR-21 live collect; MVP scope §6.1 | **Captured** |
| Moov Money, Wave | FR-22 designed adapters; §5/§6.2 non-goals for live collect; addendum §4 NEXT | **Not dropped** — deferred from live MVP |
| PI-SPI | Glossary, FR-48, NFR-9; addendum landscape §7 | **Not dropped** — socket + consent; live debit out of scope |
| Multi-country / multi-currency | CountryPack FR-45–47; vision §1; addendum §1 | **Captured** (BF only production pack in MVP) |

### Channel & rail drop check (explicit)

| Item | Dropped? | Notes |
| --- | --- | --- |
| API / hosted checkout channel | **No** | FR-14–17, UJ-3 |
| Merchant Portal / offline merchant channel | **No** | Merchant Portal, UJ-1 |
| Orange Money | **No** | Live MVP Rail |
| Moov Money | **No** (live) | Adapter + CountryPack Rail enum; live collect NEXT per §6.2, addendum §4 |
| Wave | **No** (live) | Same as Moov |
| PI-SPI | **No** (future) | Direct Debit Port; live debit LATER |

## 3. Gaps and refinements

### GAP-1 — Loyalty-program example not carried as a first-class narrative

**Source:** “loyalty program or a service requiring clients to subscribe to a monthly payment.”

**PRD:** Examples emphasize offline services (cleaning UJ-1), developer SaaS (UJ-3), and addendum persona color (school, gym, mosque). No explicit loyalty-program journey, Plan template, or FR traceability.

**Impact:** Low for MVP mechanics (same Plan + Subscription primitives apply). Medium for positioning and epic acceptance stories if founders sell loyalty-first merchants.

**Recommendation:** Add one sentence in PRD vision or addendum persona color, or accept as implicit in “any business” without a dedicated UJ.

---

### GAP-2 — “Customer logs in, enters info, configures recurring payment” vs merchant-defined Plan + consent

**Source:** Hosted path implies the **customer** configures the recurring payment during checkout (Stripe-subscription mental model).

**PRD:** Merchant (portal or API) defines Plan (amount, frequency, penalties, preferences). Customer runs identify → consent → method → confirm; Standing Authorization records consent to **merchant’s** schedule (FR-10, FR-11, FR-6). Assumption index maps “logs in” to Magic Link/OTP, not passwords.

**Impact:** Medium for API merchants expecting Session payload to let payers set amount/cadence at checkout. PRD allows Session + Plan/Obligation line items but does not require payer-side plan configuration UI.

**Recommendation:** Clarify in API capability docs (architecture/addendum): which Plan fields are merchant-fixed vs Session-overridable in MVP.

---

### GAP-3 — “Access a portal” for offline-channel customers vs no persistent Customer home

**Source:** Channel B: SMS instructions to **access a portal** and choose payment method.

**PRD:** Customer has no password account home in MVP (§11, §6.2 NEXT: consumer home). Flow is Magic Link / Hosted Checkout Page / SMS verbs — ephemeral surfaces, not a durable “my subscriptions” portal.

**Impact:** Medium UX expectation gap for customers who interpret “portal” as a standing login destination.

**Recommendation:** Align merchant-facing copy with “lien sécurisé” / one-time hosted page; track consumer subscription home as NEXT (already listed in addendum §4).

---

### GAP-4 — “Automated debits” language vs honest reminder-then-pay

**Source:** “Plan to support automated debits from customer accounts to merchant accounts later.”

**PRD:** Live model is **reminder-then-pay** only (FR-20); auto-debit marketing and merchant-initiated wallet pull are explicit non-goals (§5). Future path is Direct Debit Port / PI-SPI with consent that must remain auditably compatible (FR-11, FR-48).

**Impact:** Low if “automated debits” means regulated A2A/mandate debit post-PI-SPI. High if stakeholders read system-idea as silent recurring MoMo pull — PRD deliberately contradicts that for BF Rails today.

**Recommendation:** Treat system-idea “automated debits” as superseded by PRD wording: **payer-confirmed or scheme-mandate debit via Direct Debit Port**, not Orange silent pull.

---

### GAP-5 — “Business details” on portal vs minimal informal Merchant profile

**Source:** Offline merchants “set business details” (open-ended).

**PRD:** MVP signup is display name, MSISDN, neighbourhood (FR-1, FR-44); NINEA/RCCM/national ID only at soft KYC gate (FR-43). No broad business profile (logo, address, sector, hours) as FRs.

**Impact:** Low for core collection loop; medium for trust chrome (FR-13 uses display name + optional neighbourhood in SMS/checkout only).

**Recommendation:** If system-idea implied richer merchant profile, either scope to NEXT or add lightweight optional fields under Paramètres without blocking FR-2.

---

### GAP-6 — Billing notifications channel breadth

**Source:** “Invoices and billing notifications” (channel-agnostic).

**PRD:** Notification OS is **SMS** for MVP (FR-29); WhatsApp/IVR escalating dunning explicitly NEXT (§6.2, addendum §4). Email not required for Customers (NFR-7).

**Impact:** Low where SMS is sufficient; medium for merchants expecting email invoices or push from their own app (API Webhook covers developer channel merchants, not end-customer email).

**Recommendation:** Document that “billing notifications” in MVP = SMS Invoice + failure/receipt + reminder schedule; other channels are NEXT.

---

### GAP-7 — “Choose preferred payment method” at enrollment vs at pay time only

**Source:** Client chooses preferred payment method when entering via portal (could read as **sticky** preference at signup).

**PRD:** Customer chooses Rail at **pay time** (FR-19, FR-8); Plan does not lock Orange; no FR for storing “preferred Rail” across cycles in MVP.

**Impact:** Low; optional enhancement for repeat payers. Aligns with Moov/Wave not live yet (only Orange + cash path operationally).

**Recommendation:** If product wants saved method preference, add as NEXT when second Rail goes live.

---

## 4. Qualitative / tone items

| system-idea tone | PRD expression |
| --- | --- |
| “Simple way” vs build from scratch | Expanded into receivables-and-consent OS vs aggregators (§1, §10) — **stronger**, not dropped |
| Stripe-like | Hosted Checkout Session + Webhook — **captured** (MoMo-shaped, not card) |
| Gateway switch/integrate easily | Rail Adapter + CountryPack — **captured** |

No silent drop of dual-channel or multi-rail **vision**; MVP **scope** narrows live Rails to Orange and collection posture to reminder-then-pay.

## 5. Reconciliation verdict

**Overall:** `docs/system-idea.md` is **substantially absorbed** in `prd.md` and quoted in `addendum.md` §1. No merchant channel and no listed Rail (Orange, Moov, Wave, PI-SPI) was **removed** from the product story; Moov and Wave are **deferred from live MVP** with adapter hooks retained.

**Actionable gaps for polish (priority):** GAP-2 (who configures Plan at checkout), GAP-3 (customer “portal” wording), GAP-4 (automated debit semantics vs PI-SPI port), GAP-1 (loyalty example optional).

**Non-blockers:** GAP-5, GAP-6, GAP-7 — document or NEXT unless founder insists otherwise.
