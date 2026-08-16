# Issta Web Icons — Browser

**Live:** https://giladme-issta.github.io/issta-web-icons/

A single-file HTML page that renders every icon in Issta's icon fonts, with search, filtering and one-click copy of the class name or a ready-to-paste HTML snippet.

## Icon sets

The page covers both of Issta's icon fonts, switchable from the toggle in the header. They don't collide — different class prefix, different font family — so both stylesheets are loaded at once and only the grid changes.

| Set | Class prefix | Font family | Icons | Source |
|---|---|---|---|---|
| `issta-web-icons` | `icon-` | `icomoon` | 364 (305 single, 59 multi) | [style.css](https://cdn.issta.co.il/icons/issta-web-icons/style.css) |
| `issta-icons` | `ist-icon-` | `issta-icomoon` | 41 (39 single, 2 multi) | [style.css](https://cdn.issta.co.il/icons/issta-icons/style.css) |

Switching sets resets the grid but keeps your size, color-mode and theme settings. The quick-search chips change to match the families that actually exist in the selected set.

---

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
| Set switcher | Toggle in the header between `issta-web-icons` and `issta-icons`. |
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
- `issta-icons`: `baggage`, `checkout`, `flight`, `calendar`, `property`, `arrow`, `pencil`

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
web: { url, prefix: "icon-",     srcId: "src-css-web", font: "icomoon",       chips: [...] }
ist: { url, prefix: "ist-icon-", srcId: "src-css-ist", font: "issta-icomoon", chips: [...] }
```

Two sources feed the parser:

1. **Embedded copy** — each set's CSS is inlined in its own `<script type="text/plain">` block (`src-css-web`, `src-css-ist`) and parsed on demand. This is what makes the page work offline and from `file://`.
2. **Live refresh** — on each set load the page also `fetch`es that set's CDN URL. If it succeeds and the icon count differs, the grid re-renders from the fresh CSS. If CORS or the network blocks it, the failure is swallowed and the embedded copy stands. A late-arriving response is discarded if the user has already switched sets.

### Updating after the font changes

When new icons ship, either just reload the page on a host where the `fetch` succeeds, or refresh the embedded copy so the offline path stays current:

1. Download the set's `style.css` from the CDN.
2. Replace everything between the matching `<script type="text/plain" id="src-css-web">` (or `src-css-ist`) and its closing tag with the new file contents.

No other change is needed — the counts, badges and snippets all derive from the parse.

### Adding a third set

Add an entry to `SETS`, add a `<link>` for its stylesheet, add a `<script type="text/plain">` block with its CSS, and add a `<button class="set" data-set="…">` to the header. Nothing else is prefix-aware.

---

## Two things worth knowing if you edit this

**RTL.** The page is `dir="rtl"`. The icon element itself is explicitly forced back to LTR:

```css
.ico { direction: ltr; unicode-bidi: isolate; }
```

Without this, the negative-`margin-left` stacking used by multi-color icomoon icons falls apart inside an RTL context and the glyphs scatter.

**The white icons.** Several icons are authored in white — `icon-eye`, `icon-hotel-map`, the `path1` layer of many multi-color icons, and both `ist-icon-arrow-slider-*`. They are invisible on a light background, which is the reason for the dark theme toggle rather than it being decoration.

---

## File layout

```
issta-icons.html    # everything: markup, styles, embedded CSS copies, app script
README.md
```

The palette (`#485D77` slate, `#FFC09C` apricot, `#0DB14B` green) is taken from the colors declared inside the icon CSS itself, so the page reads as part of the same system as the icons it displays.
