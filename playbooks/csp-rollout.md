# CSP rollout playbook — earthity design system consumers

A copy-pasteable runbook for taking a static site that consumes this design system from "no CSP" to "enforced strict CSP." Project-agnostic; the consumer keeps a session-specific runbook for actual commits and decisions.

---

## When to use

- Site has no CSP today (or only `default-src 'self' 'unsafe-inline' 'unsafe-eval'` — same risk profile).
- Site consumes this design system (the `style-src` / `font-src` allowances below assume `earthity-tokens.css` is loaded — see the CSP note at the top of that file).
- You can deploy iteratively (e.g., every push to `main`).

If the site has a complex auth surface, server-side render layer, or embedded SaaS widgets, this playbook needs adapting.

---

## Phases (5 tracks)

| # | Track | Output | Exit criterion |
|---|---|---|---|
| 1 | Inline-pattern migration | every `style=` / `<script>` / `<style>` extracted to external files (or accepted as exception) | inline-pattern audit reports zero non-accepted violations |
| 2 | Static security headers | host config ships HSTS + nosniff + frame-options + referrer + permissions + COOP + CORP | `curl -I` shows all 7 |
| 3 | CSP report-only | `Content-Security-Policy-Report-Only` header live with strict directive set | `curl -I` shows it; ≥7 days console-clean |
| 4 | CSP enforce | rename header to `Content-Security-Policy` | `curl -I` shows enforced; site still functions |
| 5 | Monitoring window | report endpoint live for 30 days post-enforce | no new violations |

Tracks 1 and 2 are independent and can run in parallel. Track 3 requires track 1 done. Track 4 requires track 3 monitoring done. Track 5 begins at track 4.

---

## Track 1 — Inline-pattern migration

Run an inline-pattern audit on the deployable directory. Common findings:

- inline `<script>` blocks (incl. JSON-LD)
- inline `<style>` blocks
- inline `style=` attributes (often the largest count by far)
- inline `on*=` event handler attributes
- `data:` URIs in `src=`/`href=`

**Per-section migration cadence:** identify natural sections of the page (hero, nav, footer, each product block). Migrate one section per commit — extract inline patterns into BEM-ish component classes in your consumer stylesheet, re-run the audit, smoke-test the page in a browser, commit.

Class-naming convention: see `../CLAUDE.md` §5b. Reuse `.eu-section-header` from `earthity-components.css` instead of rolling your own.

> ⚠️ **Critical gotcha — `[style*="..."]` media queries.** Before extracting inline styles, grep your stylesheet for `@media` rules that target inline-style attribute selectors:
>
> ```bash
> grep -rn '\[style\*=' your-css-dir/
> ```
>
> Those rules silently break when the inline style they target is removed — responsive layout snaps back to desktop on smaller viewports with no error. Rewrite each `[style*="..."]` selector to use the new component class **in the same commit** as the inline-style removal. Also documented in `../CLAUDE.md` §5b.

### Accepted exceptions

Some inline patterns are by-design and the audit will flag them forever. Don't migrate; accommodate in the CSP directive set:

| Pattern | Why kept | CSP allowance |
|---|---|---|
| `<script type="application/ld+json">` | search-engine structured data | hash via `'sha256-…'` in `script-src` |
| `<defs><style>` inside an SVG file loaded via `<img src="…svg">` | SVG-as-image is sandboxed; inline style does not execute against the parent CSP | none — false positive |
| `data:` URI inside an `<img>`-loaded SVG (e.g. inlined AVIF) | inlines the only instance of a small binary; saves a request | `img-src 'self' data:` |

Document each exception in your project's migration-tracking doc so a future session knows it's a decision, not a violation.

---

## Track 2 — Static security headers

Independent of track 1. Ship the boring-but-required headers first so the site is hardened against MIME sniffing, clickjacking, referrer leak, etc., before tackling CSP.

Vercel example (other hosts: similar map; consult host docs):

```json
{ "key": "X-Content-Type-Options",       "value": "nosniff" },
{ "key": "X-Frame-Options",              "value": "DENY" },
{ "key": "Referrer-Policy",              "value": "strict-origin-when-cross-origin" },
{ "key": "Permissions-Policy",           "value": "camera=(), microphone=(), geolocation=(), interest-cohort=(), payment=(), usb=()" },
{ "key": "Cross-Origin-Opener-Policy",   "value": "same-origin" },
{ "key": "Cross-Origin-Resource-Policy", "value": "same-origin" }
```

