# earthity — Analytics Event Contract (canonical)

The human-readable companion to [`analytics-contract.json`](./analytics-contract.json) (the
machine-readable core that code/tests consume). Companions:
[`architecture.md`](./architecture.md), [`routes.md`](./routes.md).

> **Change protocol:** edit `analytics-contract.json` first (event names, `cta` enum, `regions`,
> `freeEmailProviders`), then update both consumers in lockstep. Renames touch both repos in one
> coordinated change.

## Naming convention
`snake_case`, **past-tense** verb, object/persona prefix. Matches the app's `events.ts` house style
(`signed_up`, `demo_entered`, `*_onboard_step_viewed`). The website conforms to the app.

## Website events (`earthity-website`, captured in `public/js/analytics.js`)
| Event | Fires when | Properties |
|---|---|---|
| `cta_clicked` | click on any primary CTA anchor (`data-url-key` ∈ host/fly/operatorOnboard/signIn/bookCall, **or** href `/contact/`). One physical click = one event. | `cta, page, section, label, destination, is_outbound` |
| `lead_submitted` | contact form POST returns 200 (delivery, not attempt) | `form, page, source, email_domain, email_type` |
| `newsletter_subscribed` | newsletter POST returns 200 | `page, email_domain, email_type` |
| `data_room_requested` | valid submit of `#data-room-form` | `page, email_domain, email_type` |
| `data_room_accessed` | success branch, just before the Drive redirect | `page, email_domain, email_type` |
| `stack_step_view` | a drone-program Stack step is opened (nav click / card expand) | `step, step_label, page` |

`$pageview` (built-in, `capture_pageview:true`) anchors page-to-page funnels and auto-carries
`$current_url` + `utm_*` + `$referring_domain`. `$autocapture` is a **discovery layer only** — never a
funnel step.

The old `outbound_outpost_click` is **retired**: an outpost-bound CTA is the single `cta_clicked` with
`destination:'outpost'` + `is_outbound:true`.

## App events (`earthity-dev`, captured via the typed `capture()` wrapper)
Client (acquisition): `signed_up{method}`, `demo_entered`, `dashboard_viewed{persona}`,
`operator_onboard_step_viewed/completed{step,stepName,archetype?}`,
`operator_onboard_completed{archetype?}`, `host_onboard_step_viewed/completed{step,stepName,intendedUse?}`,
`host_onboard_completed{intendedUse?}`.

Client (dashboard behavior — demo **and** real; every event carries the `is_demo` super-property so
demo exploration segments cleanly from real-user activity): `dashboard_section_viewed{persona,section}`,
`entity_detail_opened{persona,kind}`, `entity_detail_panel_toggled{persona,kind,panel,open}`,
`entity_edited{persona,kind,field}`, `add_flow_started{persona,kind}`,
`demo_milestone_reached{persona}` (demo-only; fires once per demo session on first meaningful
action — the trigger for the surgical demo survey). Demo sessions stay **anonymous**
(never `identify()`'d — every anon session is the one demo@outpost.com row) but stitch to the real user
on signup. Note: distinct from the website's deferred `section_view` (scroll depth).

Client (first-login profile capture — real users only, after the persona prompt):
`profile_prompt_shown{persona}`, `profile_completed{persona}`, `profile_prompt_dismissed{persona}`.
Collects name + org (the funnel defers identity); skippable but re-shows until completed. Payloads carry
`persona` only — **no name/org/email** (PII stays out of analytics). The name/org writes are audited
(`profile_updated`, entityType `profile`).

Client (org join requests — the user-initiated path to join an existing org, admin-approved):
`join_requested{persona,via}` where `via ∈ {domain,search}` (email-domain banner vs name typeahead). No org
name/id in the payload. Server-truth request lifecycle is audited (`join_requested` / `join_request_approved`
/ `join_request_denied`, entityType `join_request`) → also dual-emitted to PostHog as `audit_event`.

Client (performance): `client_fetch_timed{path,durationMs,ok,status}`,
`optimistic_commit_timed{persona,durationMs,ok}`. Server-side DB-query timing is **not** a PostHog event
(a Prisma hook has no serverless flush point) — slow queries go to `AppLog` (admin System tab) via `log.warn`.

Server (outbox relay): `intent_pin_created`, `site_created`, `plot_created`, `network_created`.

## Property dictionary
See `analytics-contract.json` → `properties`. Highlights: `page` = normalized `location.pathname`;
`section` = `data-cta-section`; `destination` = resolved-href host (or `'outpost'`); `is_outbound` =
host ≠ page host; `email_domain` = domain only (no local part); `email_type` = `corporate|free` via
`freeEmailProviders`. **No raw email** is ever sent to PostHog (it stays in SheetMonkey).

## Canonical funnels (build in the PostHog UI; order **Sequential**, window **7–14d**)
| Funnel | Steps |
|---|---|
| Host acquisition | `$pageview` → `cta_clicked{cta:'host', destination:'outpost'}` → app `host_onboard_step_viewed{step:1}` → `host_onboard_completed` → `signed_up` |
| Operator acquisition | `$pageview` → `cta_clicked{cta∈[fly,operatorOnboard]}` → app `operator_onboard_step_viewed{step:1}` → `operator_onboard_completed` → `signed_up` |
| Program lead | `cta_clicked{cta:'contact'}` → `$pageview(/contact/)` → `lead_submitted` |
| Book-a-call (SQL) | `$pageview` → `cta_clicked{cta:'bookCall'}` |
| Investor / data-room | `$pageview(/data-room/)` → `data_room_requested` → `data_room_accessed` |
| Cross-property www→outpost | anonymous `$pageview`(www) → `cta_clicked{destination:'outpost'}` → app onboarding → `signed_up` (stitched by `distinct_id`) |
| Demo engagement (`is_demo:true`) | `demo_entered` → `dashboard_viewed` → `entity_detail_opened` → (`entity_edited` \| `add_flow_started`) → `cta_clicked{cta:'signIn'}` → `signed_up` |
| Drone-program read-depth | `$pageview(/drone-program/)` → `stack_step_view{step:'01'}` → `…{step:'03'}` → `…{step:'06'}` → `cta_clicked{cta:'contact'}` |

Breakdown dimensions: `page, section, cta, utm_source, $referring_domain, email_type`.

## How each repo consumes this contract
- **app** — `import` `analytics-contract.json`; source `regions.ts` from `regions.required`; may
  validate event names against `events.app`.
- **www** — `public/js/analytics-events.js` is the hand-authored runtime copy (no build step); a
  Playwright test asserts it equals `events.website` + `cta`. `middleware.js`'s region array is
  asserted equal to `regions.required`.

## Deferred to v2
`section_view`/scroll depth, a generic `outbound_click` (partner links, LinkedIn, pitch deck), and an
autocapture `css_selector_allowlist`.
