# earthity Design System

## Company Overview

**earthity** (always lowercase) is a company building infrastructure for the autonomous airspace economy. The name is stylized in lowercase at all times, set in Manrope.

### Products

#### Outpost — The Autonomous Airspace Network
The primary product. Outpost is a two-sided platform connecting two distinct user groups:

- **Hosts** — property owners or operators who open their physical locations as nodes in the airspace network (landing pads, charging stations, relay points).
- **Operators** — organizations that fly drones through the network. Operators include:
  - Drone service providers (DSPs)
  - IT providers running drone programs on behalf of enterprise clients
  - Enterprises with their own internal drone operations

Outpost is infrastructure. It is where autonomous drone traffic is coordinated, authorized, and managed across a shared network.

#### Drone Program Integration — End-to-End Custom Deployments
The secondary product. A professional services and platform offering for IT providers and enterprises that want a complete drone program built from scratch. This includes hardware selection, software integration, regulatory compliance, training, and ongoing management. Drone Program Integration deployments typically plug back into Outpost as the network backbone.

---

## Sources
No external codebase or Figma link was provided. This design system was built from the design brief alone, using IBM Carbon as the foundational visual language adapted for earthity's brand.

---

## CONTENT FUNDAMENTALS

### Tone & Voice
- **Clinical precision meets forward gravity.** earthity's copy reads like engineering documentation written by someone who genuinely believes in the mission. No hype, no adjective stacking.
- **Declarative, not promotional.** Sentences make statements. "Outpost connects hosts and operators." Not "Outpost is the revolutionary platform that transforms how drones..."
- **The future is assumed, not promised.** Copy doesn't say "soon" or "imagine." It describes what the network does as if it already operates at scale.

### Casing & Punctuation
- **earthity**: always lowercase, no exceptions — in headlines, at the start of sentences, everywhere.
- **Outpost**: Title case, treated as a proper noun.
- **Drone Program Integration**: Title case as a product name.
- Headlines: sentence case (only first word + proper nouns capitalized).
- No Oxford serial comma avoidance — use the Oxford comma.
- Em dashes (—) over en dashes for asides. No spaced hyphens.
- Minimal exclamation points. The work speaks for itself.

### Perspective
- External marketing: "you" for direct address, "we" for earthity.
- Operator docs: "your fleet," "your program," "your operations."
- Host docs: "your location," "your site."
- Avoid: "users," "customers," "clients" — prefer role-specific nouns (operators, hosts, providers).

### Terminology
- Airspace, not sky. Network, not platform (in operator contexts). Program, not project. Integration, not onboarding.
- Drone is acceptable; UAV or UAS acceptable in regulatory/technical contexts.
- Avoid: "revolutionize," "disrupt," "seamless," "cutting-edge."

### Emoji & Special Characters
- No emoji in product UI or documentation.
- Arrows (→, ←) used as directional indicators in UI copy.
- Registered trademark (®) used where legally required.

---

## VISUAL FOUNDATIONS

### Design Language
earthity's visual system inherits from IBM's Carbon Design System — a grid-disciplined, token-driven, radically rectangular design language. The system is adapted with Manrope as the brand typeface (replacing IBM Plex Sans for brand-level expressions) while retaining IBM Plex Sans for UI density and technical surfaces. The overall character is: **dark, precise, flat**.

### Color
- Two-value base: near-black (`#161616`) and white (`#ffffff`), with a full gray ramp.
- Single accent: **earthity Blue** = IBM Blue 60 (`#0f62fe`). This is the only chromatic hue in the core UI. All interactive elements use this one blue.
- Dark theme is the primary brand expression (dark nav, dark hero sections). Light theme is used for content-dense interior pages.
- No gradients. No secondary accent colors. No tinted backgrounds except Gray 10 (`#f4f4f4`) for surface layering.

### Typography
- **Manrope** (brand font): Used for earthity wordmark, display headlines on marketing surfaces, and brand-level expressions. Weight 300 at display; weight 600/700 for the wordmark.
- **IBM Plex Sans**: UI typography, body copy, navigation, forms, documentation. Weights: 300 (display), 400 (body), 600 (emphasis).
- **IBM Plex Mono**: Code, technical labels, data readouts, coordinates, flight IDs.
- Letter spacing: 0.16px at 14px; 0.32px at 12px. Zero at display sizes.

### Backgrounds & Surfaces
- Page background: `#ffffff` (light) or `#161616` (dark).
- Card/tile surface: `#f4f4f4` (Gray 10) on white; `#262626` (Gray 90) on dark.
- No shadows on cards — depth is achieved through background-color layering only.
- Floating elements (dropdowns, tooltips, modals) use `box-shadow: 0 2px 6px rgba(0,0,0,0.3)`.
- No gradients. Solid, flat surfaces only.

### Corner Radii
- **0px**: All primary UI — buttons, inputs, cards, tiles, panels. Carbon is rectangular.
- **24px (pill)**: Tags and labels only.
- **50%**: Avatars and circular icon containers.

