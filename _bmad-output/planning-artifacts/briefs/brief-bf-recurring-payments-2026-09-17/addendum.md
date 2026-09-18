---
title: Product Brief Addendum — bf-recurring-payments
status: complete
created: 2026-09-17
updated: 2026-09-17
---

# Addendum — depth parked from the brief

This file holds matrices, landscape notes, and rejected-alternative rationale that a PRD or architecture skill will need. It is not the brief. Audit/override history lives in `.memlog.md`, not here.

Sources extracted (not ingested wholesale): `docs/system-idea.md`, `README.md`, brainstorm intent + memlog + html (2026-09-17), light public landscape scan 2026-09-17. No API keys requested or recorded.

---

## 1. Spine and chosen directions (binding)

1. **Spine.** Dual-channel receivables + reminder-pay on one live rail, with a channel-agnostic consent state machine.
2. **Architecture bet to carry (not a design).** Rail adapters + vacant `DirectDebitPort` + CountryPack. Country → scheme → rail → adapter. Do not encode `if-Burkina`. PI-SPI is one future scheme.
3. **GTM wedge.** Offline SMS/portal merchants plus send-a-bill / reminder-pay, then richer subscription packaging. Guilds over ads. Developers as the online-merchant channel.
4. **Deferred moat.** Consented recurring mandates + PI-SPI-shaped sandbox. Market **on-time collection**. Never market auto-debit.

**Two-sided job (same loop).** Merchant: stop chasing weekly/monthly money without hiring a collector. Customer: stay in good standing without walking to a kiosk every cycle.

**What recurring is.** Standing customer authorization + scheduled obligation. Not a merchant-initiated wallet pull (those barely exist on BF MoMo; even Orange web-pay patterns typically require a customer USSD/OTP confirm).

**Collection primitive.** Invoice → SMS/USSD → customer confirms → wallet debit → merchant credit.

**Unified consent/checkout.** identify → consent → method → confirm. Renderers: hosted web, SMS verbs, later USSD, later IVR. Literacy forces one protocol.

**Plan vs collection.** Separate *what is owed* from *how it is collected* (cash, MoMo, future bank debit).

**A subscription is five primitives.** Promise, schedule, payment instruction, record, reminder. Everything else is packaging.

---

## 2. Theme map (vocabulary only — not a backlog)

| # | Theme | Use |
| --- | --- | --- |
| 1 | Collection Primitive | reminder-then-pay; standing auth; scheduled obligation |
| 2 | Dual-channel Access | API checkout + portal/SMS as two skins |
| 3 | Identity & Household | MSISDN primary key; shared phones; payer ≠ beneficiary |
| 4 | Merchant Receivables OS | living who-owes-what book |
| 5 | Rail Abstraction & PI-SPI | MNO adapters; vacant DirectDebitPort |
| 6 | Notifications Mesh | SMS as OS; WhatsApp/IVR/flash as failover |
| 7 | Trust / Fraud / Proof | named receipts; matching shop; payout clock; guilds |
| 8 | Cash Culture Hybrid | cash mark-paid first-class; graduate to MoMo |
| 9 | Developer Experience | session + webhooks + sandbox + local samples |
| 10 | Informal Onboarding | name + phone < 60s; merchant sitting < 15 min |
| 11 | Vertical Templates | weekly clean, school term, mosque, gym, rent |
| 12 | Platform Economics & Growth | take-rate after first success; guilds; developers |
| 13 | Multi-country Architecture | CountryPack; BF is first pack, not the schema |
| 14 | Literacy-first UX | USSD/SMS/numeric verbs; web is a fallback |

---

## 3. Opportunity matrix

### MVP MUST

The floor is the brief **Scope** section (grouped). Do not treat this addendum as a second source of truth.

**Renderer note.** The consent protocol is designed for USSD. First *shipped* customer paths are hosted web + SMS. A live `*shortcode#` is NEXT unless a cheap path appears; missing USSD is not a hole in the model.

### NEXT SHOULD

| Item | Why it waits |
| --- | --- |
| Moov + Wave + in-session failover (Orange timeout → offer Moov) | Adapter designed now; live integration after Orange path is trusted |
| WhatsApp receipts + IVR | SMS is the OS first; mesh comes after delivery is reliable |
| Escalating dunning + payday-aligned windows (~26th–5th) | Basic retry inbox is enough to learn failure modes |
| Collector offline cash + cash-day calendar | Cash mark-paid is the wedge; collector mode is scale |
| Family / group pay + share-pay-link | Household is real; payer≠beneficiary is a trust/UX risk |
| Consumer home listing all platform subscriptions | Demand-side only after people already pay through us |
| One-off invoices as a named product | Primitive exists in MVP; packaging/marketing is next |
| Vertical plan templates (maid, school term, mosque, gym) | Clone-a-plan may suffice at first |
| PWA + shop QR + paper membership card | Mobile web first |
| Payout to merchant’s own MoMo / Wave merchant code | Settlement design needed; may be pass-through from day one if viable |
| MRR / churn / failed-collection analytics | Founder can start from the receivables book |
| Laravel and Flutter samples; Ouaga / Abidjan docs | API + sandbox first; samples accelerate DX |
| Dispute freeze (“I did not get service”) | Social mediation after the happy path works |
| Mooré / Dioula keywords | French-first MVP; bilingual messages merchants do not compose |
| Guild / association launch | GTM motion, not a product surface |
| One merchant account spanning portal and API | Same account is the intent; shipping both keys can trail |
| Thermal / printed “PAID” receipt | SMS receipt first |
| MNO daily-limit display before confirm | Needs per-rail metadata |
| Known-customer whitelist | Fraud later |

