---
name: Sarati
description: Visual identity for Sarati — Burkina Faso receivables-and-consent OS — Merchant Portal, Hosted Checkout, and SMS — shadcn/ui brand-layer only.
status: final
updated: 2026-09-18
colors:
  # Brand-layer overrides. Unlisted shadcn tokens (background, foreground, muted,
  # muted-foreground, popover, card, border, input, ring, destructive) inherit.
  surface: '#F7F4EE'
  surface-raised: '#FFFFFF'
  ink: '#1C1915'
  ink-muted: '#534E45'
  border: '#C9C1B2'
  primary: '#3F5A3A'
  primary-foreground: '#FFFFFF'
  accent: '#A65D28'
  accent-foreground: '#FFFFFF'
  success-ink: '#1F4D32'
  late-ink: '#8A3B12'
  pending-ink: '#5C4A28'
  hold-ink: '#6B3A12'
  destructive-ink: '#8B1E1E'
  sms-bubble-in: '#FFFFFF'
  sms-bubble-out: '#E6EDE4'
typography:
  display:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 28px
    fontWeight: '650'
    lineHeight: '1.2'
    letterSpacing: '-0.015em'
  display-sm:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 22px
    fontWeight: '650'
    lineHeight: '1.25'
  body:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.5'
  body-strong:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 16px
    fontWeight: '600'
    lineHeight: '1.5'
  label:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 14px
    fontWeight: '600'
    lineHeight: '1.35'
  meta:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 13px
    fontWeight: '400'
    lineHeight: '1.4'
  sms:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.45'
  verb:
    fontFamily: 'ui-sans-serif, system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif'
    fontSize: 15px
    fontWeight: '700'
    lineHeight: '1'
    letterSpacing: '0.04em'
rounded:
  sm: 8px
  md: 12px
  lg: 16px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 24px
  '6': 32px
  tap: 44px
  gutter-mobile: 16px
  gutter-desktop: 24px
  max-portal: 720px
  max-checkout: 420px
components:
  button-primary:
    background: '{colors.primary}'
    foreground: '{colors.primary-foreground}'
    radius: '{rounded.md}'
    min-height: '{spacing.tap}'
  button-secondary:
    background: '{colors.surface-raised}'
    foreground: '{colors.ink}'
    border: '{colors.border}'
    radius: '{rounded.md}'
    min-height: '{spacing.tap}'
  status-pill:
    radius: '{rounded.full}'
    font: '{typography.label}'
  payout-clock:
    background: '{colors.surface-raised}'
    border: '{colors.border}'
    radius: '{rounded.lg}'
  trust-chrome:
    background: '{colors.surface-raised}'
    ink: '{colors.ink}'
  kyc-banner:
    background: '#F4E4D4'
    ink: '{colors.hold-ink}'
    radius: '{rounded.md}'
  sms-bubble-in:
    background: '{colors.sms-bubble-in}'
    foreground: '{colors.ink}'
    radius: '{rounded.lg}'
  sms-bubble-out:
    background: '{colors.sms-bubble-out}'
    foreground: '{colors.ink}'
    radius: '{rounded.lg}'
---

# Sarati — Design Spine

> Visual identity for the dual-channel receivables-and-consent OS. Paired with `EXPERIENCE.md`. Inherits shadcn/ui + Tailwind on Next.js (`apps/portal`, `apps/checkout`). This file specifies the brand-layer delta only. Spines win on conflict with any mock.

## Brand & Style

**Product:** Sarati · **Domain:** sarati.net · Trust chrome remains Merchant name + neighbourhood (public wordmark may use Sarati).


This product is a neighbourhood shop's ledger that happens to live on a phone — not a neon fintech, not a card network, not a Super-App. The visual job is to make money conversations feel **named, calm, and local**: Awa's shop name and neighbourhood do more trust work than any platform wordmark.

The aesthetic is **paper + shop awning**. Warm off-white surfaces read as a notebook, not a bank dashboard. The only chromatic brand color is an **olive earth green** that suggests a painted metal awning in Ouaga — evocative of Burkina without lifting the national flag (red / flag-green / gold / star) into chrome. A **clay accent** marks the one live action on a screen. Everything else stays ink, paper, and hairline earth borders.

Hosted Checkout is a **Merchant-named page**, not a platform stage. Fatou must recognize the cleaner she already knows. The platform is a quiet footer: reminder-then-pay, never "prélèvement automatique."

