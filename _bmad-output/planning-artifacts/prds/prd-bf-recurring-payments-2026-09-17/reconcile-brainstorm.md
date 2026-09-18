---
title: Input reconciliation — brainstorm-intent → PRD + addendum
input: brainstorm-intent.md
targets: prd.md, addendum.md
created: 2026-09-17
---

# Reconcile: `brainstorm-intent.md`

## Method

Compared the 2026-09-17 brainstorm handoff seed (`brainstorm-intent.md`) against `prd.md` and `addendum.md`. Marked **retained** where binding stance, MVP MUST, critical insights, or tone sections explicitly carry the idea. Marked **gap** where the FR/NFR spine preserves mechanics but drops qualitative product feel, voice hierarchy, or strategic nuance the brainstorm treated as binding.

## Well retained (no action required)

| Brainstorm element | PRD / addendum anchor |
| --- | --- |
| Receivables-and-consent OS; dual-channel one product | Vision §1; FR-3, FR-10; addendum §1–2 |
| Reminder-then-pay; never auto-debit marketing | FR-11, FR-20; Non-Goals; NFR-11 |
| Plan vs collection / Rail at pay time | FR-6, FR-8, FR-19 |
| Orange live + Moov/Wave designed; CountryPack; vacant Direct Debit Port | FR-21–22, FR-45–48 |
| Cash Mark-paid first-class; obligation survives failure | FR-23, FR-25; SM-C4; §21 tone |
| MSISDN, French-first, XOF as pack attribute | FR-46–47, FR-15, NFR-1 |
| Silence on success; shame-free reminders | FR-32, NFR-11; §21 |
| Take-rate after first MoMo success | FR-40–42 |
| Sub-15-minute sitting; client under 60s | FR-2, FR-7; SM-1, SM-2 |
| SMS verbs; STOP mapping; invoice fields | FR-29–31, FR-30 |
| Payout Clock / settlement finality | FR-35–37, FR-39; vision + FR-34 |
| Sandbox + Stripe-shaped DX (capability) | FR-5, FR-14, FR-50; UJ-3 |
| Guilds / PI-SPI moat as GTM or socket (not MVP FR) | addendum §4–5, §7; PRD §5–6 Out of Scope — intentional deferral |

---

## Gaps (qualitative / tone / feel the FR structure thins or drops)

### Gap 1 — Literacy-first channel hierarchy (SMS/USSD primary; web as fallback)

**Brainstorm:** Theme 14 and critical insights treat literacy constraints as forcing **one consent protocol** where feature-phone `*shortcode#`, SMS numeric verbs, and hosted URL are **equal renderers** — with explicit stance that **web is fallback, not the default**, and that **one SMS must carry invoice + link + USSD** so Fatou never needs a “second product” to pay.

**PRD:** NFR-8 states verbs are not degraded and hosted web is “equal, not superior.” MVP surfaces (§11) ship SMS + Hosted Checkout Page; USSD is NEXT (FR-49). FR-29 specifies invoice content but **no consequence** that a single outbound message must expose pay paths (verb, magic link, and future USSD stub) together. UJ-2 climax is receipt + shop match on **web or PAY**, not verb-first completion.

**Feel lost:** The product reads as **Stripe-like hosted checkout with SMS notifications**, not **SMS-as-OS with web as optional renderer**. UX/epics may default to “send link, hope for browser” unless EXPERIENCE.md restores the brainstorm hierarchy.

**Severity:** Medium (MVP can ship; conversion and trust on feature phones depend on restoring this in UX copy and SMS templates, not new FRs.)

**Suggested carry-forward (non-FR):** UX spine — composite SMS layout rule; success metrics on PAY-verb vs link-open mix; avoid portal/checkout IA that implies web superiority.

---

### Gap 2 — “Living receivables book” as hero job vs subscription/checkout mechanics

**Brainstorm:** Theme 4 — **Merchant Receivables OS**: a **living who-owes-what book**, explicitly **not** generic subscription SaaS. Merchant bedrock job is **stop chasing** without a collector; the loop is **shame-free book + reminder-then-pay**, same as customer “stay in good standing.”

**PRD:** Vision and IA name Accueil as receivables book (§22). FRs are organized around Plan, Subscription, Hosted Checkout Session, and Collection Attempt — correct primitives, but **testable consequences center ledger states and APIs**, not the merchant’s felt shift from WhatsApp chase to **organized, private dignity**. Emotional JTBD (§2.1) mentions shame-free reminders; **no FR or SM** validates book clarity, lateness visibility, or “I look professional” social job. Working title and prose still lead with **“subscriptions and recurring collection.”**

