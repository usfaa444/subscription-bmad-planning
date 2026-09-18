# Input reconciliation — brief → PRD (+ addendum)

**Date:** 2026-09-17  
**Inputs:** `brief.md`, `prd.md`, `addendum.md`  
**PO lock log:** `prd-bf-recurring-payments-2026-09-17/.memlog.md`  
**Rule:** Gaps only below; no PRD rewrite.

---

## 1. Executive extract (brief stance preserved in PRD?)

| Brief theme | Brief anchor | PRD / addendum landing | Status |
| --- | --- | --- | --- |
| One product, two skins | Exec summary, Solution | §1 Vision, FR-3, UJ-1/3, §11 surfaces | OK |
| Receivables-and-consent OS | Problem, Solution, Different | §1, Glossary, FR-6–9, FR-34 | OK |
| Reminder-then-pay, never merchant pull | Exec, Scope Collect | FR-11, FR-20, §5 Non-Goals, §15 | OK |
| Orange live; Moov/Wave designed not live | Scope Collect, Out | FR-21, FR-22, §6.2 Out | OK |
| Cash mark-paid first-class | Scope Collect, Different | FR-23, FR-42, SM-C4, §21 tone | OK |
| MSISDN, French-first, XOF | Success table, Scope Locale | FR-46, FR-47, FR-15, NFR-1 | See gap G-3 |
| CountryPack, no second-country prod | Scope Locale, Out | FR-45, NFR-9, §6.2 | OK |
| Sandbox without live Orange | Success DX, Scope Locale | FR-5, FR-50, UJ-3 | OK |
| Future-debit consent; vacant Direct Debit | Scope Locale, Different | FR-11, FR-48, addendum §2 | OK |
| Never market auto-debit | Different, Out | FR-11 consequences, §5, §15 | OK |
| Take-rate after first success | Scope Motion, Success Economics | FR-40–42, §12 | See PO-1 (logged) |
| Sub-15 min / sub-60 s client | Success, Scope Motion | FR-2, FR-7, SM-1, SM-2 | OK |
| Send-a-bill = same primitive | Scope Motion | FR-9, Glossary Subscription | OK |
| Trust: named SMS, payout clock, no double-charge | Success Trust, Different | FR-13, FR-33, FR-24, FR-35–39, SM-4 | See gap G-2 |
| Settlement finality as product | Different (headline) | FR-4.6 description, FR-34–37, NFR-5, §20 | OK (headline in prose; behavior in FRs) |
| Shame-free vs public chasing | Problem, persona tone in brief addendum | NFR-11, §2.1 Emotional, §21, UJ-2/4 | OK |
| Silence success; ping failure/change | Solution | FR-32, FR-29 | See gap G-1 |
| Staff PIN shared devices | Scope Checkout | FR-4 | OK |
| Failed retry inbox | Scope Collect | FR-26 | OK |
| Pause / Skip | Scope Collect | FR-27, FR-28 | OK |
| Idempotent collection tokens | Scope Collect | FR-24 (Collection Attempt token) | OK |
| Adapter hides Orange vs aggregator | Solution assumption | FR-21, memlog Orange path | OK (PO logged) |
| PI-SPI / second country / cards / native apps / PI license / customer balances | Scope Out | §5, §6.2, FR-38, addendum §5 | OK |

---

## 2. Brief Scope **In** → MVP FR or explicit Out

Each bullet from brief § Scope **In (MVP floor)** traced to PRD §6.1 + FR IDs (addendum §1 defers to PRD §6 as floor).

### 2.1 Checkout + portal (one machine)

| Brief IN item | MVP FR / scope | Explicit Out if N/A |
| --- | --- | --- |
| Hosted Checkout Session API + webhooks | FR-14, FR-16, §6.1, §18 | — |
| French MoMo hosted page (no cards) | FR-15, FR-10 | — |
| Merchant portal: plans, clients by phone, SMS invite | FR-6, FR-7, FR-18, §6.1, §22 IA | — |
| Magic-link / OTP identity | FR-12 | — |
| Staff PIN on shared devices | FR-4 | — |

**Verdict:** Fully covered.

### 2.2 Collect

| Brief IN item | MVP FR / scope | Explicit Out if N/A |
| --- | --- | --- |
| Reminder-then-pay only live path | FR-20 | — |
| Orange sole live rail; adapter designed for Moov/Wave | FR-21, FR-22 | Live Moov/Wave → §6.2 Out |
| Cash mark-paid | FR-23 | — |
| Idempotent collection tokens | FR-24 | — |
| Failed-collection retry inbox | FR-26 | — |
| Basic pause / skip | FR-27, FR-28 | — |

