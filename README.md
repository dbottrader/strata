# STRATA — static deployment

STRATA is a scientific evidence atlas for reading the record across time:
50 evidence events in 9 classes on a shared clock, 9 case files with
competing models, a per-event receipt for every record, and a convergence
engine that tests independent streams for temporal overlap against a
shuffled-years null model.

**Live site:** https://dbottrader.github.io/strata/

## How this deployment is built

The app (TanStack Start + React + Nitro, built with Grok) targets Vercel
SSR. This repo's `gh-pages` branch is a static snapshot of that exact
build: every route was server-rendered by the built app and saved as
static HTML, with the production asset bundle served alongside. All data
is bundled in the client, so the atlas, case files, receipts and the
convergence engine run entirely in the browser.

Build/deploy workspace: `~/workspace/strata-pages-build` (a deployment
copy — the original app at `~/workspace/grok-atlas` is untouched).
Snapshot tooling: `/tmp/strata-snapshot.mjs` (route list + OG fixups).

## One fix applied in the deployment copy only

In the app as shipped, `/cases/<id>` was unreachable: the generated
route tree makes `cases.$id` a child of `cases`, but `cases.tsx`
rendered the case list with no `<Outlet/>`, so all 9 case detail pages
fell back to the list (verified in SSR output and by browser
click-through). The deployment copy applies the standard TanStack
split — list moved to `cases.index.tsx`, `cases.tsx` is now a
pass-through layout — and the detail pages render fully. The fix is
NOT in the original app; porting it upstream is a 2-file change.

## What does not survive static hosting (stated honestly)

- **AI interrogation on /converge** ("Ask Grok") calls a server function
  with an `XAI_API_KEY`; there is no server here, so that one button
  cannot answer. Everything else on /converge — windows, expansion
  slider, cluster engine, p-values — computes locally and works.
- **Multiplayer presence** (optional P2P rooms) has no signalling
  server; solo use is unaffected.
- **PWA install page / dynamic manifest**: replaced by a static
  manifest scoped to `/strata/`; the `?install=1` tutorial route is a
  server feature and is not present.
- `og.jpg` and `__grok/icon-180.png` are binary files; the connector
  push path is UTF-8 text only, so they may land in a follow-up push.
  Until then the social-card image and apple-touch-icon 404 (cosmetic
  only — all pages, styles and scripts are unaffected).

## Verification (2026-10-21)

- Production build clean (Vercel preset, base `/strata/`).
- Unit tests: 187 pass; 8 failures in
  `scripts/grok-pwa-plugin.test.mjs` are pre-existing — the pristine
  original fails the identical 8 (they need Grok-sandbox fixture
  identities). Zero regressions from the routing fix.
- Static snapshot rendered in headless Chromium under a plain static
  file server: all 14 routes HTTP 200, zero page errors, case detail
  and convergence engine confirmed visually, client-side navigation
  list → case detail works.