**Feel lost:** Builder implementing FR-6–FR-7 may ship a **billing admin UI** rather than a **collector replacement** that feels like upgrading from a notebook. The brainstorm’s anti-pattern (Flutterwave-style “recurring” language) is in addendum §3 but not wired to merchant-facing voice guardrails beyond auto-debit.

**Severity:** Medium–high for offline wedge (Awa).

**Suggested carry-forward:** §21 + merchant portal copy standards; SM or qualitative review criterion for “chase replacement” (e.g. lateness list without shame labels); prefer “receivables” / “encaissements” over “subscription SaaS” in merchant strings.

---

### Gap 3 — Trust artifacts as primary conversion (not feature checklist)

**Brainstorm:** Critical insight — **trust artifacts beat features**: named SMS, shop + **neighbourhood**, **regulated-looking receipt**, **guild** distribution, **visible payout clock** (customer paid vs merchant received) will **out-convert** extra features in a cash culture.

**PRD:** FR-13 (shop name; neighbourhood when space allows), FR-33, FR-39, §21 “official-enough receipt.” **Guilds** correctly deferred to GTM (addendum §4). Gap is **depth of feel**: no FR/NFR defines **receipt/regulated aesthetic** (layout, wording, what makes Fatou show spouse/parent), no **neighbourhood as trust signal** beyond optional SMS real estate, no success metric on **trust-match** (shop name recognition, receipt shareability). FR-13 consequences are **presence**, not **quality bar**.

**Feel lost:** Implementation can satisfy FR-13 with minimal strings while missing the brainstorm’s **“looks like the real economy, not a neobank”** bar that drives MoMo confirm rates.

**Severity:** Medium (differentiation vs aggregators).

**Suggested carry-forward:** DESIGN.md receipt mock as binding; UX review checklist cited in §21; optional SM on merchant NPS/trust proxy post-first-collect.

---

### Gap 4 — Cash-to-MoMo graduation loop (growth without shame)

**Brainstorm:** Theme 8 — cash is **on-ramp**; **graduating cash payers to MoMo is the growth loop**, not a v2 apology; **do not shame cash payers**. SM-C4 in PRD aligns on **not** treating cash share as failure.

**PRD:** FR-23 Cash Mark-paid; no take-rate; tone §21 “cash is on-ramp, not apology.” **No qualitative guidance** for merchant or customer moments after cash mark-paid (neutral dignity vs silent vs gentle MoMo invite next cycle). Brainstorm framed graduation as **product narrative**; FR structure treats cash as **ledger event only**.

**Feel lost:** Portal could accidentally signal “failed digital” on cash rows, or omit the brainstorm’s **hopeful path** from notebook + hand cash → standing authorization + MoMo when wallet has float — without building full “collector mode” (NEXT).

**Severity:** Low–medium for MVP; high for GTM story with informal merchants.

**Suggested carry-forward:** EXPERIENCE.md states for cash mark-paid UI/SMS tone; merchant help copy on cash as valid completion; defer nudges to NEXT unless a single non-spam line is agreed.

---

### Gap 5 — Identity & household as design north star (payer ≠ beneficiary)

**Brainstorm:** Theme 3 — **MSISDN primary key; shared phones; payer ≠ beneficiary**. NEXT items include family/group pay and share-pay-link; critical insights tie **demand-side consumer home** to later household complexity.

**PRD:** Shared-phone **edge case** (UJ-1 Staff PIN; FR-12 disclosure; UJ-2 spouse shows SMS). **Payer ≠ beneficiary** is Out of Scope (§5–6) with no preserved **MVP feel rule**: e.g. invite/consent copy acknowledging “this phone may pay for someone else,” or merchant Client Record fields for beneficiary name vs payer MSISDN. Household theme collapsed to **security** (PIN, OTP) not **social payment reality** in BF.

**Feel lost:** Fatou paying for a child’s school fee or a relative’s gym fee — brainstorm names this trust/UX space; PRD MVP flows assume **one MSISDN = one obligated customer** emotionally even when mechanics work.

**Severity:** Low for MVP scope; medium for SMS/consent **voice** authenticity.

**Suggested carry-forward:** Consent/SMS copy hooks for “who is served” without building group pay; Issa/Mariam persona color stays in addendum until vertical templates NEXT.