**Verdict:** Fully covered.

### 2.3 Ledger + proof

| Brief IN item | MVP FR / scope | Explicit Out if N/A |
| --- | --- | --- |
| Plan separated from collection method | FR-8, FR-6 | — |
| SMS invoice (what / whom / often / how much / by when) | FR-29 consequences, FR-33 | — |
| SMS receipt | FR-29, FR-33 | — |

**Verdict:** Fully covered.

### 2.4 Locale + sockets

| Brief IN item | MVP FR / scope | Explicit Out if N/A |
| --- | --- | --- |
| CountryPack skeleton (BF + XOF + MSISDN; no `if-Burkina`) | FR-45, FR-46, FR-47, NFR-9 | — |
| Sandbox | FR-5, FR-50 | — |
| Future-debit-capable consent copy | FR-11 | — |
| Zero English / `$` / card-logo defaults | FR-15, NFR-1, SM-8 | See gap G-3 |

**Verdict:** Covered with one locale-default nuance (G-3).

### 2.5 Motion

| Brief IN item | MVP FR / scope | Explicit Out if N/A |
| --- | --- | --- |
| Sub-15-minute onboarding | FR-2, SM-1 | — |
| Take-rate after first success | FR-40, §12 | PO narrows “success” → MoMo (PO-1) |
| First bill = subscription cycle primitive (send-a-bill wedge) | FR-9, Glossary | — |

**Verdict:** Covered; fee trigger narrowed and logged (PO-1, not a reconciliation defect).

---

## 3. Brief Scope **Out** → PRD explicit Out

| Brief OUT | PRD |
| --- | --- |
| Live PI-SPI debit | §5, §6.2, FR-48 vacant only |
| Auto-debit marketing | §5, FR-11 |
| Second-country production pack | §5, §6.2, FR-45 |
| Live Moov / Wave | §5, §6.2, FR-22 NON-GOAL |
| Native merchant apps | §5, §11 |
| Cards | §5, FR-15 |
| Holding customer balances if pass-through viable | §5, FR-38, §8.1 open |
| Payment-institution license | §5, §15 |

**Verdict:** All brief OUT items appear in PRD §5 / §6.2 or bound FRs.

---

## 4. Binding phrases (not diluted?)

| Phrase (brief or brainstorm floor) | PRD expression | Notes |
| --- | --- | --- |
| Customer-authorized pay-on-reminder; never merchant-initiated wallet pull | FR-20, FR-11, Glossary Reminder-then-pay | Strong |
| One consent protocol / state machine: identify → consent → method → confirm | FR-10, Glossary | Strong |
| Never market auto-debit | FR-11 consequences (incl. French “prélèvement automatique”) | Strong |
| Next due date shown before it happens | FR-11 consequences | Strong |
| Silence means success | FR-32 | Strong for success path |
| Ping on failure or change | FR-32 → FR-29 | **Weak for “change”** → G-1 |
| MSISDN as customer key | FR-46 | Strong |
| Wallet secrets never on merchant servers | FR-17 | Strong |
| Obligation survives rail failure | FR-25, SM-3 context | Strong |
| No double-charge on retry | FR-24, SM-4 | Strong |
| Cash is on-ramp / equal method | FR-23, SM-C4, §21 | Strong |
| Feature phones first-class in model (SMS/USSD-capable) | FR-49, NFR-8 | Strong (USSD live Out) |
| On-time collection story, not autopilot wallets | §1, §10, §21 | Strong |

---

## 5. Qualitative tone (French, shame-free, named SMS, settlement finality)

