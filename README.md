# Minimal CSS Framework

A small, token-driven CSS layer for any WordPress or general front-end
project. Builder-agnostic — no dependency on a specific page builder or
theme system. Four files, load in this order:

1. `tokens.css` — custom properties: palettes, spacing/type scale, radius,
   shadows, motion, z-index, containers, semantic aliases.
2. `base.css` — reset + accessibility foundations (focus rings, reduced
   motion, `.visually-hidden`, `.skip-link`).
3. `utilities.css` — a deliberately small utility layer.
4. `components.css` — example BEM components (buttons, cards, badges,
   form fields). Reference implementations — edit or delete freely.

## Loading it into your project

No build step required — paste the contents (or `@import` the files) into
your builder's global/custom CSS panel if it has one, or enqueue them as a
stylesheet from your theme's `functions.php`:

```php
add_action( 'wp_enqueue_scripts', function () {
    wp_enqueue_style( 'minimal-tokens', get_stylesheet_directory_uri() . '/css/tokens.css', [], '1.0' );
    wp_enqueue_style( 'minimal-base', get_stylesheet_directory_uri() . '/css/base.css', [ 'minimal-tokens' ], '1.0' );
    wp_enqueue_style( 'minimal-utilities', get_stylesheet_directory_uri() . '/css/utilities.css', [ 'minimal-base' ], '1.0' );
} );
```

Add `theme-dark` / `theme-light` to `<html>` (e.g. via a small inline script
reading `prefers-color-scheme`, or a theme toggle) to switch palettes.

## Design decisions

- **1rem = 10px, not 16px.** Done via `html { font-size: 62.5%; }` in
  `base.css` rather than a hardcoded `font-size: 10px`. 62.5% of a 16px
  browser default is 10px, but unlike a fixed px value, this still scales
  correctly for a visitor who has changed their browser's default text
  size — hard-coding `10px` would silently override that preference and is
  a genuine accessibility regression, so it's avoided here.
- **Every rem-based token is scaled for that 10px root.** Spacing, type
  scale, and radius are all expressed in rem, sized so their rendered
  pixel values line up with the numbers you'd expect — i.e. `--space-m`
  reads as "16px-ish," not "10px-ish," even though 1rem is 10px.
  `--radius-full` is the one exception: it's set in `px` (`9999px`) since
  "fully round" isn't a type-scale-relative concept.
- **Both palettes have complete dark-mode ramps.**
  - `p1` (brand): each color's dark variant reduces lightness by a
    consistent ratio, so the palette stays internally coherent rather than
    each dark color being picked by eye. Two extra tones were added —
    `p1-c7` (brand ink, for text) and `p1-c8` (brand tint, for subtle
    backgrounds) — both derived from the same brand hue.
  - `n1` (neutral): the dark ramp **reverses** the light one (`c1`↔`c10`,
    `c2`↔`c9`, …) rather than inventing new grays. That way a token keeps
    its *role* across themes — whatever you used `n1-c9` for as a light
    background stays usable as a background in dark mode too, because the
    variable itself now resolves dark. Sanity-check contrast if you lean on
    this heavily; a mirrored ramp is a good starting point, not a
    guarantee.
- **A `theme-inverted` / `theme-always-light` / `theme-always-dark` pattern**
  lets you flip an individual subtree to the opposite theme, or pin it to
  one theme regardless of context — handy for e.g. a permanently-dark
  "featured" panel on an otherwise light page.
- **Extra token groups**, since a minimal framework for real projects needs
  a few things beyond raw color/spacing/type: font stacks/weights/
  line-heights, motion (duration + easing), elevation (shadows — dark mode
  swaps to hairline outlines since dark-on-dark shadows barely read),
  z-index (named steps instead of magic numbers), container widths, border
  tokens, and a small independent status palette (`s1-success/warning/
  danger/info`) since those shouldn't be borrowed from brand colors.
- **Semantic aliases** (`--color-text`, `--color-bg`, `--color-primary`,
  etc.) sit on top of the raw palettes. Components should reach for these,
  not `--p1-c1` / `--n1-c3` directly — it keeps re-theming a one-file edit
  and matches how the `theme-inverted` / `theme-always-*` hooks are meant
  to be used.

## Conventions

- **BEM** for anything reusable: `.block`, `.block__element`,
  `.block--modifier`. See `components.css` for the pattern in practice.
  Utilities (`utilities.css`) are the deliberate exception — flat,
  single-purpose classes (`u-` prefix) are appropriate there since they're
  not "components."
- **Low specificity by default.** `base.css` wraps every reset rule in
  `:where()`, so it always loses to a real class without needing
  `!important` or selector-weight games — useful if you're layering this
  under a builder or theme that generates its own classes.
- **Logical properties** (`margin-block`, `padding-inline`, etc.) are used
  instead of physical ones (`margin-top`, `padding-left`) so the framework
  doesn't need a separate RTL pass.
- **Semantic HTML state over class-only state.** e.g. `.btn:disabled` /
  `[aria-disabled]`, `.field--invalid` paired with `aria-invalid="true"` on
  the actual input — a visual-only "disabled" or "invalid" class is
  invisible to assistive tech.

## Utilities are intentionally small

If you're building with a visual page builder, it likely already handles
spacing, layout, and typography visually for anything placed by hand.
`utilities.css` only adds what that kind of panel typically doesn't cover
well: vertical rhythm across dynamic content you can't select block-by-block
(`.flow`, inside a loop or rich-text output), and two layout primitives
(`.u-stack`, `.u-cluster`) for quick flex recipes. Resist the urge to grow
this into a full utility-first system — that fights hand-authored or
builder-generated markup more than it helps it.

## Accessibility checklist this framework sets up for you

- Visible focus ring on keyboard navigation only (`:focus-visible`), using
  `--focus-ring-color` / `-width` / `-offset`.
- `prefers-reduced-motion` respected globally.
- `.visually-hidden` + `.skip-link` for screen-reader/keyboard-only content.
- Fluid type never drops below a readable floor at any viewport width.
- Everything above is a floor, not a ceiling — run an actual audit
  (axe, Lighthouse, or a screen reader pass) before shipping.

## Browser support

`clamp()`, `:focus-visible`, `:where()`, logical properties, and
`aspect-ratio` are all Baseline-widely-available. `text-wrap: balance` /
`pretty` degrade harmlessly (normal wrapping) in browsers that don't
support them yet, so no fallback is needed.