### LATER / WON’T this version

Live PI-SPI direct debit · Second-country pack in production (CI = paper shadow) · Tontine / e-susu as a product · B2B supplier credit · NGO / government / institutional payers · Customer-invites-merchant demand-side growth · Split / franchise payouts · Credit scoring / bureau-style default risk · Payment-institution or e-money license · Recurring payouts / salary-like out · Savings-toward-due envelopes · Prepaid auto-refill merchant wallets · Tax / e-invoicing (empty hooks only) · White-label domains · Per-country aggregators as a product · Voice-mandate as a legal instrument · Women-support line as a standalone product · Super-App consumer wallet · Native apps for offline merchants · Card PAN checkout · Platform-held customer balances if pass-through is viable · Auto-debit marketing or copy that implies merchant-initiated pull

---

## 4. Personas (depth)

**Awa — weekly cleaning, Ouaga neighbourhood.** Informal, no tax ID yet, shared family Android. Writes names in a notebook. Success = Friday money without walking to every compound. Enters via cash mark-paid; some clients later pay Orange after the SMS. Needs staff PIN when a nephew helps.

**Issa — private school bursar.** Term-peaked fees, many MSISDNs, parents who pay from a different phone than the child’s. Needs proof-of-payment and a late list before the next term. Vertical template is NEXT; MVP is a cloned plan + SMS.

**Mariam — gym / salon.** Monthly standing, high no-show, wants pause/skip when the hall is closed. Shame-free private reminder, not a public “defaulter” wall.

**Imam / mosque treasurer.** Periodic due, cash-heavy, trust is the brand. Named SMS that matches the mosque name matters more than a dashboard.

**Developer at a small BF SaaS.** Needs `createSession` → URL → webhook, French XOF checkout, sandbox wallets, no card-shaped form. Will bring the online merchant if docs feel like Stripe, not like a bank PDF.

**Customer — Fatou.** Feature phone or mid-range Android on 3G. Wants one SMS that she can show her husband. Will confirm Orange if the shop name matches. Will not create an email account.

---

## 5. Competitive and rail landscape (light scan, 2026-09-17)

Not a TAM study. `bmad-deep-recon` was deliberately not run.

**Checkout aggregators in/near BF.** CinetPay (Orange BF, Moov BF, cards; published collect fees often 3.5–4.5% on BF MoMo), Flutterwave francophone MoMo (BF: `ORANGEMONEY`, `MOBICASH`; Orange often needs a customer USSD authorization code), PayDunya, Hub2 (WAEMU package), Sappay-class “one integration” pages. These validate that **one-shot Orange/Moov pay-in is already available** and that **recurring wallet pull is not**.

**Wallets.** Orange Money is the national leader (public commentary in the 40–55% range). Moov is the second rail and stronger in some rural corridors. Wave has operated in BF since 2021, claimed 1M+ users by mid-2025, and was still opening Ouaga agencies in 2026. Design all three adapters; go live on Orange only.

**Orange integration path.** Orange Developer “Web Payment / M Payment” public country list has historically omitted Burkina Faso. Local Orange Money merchant APIs and aggregators are the practical paths. [ASSUMPTION] MVP Orange adapter may sit on a local API or an aggregator; do not hard-lock the commercial counterparty in the product brief.

**PI-SPI / BCEAO (update vs system-idea.md).** The original idea said the public API was not out. As of this scan: PI-SPI launched 30 Sep 2025; BCEAO prolonged participant connection (banks/EMEs to 30 Sep 2026; supervised MFIs to 30 Jun 2027); a Business API sandbox is documented; a 16 Sep 2026 communiqué homologated participant Business APIs for enterprise payment services. **Implication:** sockets and consent language are more urgent, not less. **Non-implication:** we still do not ship live PI-SPI debit, do not become a payment institution, and do not market auto-debit. Access will almost certainly run through a licensed participant, not as a raw public “anyone can debit” API.

**Pricing comps (context only).** Aggregator take on BF Orange/Moov is commonly low-to-mid single digits plus SMS cost. Our stance: free until first successful collection, then a modest % TBD. SMS may be bundled into the take-rate or sold as merchant bundles — open.

---

## 6. Economics and legal (open, not invented)