| Tone element | Brief | PRD |
| --- | --- | --- |
| French-first; no `$ / card chrome on first-run | Success Identity; Scope Locale; Customers § | FR-15, NFR-1, SM-8, §21 | G-3 for dev-English carve-out |
| Shame-free private reminders vs public chasing | Problem; brief addendum defaults | NFR-11, §2.1 Emotional, §21, Mariam persona in addendum §8 | OK |
| Named SMS matching shop | Different; Success Trust | FR-13, FR-33, UJ-2 | OK |
| Visible customer-paid / merchant-received clock | Different; Success Trust | FR-35–37, FR-39, Payout Clock glossary, UJ-4 | OK |
| Settlement finality is the product | Different headline | FR-4.6 lead + ledger FRs; not a standalone NFR title | Acceptable (behavior locked) |
| Official-looking receipt | Different trust artifacts | §21 “official-enough receipt”; FR-29 generic SMS Receipt | **G-2** |

---

## 6. Product-owner locks overriding brief opens (acceptable if logged)

| Override | Brief open | PRD lock | Logged |
| --- | --- | --- | --- |
| PO-1 Take-rate % and trigger | “Modest % TBD”; “first successful collection” | 2.5% after first successful **MoMo** collect ever; 0% fail/cash | `.memlog.md` L9, L25; PRD §9 assumption |
| PO-2 Orange commercial path | Brief assumption: aggregator or local API | Outcome FR-21; counterparty architecture-owned | `.memlog.md` L10 |
| PO-3 KYC gate math | Informal start (brief); details open | 500k XOF/month cumulative or payout change | `.memlog.md` L13; FR-43 |
| PO-4 SMS STOP mapping | Brief addendum had open bundles | FR-31 locked mapping; SMS bundled in MoMo take-rate | `.memlog.md` L14 |
| PO-5 USSD live | Model in brief | SMS + web MVP; shortcode NEXT | `.memlog.md` L15; FR-49 |
| PO-6 Payout clock T+1 | Brief trust bar (no numeric SLA) | T+1 target + pending settlement | `.memlog.md` L11–12, L16 |
| PO-7 One portal+API account | Implied in brief | FR-3, FR-5 immediate keys/sandbox | `.memlog.md` L17 |
| PO-8 Staff PIN / Pause-Skip stay MVP | Brief addendum had NEXT ambiguity on some items | FR-4, FR-27, FR-28 | `.memlog.md` L22 |

**Verdict:** All material overrides referenced in PRD §0, §8–9, FR text, or addendum §6; audit event L27 claims capture complete.

---

## 7. Addendum role (reconciliation)

- Addendum §1 states MVP floor = PRD §6 + brief Scope — consistent with brief Scope IN trace above.
- NEXT/LATER catalogs (addendum §4–5) not promoted into MVP FRs — matches brief pointer to addendum for matrices and PRD §6.2.
- Landscape / technical-how / persona color appropriately out of FR spine — no brief Scope IN item left only in addendum.

---

## 8. Gaps (action list — do not fix in this pass)

### G-1 — “Ping on change” not specified in transactional SMS set

- **Brief:** “Silence means success; ping on failure or **change**.” (Solution §)
- **PRD:** FR-32 says change produces SMS and points to FR-29, but FR-29 only enumerates invite, invoice, receipt, and **collection failure** — not Plan amount/frequency/due-date changes, Merchant edits, or Skip/Pause confirmations beyond verb flows.
- **Risk:** UX/spec may omit Customer/Merchant notifications when terms change mid-subscription while still compliant with “silence on success.”

### G-2 — “Receipt that looks official” lacks testable FR bar

- **Brief:** Trust artifacts include “a receipt that looks **official**” alongside named SMS and payout clock (What Makes This Different).
- **PRD:** SMS Receipt is required (FR-29) and Merchant name in body (FR-33); §21 says “official-enough receipt” aesthetically — no acceptance criteria for receipt structure (reference id, amount, date, Merchant identity, paid status) distinct from generic transactional SMS.
- **Risk:** Implementation could satisfy FR-29 with minimal text and miss brief trust positioning.

### G-3 — “Zero English … defaults” slightly narrowed vs brief Locale IN

- **Brief Scope IN:** “zero English / `$` / card-logo **defaults**” (no carve-out).
- **PRD NFR-1:** French for first-run user surfaces; **English may exist in developer technical identifiers** and API error messages “intended for end users” are French — but developer-facing defaults are explicitly allowed non-French.
- **Risk:** Low for Customer paths (SM-8 covers QA bar); gap only if brief locale IN was meant **platform-wide** zero-English defaults including Developers surfaces.

### G-4 — Brief Success “first successful collection” vs locked MoMo-only fee start (documentation trace only)

- **Not a PRD content gap** given PO logging, but **brief text unchanged**: Success table Economics still reads “first successful **collection**” while PRD FR-40 charges take-rate after first **MoMo** success (cash excluded per §9 assumption).
- **Recommendation for brief maintenance (out of scope here):** align brief Success row wording with PO-1 so future reconciliations do not re-flag.

---

## 9. Reconciliation verdict

- **Scope IN:** All brief Scope IN bullets map to MVP FRs or explicit §5/§6.2 Out (with PO-1 as logged narrowing, not missing coverage).
- **Scope OUT:** Aligned.
- **Binding posture:** Strong except **change notifications** (G-1).
- **Qualitative tone:** French, shame-free, named SMS, and settlement visibility are carried; **official receipt** gravitas is under-specified (G-2).
- **PO locks:** Logged in `.memlog.md` and reflected in PRD; no unlogged overrides found.

**Open gaps for downstream (UX/architecture/epics):** G-1, G-2, G-3 (minor); G-4 is brief-doc hygiene only.