HSTS often comes from the host by default; check before overriding.

Verify after deploy: `curl -I https://<your-host>/` should show all 7 headers (HSTS + the 6 added).

---

## Track 3 — CSP report-only

**Prerequisite:** track 1 audit shows only accepted exceptions.

### Compute the JSON-LD hash (if you accepted that exception)

```bash
node -e "
const crypto = require('crypto');
const fs = require('fs');
const html = fs.readFileSync('<path-to-html>', 'utf8');
const m = html.match(/<script type=\"application\/ld\+json\">([\s\S]*?)<\/script>/);
if (!m) { console.error('no JSON-LD found'); process.exit(1); }
const hash = crypto.createHash('sha256').update(m[1]).digest('base64');
console.log('sha256-' + hash);
"
```

Output is `sha256-XXXXXXXX…` — paste into `script-src`.

> ⚠️ **LF↔CRLF caveat.** The hash is over the EXACT bytes between `<script>` and `</script>`. If `git config core.autocrlf` flips line endings on deploy (Windows hosts especially), the deployed file's bytes won't match the local hash and the browser will block the script. If reports show JSON-LD violations after deploy: re-run the recipe on the deployed file's content, or set `.gitattributes` for the source HTML to `text eol=lf`.

### CSP directive set

Default starting point for a design-system consumer:

```
default-src 'self';
script-src 'self' 'sha256-<HASH>';
style-src 'self' https://fonts.googleapis.com;
img-src 'self' data:;
font-src 'self' https://fonts.gstatic.com;
connect-src 'self';
base-uri 'self';
form-action 'self';
frame-ancestors 'none';
upgrade-insecure-requests
```

Adjust per the accepted-exceptions table above:
- Drop `'sha256-<HASH>'` from `script-src` if no inline `<script>` is intentionally kept.
- Drop `data:` from `img-src` if no `data:` URI is kept.

The `style-src https://fonts.googleapis.com` and `font-src https://fonts.gstatic.com` allowances are **required** because `earthity-tokens.css` opens with an `@import url('https://fonts.googleapis.com/...')`. The `@import` fetch inherits the parent `style-src` directive (NOT `font-src`, NOT `connect-src`); the actual WOFF2 font files are then fetched under `font-src`. To drop both: vendor the CSS into your repo and self-host the WOFF2s.

Ship as `Content-Security-Policy-Report-Only`. Without a `report-uri` / `report-to` endpoint, browsers log violations to the console only — sufficient for initial monitoring; add an endpoint for higher-fidelity collection (see your security-headers skill if you have one).

### Monitoring period

**Run report-only for the longer of: 7 days of real traffic, OR 48h with zero new violation types.** Open the deployed site in browser dev tools → Console; click through every interactive surface (nav, modals, every CTA, every form, every outbound link). Any "Refused to load…" / "Refused to execute…" entries are violations to fix BEFORE flipping to enforce.

---

## Track 4 — CSP enforce

After clean monitoring:
- Rename `Content-Security-Policy-Report-Only` → `Content-Security-Policy`.
- Drop `X-Frame-Options: DENY` IF `frame-ancestors 'none'` is in the policy (CSP3 supersedes; older browsers still respect XFO, so keeping both is also fine for defense-in-depth).

Deploy. `curl -I` should show the enforced header. Smoke-test the same surfaces.

---

## Track 5 — Long-tail monitoring

Leave the report endpoint live (or keep checking dev-tools console after each deploy) for **30 days post-enforce**. New violations almost always mean:
- A new third-party dependency added without updating the directive set.
- The design system was bumped and pulled in a new origin (font weight, icon CDN, etc.).
- A copy-pasted snippet brought back an inline `style=`/`<script>`.

Each one needs a one-line directive update or a rollback of the change that caused it.

---

## Re-baseline cadence

| Changed | Re-run |
|---|---|
| HTML / CSS / JS inline patterns | inline-pattern audit |
| Host config headers (after redeploy) | `curl -I https://<your-host>/` |
| HTML payload, render-blocking assets, hero image, fonts | Lighthouse mobile (perf budget impact) |

If the project tracks progress in a baseline-driven dev-docs folder (`migration-plan.md` + `<domain>-findings.md`), refresh those after each meaningful change so Δ-from-baseline is preserved over time.
