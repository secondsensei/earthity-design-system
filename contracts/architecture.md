# earthity — System Architecture (canonical)

Single source of truth for how earthity's web properties fit together. Lives in the
`earthity-design-system` submodule so both consuming repos read one copy. Companions:
[`routes.md`](./routes.md), [`analytics-events.md`](./analytics-events.md),
[`analytics-contract.json`](./analytics-contract.json).

## Two properties, one analytics project
| Property | Repo | Stack | Host |
|---|---|---|---|
| Marketing site | `earthity-website` | Static HTML/CSS/vanilla JS, no build | `www.earthity.com` (apex 308→www) |
| Outpost app | `earthity-dev` | Next.js | `outpost.earthity.com` |

Both send to **one PostHog project** (`NEXT_PUBLIC_POSTHOG_KEY` =
`phc_wG7ChQBd9u6jBdNag4qGUbDYBpiFyJVKDkRBtAKkvGLG`), so a visitor's marketing journey and product
onboarding are a single, joinable funnel.

## Shared PostHog invariants (keep identical on both sides)
- `api_host: '/ingest'` — first-party reverse proxy (www `vercel.json`, app `next.config.ts`), so CSP
  stays `connect-src 'self'` / `script-src 'self'` and the loader is ad-blocker resilient.
- `person_profiles: 'identified_only'` — anonymous events flow and funnel by `distinct_id`; person
  profiles are created only when the **app** calls `identify()`.
- `cross_subdomain_cookie: true`, cookie domain `.earthity.com` — the load-bearing identity stitch
  across `www` ↔ `outpost`.
- `disable_session_recording: true` (Tier-1, value-first).

## Identity model
- **www stays fully anonymous** — never calls `identify()`/`register()`/`group()`. All breakdown
  dimensions are **event** properties, never person properties.
- The **app owns identity** — `identify(user.id, {email,name,persona})` at signup
  (`PostHogProvider.tsx`), which merges the prior anonymous `distinct_id` via `$anon_distinct_id`.
  `reset()` is test-only (never before `identify()`), so the merge is safe.
- **The stitch:** anonymous visit on `www` writes a `distinct_id` on the `.earthity.com` cookie → the
  app reads the same cookie → at signup `identify()` merges all prior `www` events into the person.
- **URL fallback (hardening):** `www` appends `?distinct_id=<id>` (+ pass-through `utm_*`) to
  outpost-bound CTA hrefs; the app reads it as `bootstrap.distinctID` when no stored id exists.
  Covers Safari ITP / Brave cookie partitioning where the cookie alone may not carry. Param name is
  pinned in `analytics-contract.json` (`urlStitch.param`).

## Consent / geo model
- Edge middleware classifies `x-vercel-ip-country` into `geo_consent` = `required | free` (180-day
  strictly-necessary cookie). **Fail-closed:** missing/unknown country → `required`.
- `required` bloc = EEA (EU-27 + IS/LI/NO) + UK + CH = **32 codes** (canonical in
  `analytics-contract.json` → `regions.required`; mirrored in `earthity-website/middleware.js` and
  `earthity-dev/lib/analytics/regions.ts`).
- `free` → captured (opt-in default). `required` → opt-out by default; www is silent for these
  visitors (no banner). **Consequence:** funnels read **`free`-region traffic only** at the top — not
  total demand. Document this when interpreting numbers.

## Submodule wiring
Both parents pin `earthity-design-system` as a submodule. The website symlinks only a deployable
subset (tokens, components, fonts) into `public/`; this `contracts/` folder is **not** symlinked and
**never ships**. The app can `import` `contracts/analytics-contract.json` directly. **Deploy
ordering:** push a DS commit to its remote *before* bumping either parent's gitlink (Vercel can only
fetch what's already pushed).
