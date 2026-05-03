# earthity Design System — Consumer Guide

This file documents how to consume the earthity design system from a product project. It is loaded into Claude Code's context when working inside the design system or any consumer project that vendors it.

The system is **CSS-first** (no JS framework opinion) and **submodule-distributed**. Two stylesheets, one icon set, a `README.md` for the visual brief, and preview pages that document every component in isolation.

---

## 1. Add the system to a consumer project

Add as a git submodule at the consumer project's root:

```bash
git submodule add https://github.com/<your-org>/earthity-design-system earthity-design-system
git submodule update --init
```

The folder name `earthity-design-system` matches the repo. The system lives at the consumer project root rather than under `vendor/` — there is one design system, no need to group with other third-party code.

---

## 2. Link the stylesheets

In every HTML page that uses the system, link both files **in this order**:

```html
<link rel="stylesheet" href="/earthity-design-system/earthity-tokens.css">
<link rel="stylesheet" href="/earthity-design-system/earthity-components.css">
```

- `earthity-tokens.css` defines CSS custom properties and utility classes (`.t-*`, `.c-*`, `.bg-*`, `.p-*`).
- `earthity-components.css` defines `eu-*` component classes that consume those tokens.

Tokens load first because components depend on them.

If you serve from a different path (e.g., GitHub Pages), adjust the URL accordingly.

---

## 3. Class-name conventions

The system uses two complementary conventions:

### Utility classes — short prefixes
Defined in `earthity-tokens.css`. Stable, project-flavored. **No `eu-` prefix.**

| Prefix | Domain | Example |
|---|---|---|
| `.t-*` | type styles | `t-display-01`, `t-body-short-01`, `t-caption-01`, `t-code-01` |
| `.c-*` | text colors | `c-primary`, `c-secondary`, `c-blue`, `c-inverse` |
| `.bg-*` | backgrounds | `bg-base`, `bg-layer` |
| `.p-*` / `.mt-*` / `.mb-*` | spacing | `p-05` (16px), `mt-06` (24px) |

### Component classes — `eu-` prefix, BEM-ish
Defined in `earthity-components.css`.

| Pattern | Form | Example |
|---|---|---|
| Block | `.eu-{name}` | `.eu-btn`, `.eu-tag`, `.eu-tile` |
| Element | `.eu-{name}__{part}` | `.eu-btn__icon`, `.eu-tile__title` |
| Modifier | `.eu-{name}--{variant}` | `.eu-btn--primary`, `.eu-tag--success` |
| State | attribute selectors | `[aria-current="page"]`, `[aria-selected="true"]`, `[disabled]`, `[aria-busy="true"]` |

The `eu-` prefix prevents collisions with consumer-defined classes. Use it for every component class.

### Why two conventions
Utilities are short because they appear inline and frequently. Components are prefixed because they're public-API surfaces with state behavior; collisions there would be costly. This split is intentional, not an oversight.

---

## 4. Use the components

Each preview file in `earthity-design-system/preview/` is a working example of one component or token group. To use a component, copy the markup pattern from the relevant preview page and adapt the content.

```html
<!-- A primary button (asymmetric padding, reserves space for trailing icon) -->
<button class="eu-btn eu-btn--primary">Get started</button>

<!-- A button without a trailing icon — symmetric padding -->
<button class="eu-btn eu-btn--primary eu-btn--symmetric">Get started</button>

<!-- A status tag -->
<span class="eu-tag eu-tag--success">Active</span>

<!-- A clickable filter chip -->
<button class="eu-tag eu-tag--interactive eu-tag--selected">Active corridors</button>

<!-- A native modal -->
<dialog class="eu-modal" id="confirm">…</dialog>
<button class="eu-btn eu-btn--primary" onclick="document.getElementById('confirm').showModal()">Open</button>
```

The full component catalog is browsable from `earthity-design-system/index.html`.

---

## 5. Override safely

Two layers of override are supported. Use the right one:

**Theme overrides — use `[data-theme="dark"]` or `.theme-dark`** to flip the entire system to dark mode. The token sheet defines the dark overrides; activate by adding the attribute or class to a parent (commonly `<html>` or `<body>`).

```html
<html data-theme="dark">
  <body><!-- system uses dark tokens throughout --></body>
</html>
```

**Per-instance overrides — override component classes via specificity** (or with `!important` only when truly necessary):

```css
/* Product-specific tweak: corridor-row tags are slightly larger */
.corridor-row .eu-tag { padding: 4px 12px; }
```

**Do not** override the token CSS variables themselves except via `[data-theme]`. Doing so anywhere else creates invisible drift — a token variable is a contract.

**Do not** redefine `.eu-*` classes in your own stylesheet. If you need a new variant, request it upstream so every consumer benefits.

---

## 5b. Consumer CSS conventions (what to use in your own stylesheet)

§5 covers what to leave alone. This covers what to reach for when you write your project's own CSS — `site.css`, `app.css`, whatever.

### Naming — BEM-ish, no `eu-` prefix

Mirror the system's pattern but drop the prefix (the `eu-` namespace is reserved):

| Pattern | Form | Example |
|---|---|---|
| Block | `.section-name` | `.hero`, `.footer`, `.product-grid` |
| Element | `.section-name__part` | `.hero__copy`, `.footer__grid`, `.product-grid__tile` |
| Modifier | `.section-name__part--variant` | `.product-grid__tile--featured`, `.footer__heading--inv` |