| Topic | Stance | Open? |
| --- | --- | --- |
| Merchant fee | Free until first successful collection; then take-rate | Yes — % TBD |
| Failed attempts | Do not charge take-rate on failed collection | [ASSUMPTION] yes |
| Cash mark-paid | Ledger event; [ASSUMPTION] no take-rate on cash, or a smaller SMS-only fee | Yes |
| SMS cost | Fold into take-rate vs prepaid bundles | Yes |
| Settlement | Prefer pass-through to merchant MoMo; avoid holding customer balances | Yes — legal review |
| License | Not a payment institution / e-money issuer this version | No (out of scope) |
| KYC | Informal merchant can start; formalize path later; KYC-lite when amount/risk crosses a threshold | Yes — threshold TBD |
| Entity | Per-country legal entity when we expand; BF entity first | Yes |

---

## 7. Identity, locale, CountryPack

- Customer primary key: MSISDN + country code. Not email.
- Shared phones: staff PIN on merchant devices; household payer≠beneficiary is NEXT.
- Identity proof for small amounts: “this phone paid this merchant before,” not government KYC on day one.
- Locale: French-first UX and SMS. No English, `$`, or card-network defaults on first-run screens. Mooré / Dioula keywords are NEXT.
- Currency is a tenant/country attribute, never a hardcoded XOF constant, even while only BF is live.
- CountryPack isolates strings, rails, KYC rules, holidays, MSISDN parsing, wallet prefixes. All IDs country-prefixed. Côte d’Ivoire = paper shadow pack only.
- Compliance calendar (BF license vs UEMOA passporting) is a later ops artifact, not MVP scope.

---

## 8. Consent, rails, and failure

**Consent today, debit later.** Enroll first; collect on the agreed date. Copy must cover future A2A/PI-SPI debit without rewriting merchant APIs. Show next due date before it happens.

**Pick the rail at pay time.** Wallets do not interoperate cleanly. Customer chooses Orange vs (later) Moov vs Wave from whichever wallet has money *today*.

**Adapters.** Design Orange, Moov, Wave. Live: Orange. Vacant `DirectDebitPort`. Gateway-agnostic ledger events: intent, authorized, collected, settled, failed, refunded, written-off.

**Idempotency.** Retries, double-SMS, and webhook replay must not double-charge.

**Failure default.** The obligation remains; only the collection attempt failed. Failed-collection inbox: retry, later switch rail, or mark cash.

**Settlement finality.** Visible clock: customer paid vs merchant received. If the merchant cannot trust arrival, nothing above it matters.

**Notifications.** SMS is the OS. Designed verbs: PAY, PAUSE, SOLDE, AIDE, STOP (French-first keywords; local-language verbs later). One message carries invoice + link + (later) USSD. Silence = success.

---

## 9. Rejected or deferred alternatives (why)

| Alternative | Why not now |
| --- | --- |
| Two products (API company + portal company) | Duplicates the consent machine and event bus |
| Market as auto-debit / “set and forget” | Dishonest on current BF rails; destroys trust |
| Cards / English / `$` checkout | Wrong default; kills conversion |
| Build Côte d’Ivoire in v1 | CountryPack sockets yes; production pack no |
| Live PI-SPI debit | Public scheme is moving; we are not a licensed participant and the merchant API must not change later |
| Native merchant apps | Informal merchants will not install; mobile web (+ later PWA) |
| Platform-held customer wallets | License and trust risk; pass-through if viable |
| Demand-side “customer invites merchant” | Only after people already pay through us |
| Tontine / Super-App / credit bureau | Different products; steal focus from the five primitives |
| Guilds as a built product surface | Distribution motion, not an MVP feature |

---

## 10. Critical insights (would change a PRD if ignored)

Carry these; they already shape the brief. Restated so a PRD author does not have to re-derive them.

- Same loop, two skins — do not spec API and portal as separate systems.
- Reminder-pay is the honest BF primitive; no live PI-SPI is the wedge, not a hole.
- Literacy forces one consent protocol (`*shortcode#` equals `https` in the model).
- Cash is the on-ramp. CountryPack without building CI. Two wedges, one account.
- Trust artifacts and settlement finality beat extra features. Pick the rail at pay time.
- Defaults: silence = success; take-rate after first collect; never market auto-debit; shame-free private reminders.

---

## 11. Open questions for PRD (not blocking this brief)

1. Exact take-rate and cash-vs-MoMo fee treatment.
2. Orange commercial path: direct local API vs aggregator; settlement T+?.
3. Whether pass-through settlement is legally viable without holding funds.
4. Informal-merchant KYC threshold and when a tax ID is required.
5. SMS unit economics and STOP / consent retention rules (ARCEP / e-money participant constraints).
6. How soon a USSD shortcode is obtainable vs staying SMS + hosted web.
7. Payout clock SLA we are willing to publish.
8. Whether “one merchant account = portal + API” ships in the same increment as the two skins.

Do not start architecture from these; they are PRD discovery prompts.
