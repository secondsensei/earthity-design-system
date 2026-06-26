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

`$pageview` (built-in, `capture_pageview:true`) anchors page-to-page funnels and auto-carries
`$current_url` + `utm_*` + `$referring_domain`. `$autocapture` is a **discovery layer only** — never a
funnel step.

The old `outbound_outpost_click` is **retired**: an outpost-bound CTA is the single `cta_clicked` with
`destination:'outpost'` + `is_outbound:true`.

## App events (`earthity-dev`, captured via the typed `capture()` wrapper)
Client: `signed_up{method}`, `demo_entered`, `dashboard_viewed{persona}`,
`operator_onboard_step_viewed/completed{step,stepName,archetype?}`,
`operator_onboard_completed{archetype?}`, `host_onboard_step_viewed/completed{step,stepName,intendedUse?}`,
`host_onboard_completed{intendedUse?}`. Server (outbox relay): `intent_pin_created`, `site_created`,
`plot_created`, `network_created`.

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

Breakdown dimensions: `page, section, cta, utm_source, $referring_domain, email_type`.

## How each repo consumes this contract
- **app** — `import` `analytics-contract.json`; source `regions.ts` from `regions.required`; may
  validate event names against `events.app`.
- **www** — `public/js/analytics-events.js` is the hand-authored runtime copy (no build step); a
  Playwright test asserts it equals `events.website` + `cta`. `middleware.js`'s region array is
  asserted equal to `regions.required`.

## Deferred to v2
`stack_step_view` (drone-program 6-step Stack stepper), `section_view`/scroll depth, a generic
`outbound_click` (partner links, LinkedIn, pitch deck), and an autocapture `css_selector_allowlist`.