SMS is a first-class surface. Designed verbs sit as **bold chips**, not as app icons. The transcript is the OS.

**[ASSUMPTION]** Working name remains **BF Recurring Payments**. A public consumer wordmark is not locked. Do not invent one in chrome.

## Colors

Light-only MVP. Outdoor glare and mid-3G Androids are the contrast test, not dark-mode elegance.

- **Paper (`{colors.surface}` `#F7F4EE`)** — portal and checkout canvas. Warm enough to feel like a notebook; high enough luminance for sunlit courtyards.
- **Raised (`{colors.surface-raised}` `#FFFFFF`)** — cards, sheets, SMS inbound bubbles, Payout Clock.
- **Ink (`{colors.ink}` `#1C1915`)** — all primary text. Near-black, slightly warm. Never cool gray body text.
- **Muted ink (`{colors.ink-muted}` `#534E45`)** — meta, timestamps, "cible T+1." Must stay ≥ 4.5:1 on Paper.
- **Hairline earth (`{colors.border}` `#C9C1B2`)** — row and card edges. No drop-shadow hierarchy.
- **Olive awning (`{colors.primary}` `#3F5A3A`)** — primary buttons, active nav, focus ring companion. **Not** BF flag green. Never a full-bleed flag stripe.
- **Clay (`{colors.accent}` `#A65D28`)** — the single live action on a dense surface (Cash Mark-paid on a late row; PAY affordance in a transcript legend). Never used for "late," never used for chrome.
- **Status inks** — `{colors.success-ink}` Payé · `{colors.late-ink}` En retard · `{colors.pending-ink}` En attente de consentement · `{colors.hold-ink}` KYC hold · `{colors.destructive-ink}` real errors only. Status is **word + icon + ink**. Color fill alone is banned.

Avoid: neon teal, card-network red/yellow, gold-star motifs, gradient "fintech" meshes, `$` or USD green, English-default gray chrome.

Contrast targets (load-bearing):

| Pair | Target |
|---|---|
| `{colors.ink}` on `{colors.surface}` | ≥ 12:1 |
| `{colors.ink-muted}` on `{colors.surface}` | ≥ 4.5:1 |
| `{colors.primary-foreground}` on `{colors.primary}` | ≥ 7:1 |
| `{colors.success-ink}` / `{colors.late-ink}` on `{colors.surface-raised}` | ≥ 4.5:1 |

## Typography

System stack only — `{typography.body.fontFamily}`. No webfonts on checkout or SMS-adjacent pages (3G, NFR-2). French-friendly: generous x-height from system UI fonts; no condensed cuts; no all-caps sentences.

| Role | Use |
|---|---|
| `{typography.display}` | One hero per sitting: shop name on checkout; "Bienvenue, Awa" only on first Accueil. |
| `{typography.display-sm}` | Surface titles (Accueil, Clients, Plan). |
| `{typography.body}` | Forms, consent copy, SMS bodies in mocks. |
| `{typography.body-strong}` | Amounts (`2 000 F CFA`) and Merchant names in lists. |
| `{typography.label}` | Field labels, status pill text, nav. |
| `{typography.meta}` | Due dates, MSISDN, Payout Clock sub-lines. |
| `{typography.verb}` | SMS verbs only: PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE. |

Amounts always read **`2 000 F CFA`** in Merchant/Customer UI. Developers see **XOF** in docs and API examples. Never `$`.

## Layout & Spacing

Tailwind 4-based scale inherited; brand adds `{spacing.tap}` (44px) as the minimum interactive height/width.

- **Portal** — single column, `{spacing.max-portal}` (720px). Mobile-first. Bottom tab bar on `< md`; top nav on `lg+`.
- **Checkout** — single column, `{spacing.max-checkout}` (420px), phone-shaped even on desktop. Linear four steps; no sidebar.
- **SMS** — transcript column, inbound left / outbound right, 16px gutters.
- Gutters: `{spacing.gutter-mobile}` on phones, `{spacing.gutter-desktop}` on `lg+`.
- Modal stacks **one** level. Staff PIN is the only overlay that may sit on a confirm sheet — treat PIN as replacing the sheet, not stacking.

First sitting is a **wizard in the content column**, not a multi-step marketing carousel. One primary button per screen.

## Elevation & Depth