### Borders
- Minimal. Gray 30 (`#c6c6c6`) for dividers and subtle separators.
- Inputs: bottom-border only (2px). No boxed inputs.
- Cards: no border by default. Background color provides separation.

### Spacing
- 8px base grid. All spacing is multiples of 8 (with 2px/4px micro-adjustments).
- Scale: 2, 4, 8, 12, 16, 24, 32, 40, 48, 64, 80, 96, 160px.
- Component internal padding: 16px default.
- Section vertical rhythm: 48px standard, 80–96px for hero sections.

### Animation & Hover States
- No bounces. No spring physics. earthity is not playful.
- Transitions: `ease-in` or `ease` at 70–100ms for color; 150–200ms for layout shifts.
- Hover on dark surfaces: text shifts white (`#c6c6c6` → `#ffffff`).
- Hover on cards: background shifts to Gray 10 Hover (`#e8e8e8`).
- Hover on buttons: blue darkens (`#0f62fe` → `#0353e9`).
- Press/active: further darkens (`#002d9c`).
- No transform scale on press. No opacity flicker.

### Imagery
- Color vibe: cool, desaturated, industrial. Aerial photography, infrastructure, industrial landscapes.
- No warm-toned lifestyle imagery.
- Technical diagrams in monochrome + single blue highlight.
- No hand-drawn illustrations.
- No stock-photography smiles.

### Icons
- IBM Carbon icon set (line icons, 20px/24px grid, 2px stroke).
- No filled icons in UI. Line weight is consistent.
- Icons are monochrome: inherit text color or explicit gray/white.
- No emoji as icons.

---

## VISUAL FOUNDATIONS — Quick Reference

| Property | Value |
|---|---|
| Border radius (UI) | 0px |
| Border radius (tags) | 24px |
| Shadow (cards) | none |
| Shadow (floating) | 0 2px 6px rgba(0,0,0,0.3) |
| Focus ring | 2px solid #0f62fe inset + 1px solid #fff |
| Input style | bottom-border only |
| Button height | 48px default |
| Nav height | 48px |
| Base spacing | 8px |

---

## ICONOGRAPHY

earthity uses the **IBM Carbon icon set** as its icon system. Carbon icons are line-style, drawn on a 20px/24px grid, 2px stroke weight, sharp corners, no fill.

- **Format**: SVG preferred; icon font (`ibm_icons`) also available in Carbon.
- **Sizes**: 16px (inline/caption), 20px (default), 24px (UI feature), 32px (illustrative).
- **Color**: Monochrome. Icons inherit text color or are set explicitly to Gray 70 (`#525252`) for secondary; white on dark surfaces.
- **No emoji as icons.** No Unicode characters as decorative icons.
- **CDN**: Carbon icons are available via `@carbon/icons` npm package. For HTML prototypes, inline SVGs are extracted from the Carbon icon library.
- Assets in `assets/icons/` contain key Carbon SVG icons used across Outpost and Drone Program Integration.

### Key icons used
- `arrow--right`: CTAs, navigation, card hover states
- `network--3`: Outpost network visualization
- `drone`: Drone operations
- `location`: Host site markers
- `settings`: Configuration
- `user--multiple`: Operator/team management
- `checkmark`: Success/completion
- `warning`: Alerts

---

## File Manifest

```
README.md                          — This file
CLAUDE.md                          — Consumer integration guide (submodule setup, eu- conventions)
AUDIT.md                           — Phase 0 audit (gaps + decisions, historical)
index.html                         — Design system home page (links every preview)
earthity-tokens.css                — Tokens: colors, type, spacing, radii, motion
earthity-components.css            — eu-* component layer (consumed alongside tokens)
assets/
  icons/                           — Carbon icon subset; see assets/icons/README.md
  wordmark-dark.svg                — earthity wordmark (dark on light)
  wordmark-white.svg               — earthity wordmark (white on dark)
preview/                           — Foundation + component preview pages
  colors-primary.html              — Primary + interactive color swatches
  colors-neutral.html              — Full gray scale + surface layering
  colors-semantic.html             — Status colors + eu- semantic tokens
  type-display.html                — Display + heading type specimens
  type-body.html                   — Body, caption, label, link states
  type-mono.html                   — Monospace specimens (light + dark)
  spacing-tokens.html              — Spacing scale + radius scale
  elevation.html                   — Surface layering, focus ring, input states
  components-buttons.html          — Button variants + states + loading
  components-inputs.html           — Text inputs, select, textarea, check, radio
  components-forms.html            — Toggle, search, file upload, multi-step
  components-cards.html            — Tile variants, interactive + focus, data row
  components-nav.html              — Primary nav, tabs, dark sidebar
  components-tags.html             — Tags, filter chips, removable, notifications
  components-icons.html            — Carbon icon subset, sizes, color inheritance
  components-overlays.html         — Native modal, details, popover; spec sheets
  components-feedback.html         — Empty, skeletons, errors, toasts
  components-data.html             — Stat blocks, mono-IDs, tables
examples/                          — Worked example — the canonical product consuming the system
  outpost/index.html               — Outpost app (hosts + operators + network map)
```
