# earthity — Routes & Cross-Property Map (canonical)

Companion to [`architecture.md`](./architecture.md) and
[`analytics-events.md`](./analytics-events.md).

## Marketing site (`earthity-website` → `www.earthity.com`)
| Path | File | Notes |
|---|---|---|
| `/` | `public/index.html` | Home (hero, solutions, product tiles, customer stories) |
| `/outpost/` | `public/outpost.html` | Outpost. product page |
| `/drone-program/` | `public/drone-program.html` | Drone Program Integration; **6-step Stack stepper** (v2 instrumentation target) |
| `/charging-dock/` | `public/charging-dock.html` | Charging Dock product page |
| `/about/` | `public/about.html` | Company / founders |
| `/contact/` | `public/contact.html` | Contact form (`#contact-form`) → SheetMonkey |
| `/data-room/` | `public/data-room.html` | Investor gate (`#data-room-form`) → SheetMonkey → Google Drive |
| `/privacy`, `/terms` | `public/privacy.html`, `public/terms.html` | Legal |
| `/deck/` | submodule | Self-contained pitch deck (no analytics) |

Shared nav + footer are JS-injected partials (`public/partials/{nav,footer}.js`). The footer's
final-CTA strip (host calculator / operator join / Calendly) appears on every partial-using page.

## Outpost app (`earthity-dev` → `outpost.earthity.com`)
| Path | File | Notes |
|---|---|---|
| `/` | `app/page.tsx` | Central sign-in (`AuthLanding`); authed users redirect to persona dashboard |
| `/host/onboard` | `app/host/onboard/page.tsx` | `HostFunnel` — 4 steps (Monetize → Check roof → See earnings → Claim spot). Public. |
| `/operator/onboard` | `app/operator/onboard/page.tsx` | `OperatorFunnel` — 6 steps (Welcome → Pick dock → Add drone → Place on map → Test mission → Create account). Public. |
| `/host`, `/operator` | `app/host`, `app/operator` | Persona dashboards (post-auth) |
| `/(auth)/*` | `app/(auth)/…` | sign-in / sign-up / callback (magic-link + Google) |

Signup happens at the **final funnel step** (`ClaimSpot` host step 4 / `CreateAccount` operator step 6);
`identify()` + `signed_up` fire post-auth on the next dashboard load.

## Cross-property link map (www CTA → app route)
| www `cta` (data-url-key) | resolves to | app entry |
|---|---|---|
| `host` | `outpost.earthity.com/host/onboard` | `HostFunnel` → `host_onboard_step_viewed{step:1}` |
| `fly` / `operatorOnboard` | `outpost.earthity.com/operator/onboard` | `OperatorFunnel` → `operator_onboard_step_viewed{step:1}` |
| `signIn` | `outpost.earthity.com/` | `AuthLanding` |
| `bookCall` | `calendly.com/bhowmiktanumaya/tanumaya` | (external — Calendly) |
| `contact` | `/contact/` (www-internal) | — |

`fly` and `operatorOnboard` resolve to the **same** operator pipeline; they remain distinct CTA labels
so placement intent is preserved on the `cta` property. Canonical destinations live in
`analytics-contract.json` → `ctaDestinations`.
