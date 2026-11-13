# Sidecar Setup Guide V2

Interactive setup guide for using an **iPad 9 as the main working
display for a Mac mini M4**.

Live V1 reference:\
https://firhyun.github.io/sidecar-setup-guide/

This repository version is the **V2 frontend/prototype foundation**.

## What changed in V2

-   Fixed viewport-height architecture using `100dvh`.
-   Header stays visible.
-   Sidebar fills the usable viewport.
-   Troubleshooting, FAQ, and Overview stay reachable at the bottom.
-   Main guide area scrolls independently.
-   Mobile/tablet sidebar becomes a drawer.
-   Six chapter navigation retained.
-   Progress persists in `localStorage`.
-   Terminal commands/scripts remain copyable.
-   Premium prose uses only light copy deterrence.
-   Screenshot slots are prepared for real owner-tested captures.
-   "Access status" is explicitly a placeholder; GitHub Pages is **not**
    treated as paid-content security.
-   Structure is ready to be migrated to Supabase Auth + entitlement
    later.

## Files

``` text
index.html       V2 interactive frontend
README.md        Setup/deployment notes
SOURCE_CODE.md   Architecture and agent handoff notes
```

## Run locally

No build step is required for this prototype.

``` bash
python3 -m http.server 8080
```

Then open:

``` text
http://localhost:8080
```

You can also open `index.html` directly, although a local HTTP server is
preferred.

## GitHub Pages

1.  Replace the existing repository `index.html` with the V2 file.
2.  Commit `README.md` and `SOURCE_CODE.md`.
3.  Push to the branch currently used by GitHub Pages.
4.  Open the deployed URL and hard refresh.
5.  Test desktop height, mobile drawer, progress, and Copy Command.

## Required content work before selling

Replace screenshot placeholders with real screenshots from the tested
Mac mini M4 + iPad 9 setup.

Verify every command/script on the actual devices.

The reconnect automation runner must not be described as verified until
it has actually been tested.

## Important security note

This V2 GitHub Pages version is still a **prototype/public frontend**.

`user-select: none`, disabled image dragging, hidden UI, or JavaScript
access checks are not real paid-content security.

Production paid access should use:

``` text
Supabase Auth
      ↓
Entitlement check
      ↓
Protected guide
```

Payment direction:

``` text
Lemon Squeezy
      ↓
Verified webhook
      ↓
Supabase entitlement
      ↓
Guide access
```

Do not put Supabase service-role keys, payment API keys, or webhook
secrets in `index.html`.

## V2 acceptance checklist

-   [ ] Header stays visible.
-   [ ] Sidebar fits viewport.
-   [ ] Troubleshooting / FAQ / Overview are reachable without
    whole-page scrolling.
-   [ ] Main guide scrolls independently.
-   [ ] Mobile drawer works.
-   [ ] No critical horizontal overflow.
-   [ ] All six chapters open.
-   [ ] Progress survives refresh.
-   [ ] Copy Command works.
-   [ ] Commands remain selectable.
-   [ ] Real screenshots replace placeholders before paid launch.
-   [ ] Technical steps are owner-tested.

## Next production phase

After this frontend is approved:

1.  Refactor into the chosen production framework if required.
2.  Connect Supabase Auth.
3.  Add `profiles`, `entitlements`, `guide_progress`, and approved
    session/device logic.
4.  Protect premium guide content server-side.
5.  Add personalized watermark.
6.  Deploy production app to Cloudflare.
7.  Integrate Lemon Squeezy.
8.  Verify payment webhook.
9.  Grant/revoke entitlement automatically.
10. Run end-to-end customer QA.

See `SOURCE_CODE.md` for implementation rules.
