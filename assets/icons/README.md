# earthity icons — Carbon subset

The earthity system uses the IBM Carbon icon set (line, 2px stroke, 32×32 grid, monochrome). Icons are referenced from the canonical Carbon source rather than redistributed in this repo.

## Source

Carbon icons live at:
```
https://github.com/carbon-design-system/carbon/tree/main/packages/icons/src/svg/32
```

Each icon has size variants at `…/svg/{16,20,24,32}/{name}.svg`. The 32px asset is the canonical drawing; smaller sizes are pre-optimized.

Raw SVG URL pattern:
```
https://raw.githubusercontent.com/carbon-design-system/carbon/main/packages/icons/src/svg/{size}/{name}.svg
```

## Subset used by earthity

| name              | use                                          |
|-------------------|----------------------------------------------|
| `arrow--right`    | CTAs, card hover affordance, nav forward     |
| `network--3`      | Outpost network visualization                |
| `drone`           | Drone operations, fleet                      |
| `location`        | Host site markers, geo data                  |
| `settings`        | Configuration, preferences                   |
| `user--multiple`  | Operator / team / org switcher               |
| `checkmark`       | Success, completion, verified                |
| `warning`         | Alerts, restricted, attention required       |

## Consumption

Three valid paths for consumers, in order of preference:

**1. npm (recommended for app code)**
```sh
npm install @carbon/icons-react
```
Import individual icons by name. Carbon ships size and color props. earthity does not re-export these.

**2. Vendored copy (recommended for static prototypes)**
Copy the eight SVGs above into your repo at `earthity-design-system/assets/icons/{name}.svg`, then reference them directly:
```html
<img src="/earthity-design-system/assets/icons/arrow--right.svg" width="20" height="20" alt="">
```

**3. Inlined SVG (recommended for color theming)**
Paste the SVG markup inline and set `fill="currentColor"`. The icon then inherits text color via CSS. This is the only path that themes correctly without `mask-image`.

## Sizing

Per the README:
- 16px — inline / caption
- 20px — default UI
- 24px — UI feature
- 32px — illustrative

## Coloring

Icons are monochrome. Set color via the parent's `color` (when inlined with `fill="currentColor"`) or via `mask-image` + `background-color: currentColor` (when used as an `<img>` mask). Never embed a hex value inside the SVG.

## License

Carbon icons are Apache 2.0. Carbon's `LICENSE` file applies; this repo does not relicense.
