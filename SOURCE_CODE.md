# Sidecar Setup Guide V2 --- Source Code & Agent Handoff

## Purpose

This file explains the V2 frontend behavior and the constraints an AI
coding agent must preserve when evolving the prototype into the paid
production product.

## Current V2 architecture

The current prototype intentionally remains a single static `index.html`
so it can replace the existing GitHub Pages prototype immediately.

It contains:

-   responsive app shell
-   desktop sidebar
-   mobile drawer
-   six chapters
-   Overview
-   Troubleshooting
-   FAQ
-   local progress
-   copyable terminal commands
-   screenshot placeholders
-   explicit access-status placeholder

This is **not yet the final production security architecture**.

## Layout contract

Desktop:

``` text
100dvh application
├── fixed-height header
└── remaining viewport
    ├── sidebar
    │   ├── internally scrollable chapter area
    │   └── bottom utilities
    └── independently scrollable main guide
```

Do not reintroduce body-level scrolling that causes the bottom sidebar
utilities to disappear below the viewport.

Mobile:

``` text
Header
├── Menu button
└── Access status

Guide content

Sidebar → off-canvas drawer
```

## Guide structure

``` text
Overview

01 Before You Start
02 Connect Your iPad
03 Configure BetterDisplay
04 Make It Automatic
05 One-Tap Connection
06 Go Headless

Troubleshooting
FAQ & Safety Notes
```

## Content rules

### Copyable

Keep these selectable/copyable:

-   Terminal commands.
-   SSH commands.
-   Automation scripts.
-   Required configuration/path values.

### Casual copy deterrence only

Guide prose may use `user-select: none`.

This is not DRM or authorization.

Never corrupt commands with invisible characters or watermarks.

## Progress

Current prototype:

``` text
localStorage key:
sidecar-guide-v2-progress
```

Production must migrate progress to authenticated Supabase records.

Progress and entitlement are separate concepts.

Progress must never grant paid access.

## Screenshot placeholders

Replace `.shot` placeholders with real screenshots supplied and tested
by the product owner.

Recommended future structure:

``` text
assets/
  chapter-01/
  chapter-02/
  chapter-03/
  chapter-04/
  chapter-05/
  chapter-06/
```

Avoid publishing personal Apple Account/email, serial number, SSH
credential, or unnecessary device identifiers.

## Access button

The current Access button intentionally displays a message explaining
that the GitHub Pages build is not protected.

Do not replace it with fake JavaScript authentication.

Production flow:

``` text
Visitor
   ↓
Supabase Auth
   ↓
Check entitlement
   ├── inactive → Get Access
   └── active → Private Guide
```

## Production database direction

Conceptual tables:

``` text
profiles
entitlements
guide_progress
devices_or_sessions
```

Entitlement statuses:

``` text
pending
active
refunded
revoked
```

Use Supabase RLS.

Users must not be able to grant themselves `active` entitlement from
browser code.

## Payment direction

Initial provider:

``` text
Lemon Squeezy
```

Expected production flow:

``` text
Get Access
   ↓
Checkout
   ↓
Payment provider
   ↓
Verified + idempotent webhook
   ↓
Supabase entitlement
   ↓
Private guide unlocked
```

Future architecture should allow Lynk/QRIS without rewriting guide
authorization.

Keep provider-specific code separate from entitlement logic.

## Watermark direction

After authentication, paid guide/screenshots may display a subtle
watermark such as:

``` text
ar***@gmail.com
SG-000142
```

Never watermark exact commands/scripts.

Watermark is a leak deterrent, not screenshot prevention.

## Session direction

Business rule:

``` text
1 purchase = 1 account
```

Preferred MVP behavior may use one active session/device at a time, but
implementation must remain recoverable for legitimate users changing
devices.

Avoid invasive fingerprinting.

## Security rules

Production must have:

-   Supabase RLS.
-   Server-side entitlement validation.
-   Protected premium content.
-   Server-only service-role/payment secrets.
-   Verified webhook signatures.
-   Idempotent webhook handling.
-   Safe refund/revocation flow.
-   No secrets committed to Git.
-   No false claims that screenshots can be prevented.

## Technical content caveats

Sidecar is a local display solution, not remote desktop.

Login Items/Automator do not bypass FileVault or the macOS login screen.

The full reconnect automation runner must not be marked verified until
the owner tests it.

## Agent implementation order

``` text
1. Preserve/QA this V2 responsive experience.
2. Integrate owner-approved screenshots and final content.
3. Refactor to maintainable production architecture if needed.
4. Add Supabase.
5. Add database migrations + RLS.
6. Add Auth.
7. Add entitlement-protected guide delivery.
8. Move progress to backend.
9. Add approved session/device policy.
10. Add watermark.
11. Deploy production app to Cloudflare.
12. Add Lemon Squeezy checkout.
13. Add verified webhook.
14. Handle refund/revocation.
15. Run responsive/security/payment/end-to-end QA.
```

## Do not do

-   Do not put premium content behind CSS-only hiding.
-   Do not call GitHub Pages private.
-   Do not put payment secrets in frontend JavaScript.
-   Do not disable Copy Command.
-   Do not invent untested screenshots.
-   Do not claim the setup bypasses FileVault.
-   Do not claim screenshots are impossible.
-   Do not add multiple payment gateways before the first provider is
    stable.

## Definition of frontend V2 complete

-   desktop viewport/sidebar bug fixed
-   mobile drawer works
-   six chapters work
-   Overview/Troubleshooting/FAQ work
-   progress persists
-   command copy works
-   command text remains selectable
-   no critical horizontal overflow
-   screenshot slots ready
-   access placeholder clearly marked as non-production

## Definition of final production complete

Frontend completion is not final product completion.

Final production additionally requires:

-   Supabase Auth
-   RLS
-   entitlement
-   protected premium delivery
-   backend progress
-   session policy
-   watermark
-   Lemon Squeezy checkout
-   verified/idempotent webhook
-   refund/revoke
-   Cloudflare production deployment
-   domain/HTTPS
-   end-to-end non-admin customer test
