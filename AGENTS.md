# AGENTS.md

Static website (Microverse HTML & CSS capstone): "Home sweet home", a fake real-estate agency.
No JavaScript, no build system, no package.json. Pure HTML + Bootstrap 4.5 (CDN) + SCSS compiled to CSS.

- Live: https://vichuge.github.io/8-capstone-project/ (GitHub Pages, served from `main`, repo root)
- Repo: https://github.com/vichuge/8-capstone-project

## Layout

```
index.html    Home: hero search form -> list.html, "4 steps", 3 colored promo rows
list.html     Filter form (Colony/State/City/Type/Rooms/Garage) + 4 house cards -> house.html
house.html    House detail grid, contact line, description, Google Maps iframe
assets/img/                 Images (screenshot*.png are only used by README)
assets/stylesheets/sass/    SOURCE of styles (edit here)
  _variables.scss           Colors ($magenta #d35649, $blue, $yellow, grays) and fonts
  _framework.scss           Utility classes: .bg-*, .text-*, .font-size-mid, .text-f2, .no, global h2/h3
  _media.scss               Breakpoints (min-width 768px, 1024px)
  style.scss                Imports the partials + page/section styles
assets/stylesheets/css/style.css   COMPILED output, committed (Pages serves it). Never edit by hand.
.github/workflows/linters.yml      CI: Lighthouse, Webhint, Stylelint on PRs and pushes to main
.stylelintrc.json / .hintrc        Microverse linter configs (do not loosen them)
```

## Shared page structure

All 3 pages repeat the same `<head>`, `<header><nav>` and `<footer>` verbatim (no templating).
**A change to the nav, footer or head must be applied to all three files.**

`<head>` order: meta charset, viewport, description -> Bootstrap 4.5.3 CSS (jsDelivr, SRI) -> `<title>` ->
Google Fonts (Castoro, Noto Sans) -> Font Awesome 5.15.4 CSS (cdnjs, SRI) -> `assets/stylesheets/css/style.css`.

Body: `<body class="bg-white container-fluid no">` > `<header>` (nav) > `<main>` (page content) > `<footer>`.
There are no `<script>` tags; Bootstrap JS/jQuery are not needed (no collapses, modals, etc.).

## Conventions

- Layout with Bootstrap 4 grid/utilities (`row`, `col-*`, `mr-2`, `pl-md-0`, `font-weight-bold`...). Bootstrap 4, not 5.
- `.row` margins/paddings are forced to 0 in `_framework.scss`; spacing is done with utility classes.
- Icons: Font Awesome 5 classes (`fas`, `far`, `fab`). Always add `aria-hidden="true"` to `<i>` icons.
  Links/buttons that show only an icon need `aria-label`.
- No inline `style="..."` (Webhint `no-inline-styles` is an error). Put styles in SCSS.
- Every form control needs a `<label for>` + `id`, or an `aria-label`.
- Every `<button>` needs an explicit `type`. Iframes need `title`. Images need meaningful `alt`.
- Keep heading order (h1 -> h2 -> h3) without skipping levels.
- Escape `&` as `&amp;` inside attribute URLs.
- HTML formatting is Prettier-style (2 spaces, wrapped attributes, `/>` on void elements).
- Content is in English; placeholder text is Lorem ipsum.

## Editing styles

1. Edit files in `assets/stylesheets/sass/` (use `//` comments in partials when they should not reach the CSS).
2. Recompile and normalize the CSS (Dart Sass output needs `--fix` to satisfy Stylelint blank-line rules):

```bash
npx sass --no-source-map --style=expanded assets/stylesheets/sass/style.scss assets/stylesheets/css/style.css
npx stylelint assets/stylesheets/css/style.css --fix
```

Commit both the `.scss` and the compiled `style.css`. No source maps, no `.sass-cache` (both gitignored).

## Linters (must pass before committing)

Install in a throwaway way (do not commit package.json / node_modules; they are gitignored):

```bash
npm install --no-save stylelint@13.3.x stylelint-scss@3.17.x stylelint-config-standard@20.0.x stylelint-csstree-validator@1.x hint@7.x
npx stylelint "**/*.{css,scss}"
npx hint --telemetry=off .
```

Notes:
- `hint@6` crashes on modern Node; use `hint@7.x`.
- Webhint's `html-checker` calls validator.w3.org; offline it only warns. Alternative check:
  `curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @index.html "https://validator.w3.org/nu/?out=gnu"`
  (only `info` lines about trailing slashes are expected).
- Lighthouse in CI only collects a report (no assertions).

## Known leftovers (intentional / not yet done)

- Many links are `href="#"` placeholders (About, Blog, More, social networks, subscribe form `action="#"`).
- Forms do not filter anything; `list.html` cards are 4 identical "Demothree" houses.
- White text on `.bg-gray-light2` (#98a4a5) and `.bg-blue` has low contrast; colors come from the original design.
