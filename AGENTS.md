# AGENTS.md

Static website (Microverse HTML & CSS capstone): "Home sweet home", a fake real-estate agency in Mexico.
No JavaScript, no build system, no package.json. Pure HTML + Bootstrap 4.5 (CDN) + SCSS compiled to CSS.
Content is in English.

- Live: https://vichuge.github.io/8-capstone-project/ (GitHub Pages, served from `main`, repo root)
- Repo: https://github.com/vichuge/8-capstone-project

## Layout

```
index.html    Home: hero search (-> list.html), 4 steps, 3 featured house cards, 3 colored feature rows
list.html     "Houses for sale": filter form (Colony/State/City/Type/Rooms/Garage) + 4 house cards
house.html    "Casa Coral" detail: breadcrumb, photo, price, 8 facts, Apply/Call, description, agent contact, map
assets/img/
  bg-page.jpg (1600w) / bg-page-sm.jpg (800w)   Hero background (desktop / mobile)
  house.jpg (796x796)                            Detail photo
  house-card.jpg (600x400)                       Card thumbnail (all cards use it)
  maps.jpg, registered.jpg, transport.jpeg       Images of the 3 feature rows on index
  favicon.svg                                    Favicon
  screenshot*.png                                README only (mobile index, mobile house, desktop list)
  cellphone.png                                  Unused leftover from the original template
assets/stylesheets/sass/    SOURCE of styles (edit here)
  _variables.scss           Colors, fonts, breakpoints ($md 768px, $lg 992px)
  _framework.scss           Utilities: .bg-*, .text-*, global a/h2/h3, .btn-brand, .btn-outline-brand, .badge-sale, .skip-link
  style.scss                Components by page (header, hero, steps, features, cards, filters, house detail, footer)
  _media.scss               Breakpoint overrides; imported LAST in style.scss so it wins the cascade
assets/stylesheets/css/style.css   COMPILED output, committed (Pages serves it). Never edit by hand.
.github/workflows/linters.yml      CI: Lighthouse, Webhint, Stylelint on PRs and pushes to main
.stylelintrc.json / .hintrc        Microverse linter configs (do not loosen them)
```

## Shared page structure

All 3 pages repeat the same `<head>`, `<header>` and `<footer>` verbatim (no templating).
**A change to the head, nav or footer must be applied to all three files.**

- `<head>`: charset, viewport, description, theme-color, favicon, `<title>` (index also preloads the hero image),
  Bootstrap 4.5.3 CSS (jsDelivr, SRI), Google Fonts (Castoro + Noto Sans 400/700), Font Awesome 5.15.4 (cdnjs, SRI),
  then `assets/stylesheets/css/style.css`.
- `<body class="bg-white container-fluid no">` > `.skip-link` > `<header class="site-header">` > `<main id="main-content">` > `<footer class="site-footer">`.
- Nav items: Houses (`list.html`), About (`#about`), Contact (`#contact`); the anchors point to footer columns.
  The current page's link has `aria-current="page"` (Houses on list.html and house.html; none on index).
  Mobile: icon above a small label; desktop (>= 992px): icon + label in a row and the header is sticky.
- Footer: about text (`#about`), Explore links, Contact (`#contact`), Subscribe form, social icons (`.social-link`), copyright.
- No `<script>` tags; Bootstrap JS/jQuery are not needed.

## Layout gotcha: `.row` has no padding/margin

`_framework.scss` forces `.row { margin: 0; padding: 0 } !important`. Spacing utilities like `px-3` **on a `.row` do
nothing**; put them on the inner `.col-*` instead. Page sections use this pattern:

```html
<section class="row some-section">
  <div class="col-12 px-3 px-lg-5 py-5">...</div>
</section>
```

Card grids use `.house-grid` (flex with 10px gutters) with `<article class="col-...">` children, not `.row`.

## Reusable components

- House card: `article > .card.house-card > img.house-card-img + .card-body` with `.badge-sale`, `h3.house-card-title > a.stretched-link`,
  `.house-card-location`, `.house-card-price`, `ul.house-card-meta` (rooms/baths/garage, singular when the value is 1).
- Buttons: `.btn-brand` (solid magenta), `.btn-outline-brand`; feature rows use `.btn.btn-light.btn-feature.text-<color>`.
- Headings: `.section-title` (Castoro, home sections), `.page-title` (list h1), `.section-heading` (house h2s), `.footer-title`.
- House facts: `ul.house-facts > li.house-fact` with `i`, `.house-fact-label`, `.house-fact-value` (2-column grid).

## Colors (WCAG AA)

`$magenta #b5443a`, `$blue #2b6ea6` and `$gray-row #687475` pass 4.5:1 with white text and as text on white,
`#f5f5f5` and `#eee`. `$muted #5f6b6c` is for secondary text. Do not lighten them; Lighthouse accessibility is 100.
`$yellow` is defined but not used (white text on it would fail contrast).

## Conventions

- Bootstrap 4 classes (`mr-2`, `pl-lg-5`, `custom-select`, `sr-only`, `input-group-append`), not Bootstrap 5.
- Icons: Font Awesome 5 (`fas`, `far`, `fab`) and always `aria-hidden="true"`. Icon-only links need `aria-label`.
- No inline `style="..."` (Webhint error). Every form control has a `<label for>` (can be `.sr-only`).
- Every `<button>` has `type`. Images have `alt`, `width` and `height`; below-the-fold images use `loading="lazy"`.
- Heading order h1 -> h2 -> h3 without skipping levels. Each page has exactly one h1.
- Escape `&` as `&amp;` in attribute URLs.
- HTML is formatted with Prettier (`npx prettier --write *.html`).
- Images: resize/compress before adding them (e.g. with `sharp`); keep hero images under about 300 KB.

## Editing styles

```bash
npx sass --no-source-map --style=expanded assets/stylesheets/sass/style.scss assets/stylesheets/css/style.css
npx stylelint assets/stylesheets/css/style.css --fix   # Dart Sass output needs the blank lines added
```

Commit both the `.scss` and the compiled `style.css`. No source maps, no `.sass-cache` (both gitignored).
Stylelint `no-descending-specificity` is strict: a selector like `.x a` cannot come after `.y a:hover`.
Give elements their own class (see `.social-link`) or put shared link colors in one combined rule.

## Linters (must pass before committing)

Do not commit package.json / node_modules (gitignored):

```bash
npm install --no-save stylelint@13.3.x stylelint-scss@3.17.x stylelint-config-standard@20.0.x stylelint-csstree-validator@1.x hint@7.x
npx stylelint "**/*.{css,scss}"
npx hint --telemetry=off .
```

- `hint@6` crashes on modern Node; use `hint@7.x`.
- Webhint's `html-checker` calls validator.w3.org; offline it only warns. Alternative check:
  `curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @index.html "https://validator.w3.org/nu/?out=gnu"`
  (only `info` lines about trailing slashes are expected).
- Lighthouse in CI only collects a report (no assertions). Last local run (Nov 2026): accessibility, best practices
  and SEO 100 on all pages; performance 97-99.

## Known leftovers (intentional / not yet done)

- Forms do not filter anything (no JS/back end); all cards link to the same `house.html`.
- All cards use the same photo (`house-card.jpg`, actually an interior). Real photos per house would be an improvement.
- Social links and the subscribe form use `#`; phone numbers and emails are fake (`example.com`).
