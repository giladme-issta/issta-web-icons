# Issta Web Icons — Browser

**Live:** https://giladme-issta.github.io/issta-web-icons/

A single-file HTML page that renders every icon in Issta's icon fonts, with search, filtering and one-click copy of the class name or a ready-to-paste HTML snippet.

## Icon sets

The page covers all five of Issta's icon fonts, switchable from the toggle in the header.

| Set | Class prefix | Font family | Icons | Source |
|---|---|---|---|---|
| `issta-web-icons` | `icon-` | `icomoon` | 364 (305 single, 59 multi) | [style.css](https://cdn.issta.co.il/icons/issta-web-icons/style.css) |
| `issta-cms-icons` | `icon-` | `icomoon` | 198 (197 single, 1 multi) | [style.css](https://cdn.issta.co.il/icons/issta-cms-icons/style.css) |
| `issta-icons` | `ist-icon-` | `issta-icomoon` | 41 (39 single, 2 multi) | [style.css](https://cdn.issta.co.il/icons/issta-icons/style.css) |
| `issta-ng-icons` | `ng-icon-` | `ng-icomoon` | 23 | [style.css](https://cdn.issta.co.il/icons/issta-ng-icons/style.css) |
| `bynd-icons` | `icon-f-` | `f-icomoon` | 9 | [style.css](https://cdn.issta.co.il/icons/bynd-icons/style.css) |

Switching sets resets the grid but keeps your size, color-mode and theme settings. The quick-search chips change to match the families in the selected set, and are hidden for the two small sets where scrolling is faster than filtering.

### Collisions — why only one stylesheet is loaded at a time

The page keeps a single `<link id="icon-css">` and swaps its `href` when you switch sets. This is not an optimisation, it's a correctness requirement:

- **`issta-web-icons` and `issta-cms-icons` share both the `icon-` prefix and the `icomoon` font-family name.** They define overlapping class names with *different* codepoints — `.icon-hotel` is `\e9b8` in one and `\e9b0` in the other, `.icon-man` is `\e965` in cms but `.icon-man1` in web. Load both and the later stylesheet silently wins, so icons render as the wrong glyph rather than failing visibly.
- **`bynd-icons` uses `icon-f-`, a subset of `icon-`.** Its own selector `[class^=icon-f-]` and the others' `[class^=icon-]` both carry `!important` at the same specificity, so cascade order alone decides which font-family applies to `icon-f-*`.

The two sets with a `note` field in `SETS` surface this as a banner under the switcher, since it's a real trap for anyone consuming these fonts in a page that pulls in more than one.

## Running it

There is no build step and no dependencies. Open `issta-icons.html` in a browser.

```
open issta-icons.html          # macOS
start issta-icons.html         # Windows
```

It works from `file://`, from a local static server, or dropped onto any internal host. The only external requests are the icon font CSS from the Issta CDN and two Google Fonts (Heebo, IBM Plex Mono). Without network access the layout still works — the glyphs just won't render, and the page shows a warning explaining that.

---

## Features

| Feature | Notes |
|---|---|
| Set switcher | Toggle in the header between all five sets; swaps the active stylesheet. |
| Live search | Matches the class name **and** the character code (`e9b8`). Multiple space-separated terms are AND-ed. |
| Type filter | All / single-color / multi-color. |
| Sort | File order (as authored in the CSS), alphabetical, or by character code. |
| Size slider | 20–88px, applied via a CSS variable on the grid. |
| Color mode | Original colors (as defined in the CSS) or flat single color. |
| Theme | Light / dark background. |
| Copy | Click a card → copies the class name. Click `</>` → copies a full HTML snippet. |
| Keyboard | `/` focuses search, `Esc` clears it, cards are tab-navigable. |

### Quick chips

Preset searches for the largest families in the selected set.

- `issta-web-icons`: `hotel`, `flight`, `car`, `arrow`, `rate`, `chat`, `basis`, `policy`, `se-`
- `issta-cms-icons`: `hotel`, `flight`, `car`, `arrow`, `empty`, `icon`, `lug`, `phone`
- `issta-icons`: `baggage`, `checkout`, `flight`, `calendar`, `property`, `arrow`, `pencil`
- `issta-ng-icons`, `bynd-icons`: none — the row is hidden

---

## Single-color vs multi-color icons

This distinction matters when you paste an icon into a real component, and it's why the `</>` button exists.

**Single-color** icons are one glyph and one CSS rule:

```css
.icon-hotel:before { content: "\e9b8"; }
```

```html
<i class="icon-hotel"></i>
```

**Multi-color** icons are several glyphs stacked on top of each other, each in its own color, pulled back into place with a negative `margin-left`:

```css
.icon-Hotel .path1:before { content: "\eb32"; color: #fff; }
.icon-Hotel .path2:before { content: "\eb33"; margin-left: -.794921875em; color: #ffc09c; }
/* …through .path11 */
```

They only render correctly if every `path` span is present, in order:

```html
<span class="icon-Hotel">
  <span class="path1"></span>
  <span class="path2"></span>
  <!-- … -->
  <span class="path11"></span>
</span>
```

Cards for these icons carry an `N paths` badge, and the `</>` button emits the complete markup.

---

## How the icon list is built

The page does not hardcode a hand-maintained list. It parses the CSS with a regex built from the set's prefix:

```js
new RegExp("\\.(" + prefix + "[A-Za-z0-9_\\-]+)(?:\\s+\\.path(\\d+))?:before\\s*\\{([^}]*)\\}", "g")
```

Each match yields the full class name, an optional path index, and the `content` codepoint. Matches are grouped into `{ cls, paths[], code, order }`.

Each set is described by one object in `SETS`:

```js
web:  { url, prefix: "icon-",     srcId: "src-css-web",  font: "icomoon",       chips: [...], note }
cms:  { url, prefix: "icon-",     srcId: "src-css-cms",  font: "icomoon",       chips: [...], note }
ist:  { url, prefix: "ist-icon-", srcId: "src-css-ist",  font: "issta-icomoon", chips: [...] }
ng:   { url, prefix: "ng-icon-",  srcId: "src-css-ng",   font: "ng-icomoon",    chips: [] }
bynd: { url, prefix: "icon-f-",   srcId: "src-css-bynd", font: "f-icomoon",     chips: [], note }
```

`note` is optional — when present it renders as a banner under the switcher.

Two sources feed the parser:

1. **Embedded copy** — each set's CSS is inlined in its own `<script type="text/plain">` block (`src-css-web`, `src-css-cms`, `src-css-ist`, `src-css-ng`, `src-css-bynd`) and parsed on demand. This is what makes the page work offline and from `file://`.
2. **Live refresh** — on each set load the page also `fetch`es that set's CDN URL. If it succeeds and the icon count differs, the grid re-renders from the fresh CSS. If CORS or the network blocks it, the failure is swallowed and the embedded copy stands. A late-arriving response is discarded if the user has already switched sets.

### Updating after the font changes

When new icons ship, either just reload the page on a host where the `fetch` succeeds, or refresh the embedded copy so the offline path stays current:

1. Download the set's `style.css` from the CDN.
2. Replace everything between the matching `<script type="text/plain" id="src-css-…">` block and its closing tag with the new file contents.

No other change is needed — the counts, badges and snippets all derive from the parse.

### Adding another set

Add an entry to `SETS`, add a `<script type="text/plain">` block with its CSS, and add a `<button class="set" data-set="…">` to the header. No new `<link>` is needed — the single one is swapped by `loadSet`. Nothing else in the code is prefix-aware.

---

## Two things worth knowing if you edit this

**RTL.** The page is `dir="rtl"`. The icon element itself is explicitly forced back to LTR:

```css
.ico { direction: ltr; unicode-bidi: isolate; }
```

Without this, the negative-`margin-left` stacking used by multi-color icomoon icons falls apart inside an RTL context and the glyphs scatter.

**The white icons.** Several icons are authored in white — `icon-eye`, `icon-hotel-map`, `icon-f-secure`/`faq`/`like`, the `path1` layer of many multi-color icons, and both `ist-icon-arrow-slider-*`. They are invisible on a light background, which is the reason for the dark theme toggle rather than it being decoration.

---

## File layout

```
issta-icons.html    # everything: markup, styles, 5 embedded CSS copies, app script
README.md
```

The palette (`#485D77` slate, `#FFC09C` apricot, `#0DB14B` green) is taken from the colors declared inside the icon CSS itself, so the page reads as part of the same system as the icons it displays.