**No utility classes** in consumer code. Use semantic component names. For one-off type/color/spacing needs, use the system utilities (`.t-*`, `.c-*`, `.bg-*`, `.p-*`, `.mt-*`, `.mb-*`) defined in `earthity-tokens.css` — don't invent your own.

### No inline `style=` attributes

Every consumer style belongs in a stylesheet. Inline `style=` forces CSP `style-src 'unsafe-inline'`, defeating the strict CSP posture this system supports (see §5 token rules and the CSP note at the top of `earthity-tokens.css`).

### Promote shared patterns upstream

If a pattern repeats across pages or projects, propose it as an `eu-` component rather than re-implementing in every consumer. That keeps consumer CSS small and the system's surface coherent.

Existing patterns that are easy to miss and shouldn't be re-invented:
- **Section headers** (the blue accent bar above a section title, with optional dim continuation text) — use `.eu-section-header` (see §9 quick reference). Don't roll your own `.section-bar` / `.section-heading` in consumer CSS.

### Migration foot-gun: `@media [style*="..."]` selectors

If you're cleaning up an existing codebase and extracting inline styles into classes, **first** grep your stylesheet for media-query rules that target inline-style attribute selectors:

```bash
grep -rn '\[style\*=' your-css-dir/
```

These rules silently break when the inline style they target is removed — responsive layout snaps back to desktop on smaller viewports with no error visible. Rewrite each `[style*="..."]` selector to use the new component class you're introducing **in the same commit** as the inline-style removal.

### Full CSP migration journey

Beyond the conventions above, `playbooks/csp-rollout.md` is a complete 5-phase runbook for taking a consumer site from "no CSP" to "enforced strict CSP" — covers inline-pattern migration, static security headers, CSP report-only, hash recipes, accepted exceptions, and the long-tail monitoring window. Read it once before starting; reference per-phase as you execute.

---

## 6. Behavioral components — what the system does and doesn't ship

The system ships **visual styling** for every component. For interactive behavior, the rule is:

- **Native HTML where viable.** Use `<dialog>` for modals, `<details>` for disclosures, `<input list>` for simple comboboxes, `popover` attribute for tooltips/popovers. The system styles these natively-handled primitives via `.eu-modal`, `.eu-details`, `.eu-popover`.
- **Spec-only for the rest.** Tabs, rich dropdowns, and custom comboboxes have no native primitive that meets accessibility expectations. The system specifies behavior in `preview/components-overlays.html` (states, ARIA, keyboard map) but does not ship JS. Wire the behavior using your framework's primitive (Radix, Headless UI, Mantine, native framework component). Use the system's visual classes — do not invent new ones.

---

## 7. Update the design system

When the design system publishes a new version:

```bash
cd earthity-design-system
git pull origin main
cd ..
git add earthity-design-system
git commit -m "Update earthity design system"
```

To pin to a specific tagged release:

```bash
cd earthity-design-system
git checkout v1.2.0
cd ..
git add earthity-design-system
git commit -m "Pin earthity design system v1.2.0"
```

Pinning is the standard pattern for production. Floating on `main` is fine for prototypes and internal tools where fast iteration matters more than version stability.

---

## 8. When working with Claude Code in a consumer project

If you have a `CLAUDE.md` in your consumer project root, add a one-liner pointing here:

```markdown
## Design system
This project uses the earthity design system, vendored as a submodule at
`earthity-design-system/`. See `earthity-design-system/CLAUDE.md` for
linking, class conventions, and override rules.
```

That gives Claude awareness of the system from cold start (before it touches any file inside the submodule).

---

## 9. Quick reference

| Task | Pattern |
|---|---|
| Add a button | `<button class="eu-btn eu-btn--primary">…</button>` |
| Add a status pill | `<span class="eu-tag eu-tag--success">Active</span>` |
| Add a tile | `<a class="eu-tile eu-tile--interactive">…</a>` |
| Add a stat | `<div class="eu-stat"><span class="eu-stat__label">…</span><span class="eu-stat__value">…</span></div>` |
| Add a section header (bar + title + optional dim continuation) | `<header class="eu-section-header"><div class="eu-section-header__bar"></div><h2 class="eu-section-header__title">Lead phrase. <span class="eu-section-header__secondary">Continuation copy.</span></h2></header>` (add `eu-section-header--inv` for a single dark section on a light page) |
| Mark active nav | `aria-current="page"` (no extra class needed) |
| Mark selected tab | `aria-selected="true"` (no extra class needed) |
| Mark loading button | `aria-busy="true"` + add `<span class="eu-btn__spinner">` |
| Mark disabled | native `disabled` or `aria-disabled="true"` |
| Apply dark theme | `<html data-theme="dark">` |
| Use a Carbon icon | inline SVG with `<use href="#i-{name}">` and `class="eu-icon eu-icon--24"` (paths in `assets/icons/README.md`) |

---

## 10. Reference files

- `index.html` — visual home page; click through every preview.
- `README.md` — the full visual brief (color, type, spacing, motion, voice).
- `AUDIT.md` — Phase 0 audit; historical record of how the system was structured.
- `assets/icons/README.md` — Carbon icon subset with consumption paths.
- `examples/outpost/index.html` — worked example; the canonical product UI built entirely on the system.
- `playbooks/csp-rollout.md` — 5-phase runbook for taking a consumer site from "no CSP" to "enforced strict CSP." Generic; consumer keeps a session-specific runbook for actual commits.