---

### Gap 6 — Notifications mesh and “SMS is the OS” operational richness

**Brainstorm:** Theme 6 — SMS as OS; **every action has a verb**; mesh includes WhatsApp/IVR/flash as **failover**, not single channel. MVP MUST includes full verb set.

**PRD:** FR-30–31 implement core verbs and STOP; WhatsApp/IVR explicitly Out of Scope. **Gap is qualitative:** brainstorm’s **mesh** metaphor (failover, channel redundancy) is reduced to **four transactional SMS types**; no product stance on **delivery failure empathy** (beyond failure collect FR-25) or **channel dignity** when SMS is delayed.

**Feel lost:** “OS” implies Fatou can **live in SMS** for lifecycle; PRD guarantees verbs but not **operational completeness** feel (e.g. SOLDE after partial pay, timezone-friendly quiet hours — not in brainstorm MVP MUST either).

**Severity:** Low (mostly NEXT mesh).

**Suggested carry-forward:** Keep in addendum; note in ops runbooks; revisit with WhatsApp NEXT.

---

### Gap 7 — Strategic “mandate stockpile” narrative (PI-SPI moat as felt urgency)

**Brainstorm:** **No PI-SPI is the wedge, not a hole**; consented mandates **stockpiled now** become moat when switch flips; market **on-time collection**, never auto-debit.

**PRD:** FR-11, FR-48, §10 Why Now cover sockets and honesty. **Founder/strategic tone** — winner is whoever has mandates when API opens — lives in brainstorm critical insights but **PRD §10 is landscape-analytic**, not a **team motivator** for consent audit quality and not marketing copy.

**Feel lost:** Internal build priority for **consent text versioning** (NFR-12) is compliance-shaped, not **moat-shaped**.

**Severity:** Low for FR completeness; medium for roadmap storytelling.

**Suggested carry-forward:** addendum or memlog for GTM/internal; no new FR.

---

### Gap 8 — Vertical / mosque / offline provider parity in emotional weight

**Brainstorm:** **Who we serve** lists mosque, gym, school, cleaning equally; vertical templates in NEXT; Imam treasurer color in addendum §8.

**PRD:** Lead UJs are Awa (cleaning), Fatou, Adama; mosque/school **persona color only** in addendum. FR-6 frequencies assume weekly/monthly; term/school deferred ASSUMPTION.

**Feel lost:** Product **feels cleaning-first** in journeys even though vision lists mosques/schools — brainstorm wanted **offline providers as co-equal** in positioning, not only as template backlog.

**Severity:** Low (MVP wedge choice).

**Suggested carry-forward:** Marketing/UX examples rotate verticals; FR-6 N-day interval as bridge.

---

## Cross-gap pattern

The PRD **correctly encodes MVP mechanics and compliance guardrails** from the brainstorm MUST list. What thins most in FR form is **channel hierarchy (literacy-first)**, **merchant dignity / receivables-book hero**, **trust-artifact quality bar**, and **cash graduation narrative** — all **tone, defaults, and IA emphasis** rather than missing capabilities. Restore these in **§21 enforcement**, UX spines, SMS/DESIGN artifacts, and merchant copy review — not by inflating FR count.

## Reconciliation verdict

| Verdict | Detail |
| --- | --- |
| **Safe to build MVP from PRD** | Yes — no brainstorm MUST item missing as FR. |
| **Polish before UX/epics** | Yes — address Gaps 1–5 qualitatively so implementation does not drift toward generic MoMo checkout SaaS. |
| **Intentional drops** | Guild product surface, notification mesh, group pay, vertical templates, consumer home — aligned with brainstorm LATER/NEXT and PRD §6.2. |

## Recommended actions (pre-polish PRD)

1. Add a short **“Merchant and customer voice”** bullet block under §21 referencing literacy-first composite SMS (Gap 1) and receivables-book hero (Gap 2) — prose only, no new FR IDs unless PO wants NFR-13.
2. Tie FR-13 / SMS Receipt to a **trust-artifact review gate** in UX (Gap 3).
3. Document **cash mark-paid tone** and optional next-cycle MoMo hope in addendum §8 persona notes or EXPERIENCE handoff (Gap 4).
4. One consent/SMS sentence on **shared phone / paying for someone else** (Gap 5) without scope creep to group pay.

---

*End of reconcile — brainstorm-intent.md*