No elevation as hierarchy. Cards are Paper vs Raised plus `{colors.border}`. Sheets (Staff PIN, Cash Mark-paid) use the shadcn Sheet edge + a 1px earth border. Toasts are text-first, low shadow, never celebratory.

Checkout trust comes from **named type**, not from a lock-icon fortress or SSL theater.

## Shapes

Softer than a developer tool, tighter than a consumer "pill app." `{rounded.sm}` inputs, `{rounded.md}` buttons and rows, `{rounded.lg}` cards / Payout Clock / SMS bubbles, `{rounded.full}` status pills and verb chips only.

No flag-star geometry. No card-network lozenges.

## Components

shadcn used as-is (visual defaults, brand colors via tokens): `Button`, `Card`, `Dialog`, `Sheet`, `Input`, `Label`, `Badge`, `Toast`, `Tabs`, `Separator`, `Skeleton`, `DropdownMenu`. Do not restyle those primitives beyond tokens below.

Brand-layer components:

- **Button (primary)** — `{colors.primary}` fill, `{colors.primary-foreground}` text, `{rounded.md}`, min-height `{spacing.tap}`. Label is a French verb: *Enregistrer*, *Inviter*, *Marquer payé*, *Réessayer*.
- **Button (secondary)** — Raised + earth border. Used for Skip, later, dismiss.
- **Status pill** — Word + simple geometric icon (circle / dash / pause bars). Inks from the status set. Never a color-only dot.
- **Payout Clock** — Two-row card: **Client payé** (immediate, `{colors.success-ink}` or pending-ink) and **Vous recevez** (target T+1, labeled *cible, pas une garantie*, or *en attente de règlement* + last Rail status). Collected-but-not-settled must not look like Échec.
- **Trust chrome** — Checkout and SMS header: Merchant display name in `{typography.display-sm}`, neighbourhood in `{typography.meta}`. No card logos. No English. No `$`.
- **KYC hold banner** — Clay-tinted paper (`{components.kyc-banner}`), `{colors.hold-ink}` text. States that **payouts are held**, invoicing continues. Never "compte bloqué."
- **Receivables row** — Name, MSISDN, amount, due date, status pill. Cash Mark-paid is a clay-accent text button on open / failed rows — first-class, not a footnote.
- **Staff PIN pad** — Numeric, 44px keys, no branding flourish. Title: *Code équipe*.
- **Consent card** — Checkout step 2. Named Merchant, amount or amount rule, frequency, next due date, reminder-then-pay sentence, STOP ≠ ANNULER disclosure. Confirm checkbox is text, not a legal wall of 9pt gray.
- **SMS bubbles** — Inbound Raised, outbound olive-tinted `{colors.sms-bubble-out}`. Verbs in `{typography.verb}` as chips inside the body.
- **Retry inbox row** — Customer name, MSISDN, last Rail status, timestamp, *Réessayer* + *Marquer payé*.
- **OTP field** — six equal boxes (or one numeric input of `{spacing.tap}` height), `{typography.display-sm}`, tracking wide. No password-manager chrome labeled "password."
- **Method picker** — one full-width row: Orange Money wordmark-as-text (no card-network art). Selected = `{colors.primary}` hairline, not a rainbow tile.
- **Empty state** — `{typography.display-sm}` sentence + one primary button. No illustration mascots.
- **Verb chip** — `{typography.verb}`, `{rounded.full}`, olive ink on `{colors.sms-bubble-out}` or Paper. Used only for the seven designed verbs.

→ Composition reference: `mockups/portal-first-sitting.html`, `mockups/receivables-book.html`, `mockups/hosted-checkout.html`, `mockups/sms-transcript.html`. Spine wins on conflict.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Lead with Merchant name + neighbourhood | Lead with a coined platform logo or English wordmark |
| Olive awning + clay accent, paper surfaces | BF flag as chrome (red/green/gold/star) |
| `2 000 F CFA` | `$20`, `XOF 2000` in Customer/Merchant UI |
| Status = word + icon + ink | Color-only dots for Payé / En retard |
| Cash Mark-paid as a primary-size control | "Payer en espèces (déconseillé)" apology |
| Silence after success (one receipt) | Confetti, streak, "Félicitations !!!" |
| French-first, 8th-grade consent | "Auto-debit", "prélèvement automatique", "set and forget" |
| System fonts, 44px targets | Webfont display serifs on checkout |
| One live clay action per dense screen | Accent used as late-state or nav highlight |
