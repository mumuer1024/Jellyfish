# Jellyfish Lite

> **[English](./README.md) · [简体中文](./README.zh-CN.md)**

> Jellyfish Lite is a **performance-lightened variant** of the [Jellyfish theme](https://github.com/n00bcodr/Jellyfish) by **n00bcodr** (DBAD Public License — see [`../LICENSE.md`](../LICENSE.md)). It **aims to preserve** functional behaviour; decorative motion and expensive rendering effects are trimmed to reduce rendering pressure. The upstream files in the repository root are **untouched** — this `lite/` directory is a standalone, drop-in replacement theme.

---

## 1. Overview

Jellyfish Lite aims to keep the original theme's functionality (layout, colours, fonts, interactions, plugin compatibility) while removing or narrowing the decorative effects that cost rendering performance:

- **removed** all `backdrop-filter` glass blurs → translucent solid backgrounds;
- **removed** persistent / infinite animations (including the full-screen background scroll) and entry/hover decoration animations;
- **removed** `will-change` layer hints and large glow shadows (≥20px);
- **narrowed** `transition: all` per occurrence — necessary feedback (menus, drawers, dialogs, fixed header) is kept;
- **added by default** (Round 2): detail-page button text (`icontext.css`) and coloured parental ratings (`ratings.css`, needs `scripts/rating.js`);
- **added** (Round 3): the Nico Remix logo mechanism (theme-coloured SVG mask, decoupled from colour schemes).

The verbose change list, static metrics and the internal acceptance records live in the audit document [`../AUDIT.md`](../AUDIT.md) (Chinese, internal). This guide focuses on using the theme.

> Note: “smoother” claims in this guide reflect manual A/B observation, not instrumented benchmarks — see [`../AUDIT.md`](../AUDIT.md) §7.5.

## 2. Installation

Add **one** import URL to Jellyfin’s **Dashboard → General → Custom CSS Code** (server-wide) or **user settings → Display → Custom CSS Code** (single user):

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
```

The `@main` reference is the **current recommended entry** — it is a *floating* reference that always tracks the latest `main` branch content (updated whenever the repository is pushed), **not a pinned/fixed version**. The `lite/` files are present and live on it.

> **CDN users** — this single `@import` is all you need. The main theme, default colour scheme, and bundled styles load through it. Every configuration below (colour switching, logo modes, optional components) is done from the same **Custom CSS Code** box by adding more lines — no repository access required.
>
> **Self-hosted users** — clone/copy the repository onto your own static server and import `lite/theme.css` from your own origin. Keep the directory structure intact: every internal `@import` (fonts, icons, login page, default colour scheme, icontext, ratings, logo remix) is **relative** to the entry file and resolves against the same origin. Do not copy `lite/theme.css` somewhere on its own; it needs `fonts/`, `colors/`, `logos/`, `lite/` around it.

> ⚠️ **Version pinning:** the repository already carries an old tag `1.0`, but it predates Jellyfish Lite and does **not** contain the `lite/` files — do **not** use `@1.0` (its `lite/*` paths 404). Until a Lite-specific tag is published, stay on `@main`.

The only external dependency is the Druidblack metadata-icon CDN, identical to the original theme (unchanged).

## 3. Colour schemes

All 15 original schemes are preserved untouched (`colors/*.css`, primarily `:root` CSS variables; a few also reference a background image URL):

`aurora` · `banana` · `coal` · `coral` · `forest` · `grass` · `jellyblue` · `jellyflix` · `jellypurple` · `lavender` · `midnight` · `mint` · `ocean` · `peach` · `watermelon`

- **Default**: `jellyblue` (imported in `lite/theme.css`).
- Switching a scheme only swaps CSS variables — **`lite/theme.css` is never re-imported**, so the performance optimisations always stay active.

### 3.1 Recommended: switch colours from Custom CSS (works for everyone)

Import `lite/theme.css` first, then add the target scheme **after** it — the later-imported `:root` variables win. This works in the **Dashboard → General → Custom CSS Code** box (server-wide) and in **user settings → Display → Custom CSS Code** (single user) alike, with no repository access needed:

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/colors/peach.css"); /* this server/device uses Peach */
```

(Self-hosted? Replace the URLs with your own static server paths.)

### 3.2 Self-hosted only: change the global default

If you self-host the repository you can instead change the scheme that is the global default by editing the colour `@import` line in `lite/theme.css`:

```css
@import url("../colors/ocean.css");   /* global default becomes Ocean */
```

### 3.3 Dynamic dropdown (script)

[`../scripts/theme_selector.js`](../scripts/theme_selector.js) adds a theme dropdown to the user’s profile settings (including a random-daily option). Notes:

- the script embeds **upstream** CDN links (`n00bcodr/Jellyfish`) — if you self-host a fork, update the URLs inside the script (`scripts/` are upstream files, untouched by Lite);
- it only loads the pure-variable scheme files and **never re-imports any theme stylesheet**, so switching via the dropdown cannot pull in the un-optimised original `theme.css`.

## 4. Logo modes

### How the logo works

The logo is a **PNG image** loaded through the `--logo` variable that every `colors/*.css` defines. It appears at three surfaces: the header / page-title logo (`.pageTitleWithDefaultLogo`), the login-page logo (`#loginPage h1::before`), and the Jellyfin 12+ top-bar logo (MUI toolbar). The item detail-page logo uses the item’s own artwork and is untouched.

Lite adds a two-layer mechanism: the **base layer** (original PNG, driven by `--logo-base-image`) and an **overlay layer** (a pseudo-element that paints `--logo-remix-color` clipped by your SVG, used as a `mask-image`). The variables live in `lite/logo-remix.css`; the element rules live at the end of `lite/theme.css`.

### Switching modes — user-level CSS override (recommended, works for everyone)

There is **no configuration UI**. The logo is controlled by three `:root` variables, which you can override from the same **Custom CSS Code** box — this is the recommended method and the only one available to CDN users (the `lite/` files live on the CDN and cannot be edited).

> ⚠️ **Use absolute HTTPS URLs in your own Custom CSS.** A relative `url()` that you write in Jellyfin Custom CSS resolves against the **Jellyfin page URL** — not against the CDN stylesheet — so a relative path will almost always 404. Paste one of the blocks below as-is.

**A. Nico Remix (default)** — theme-coloured silhouette, original PNG hidden (no double logo):

```css
:root {
  --logo-base-image: none;
  --logo-mask-source: url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/logos/jellyfish-remixed.svg");
  --logo-remix-color: var(--primary-accent-color);
}
```

**B. Original Jellyfish PNG**:

```css
:root {
  --logo-base-image: url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/logos/jellyfish.png");
  --logo-mask-source: none;
  --logo-remix-color: transparent;
}
```

**C. Streamberry PNG**:

```css
:root {
  --logo-base-image: url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/logos/streamberry.png");
  --logo-mask-source: none;
  --logo-remix-color: transparent;
}
```

Nico Remix is the default even without an override, so most CDN users need **no** logo override at all.

**Self-hosted alternative:** if you keep the repository on your own server you may instead edit the three `:root` lines at the top of `lite/logo-remix.css` (copy-paste blocks are provided at the bottom of that file). There, relative URLs such as `url("../logos/jellyfish.png")` work, because `logo-remix.css` and `theme.css` share the `lite/` directory (see “Using your own SVG” below).

### Using your own SVG (Nico Remix)

- **Self-hosted:** place your SVG at **`./logos/jellyfish-remixed.svg`** (i.e. `lite/logos/jellyfish-remixed.svg`) — **not** the repo-root `logos/` directory, which holds the original PNGs only. Once the SVG is in place, **no CSS edit is needed** — Nico Remix is the default mode.
- **CDN users:** you cannot add files to the CDN copy. Host your SVG anywhere public (your own server, another CDN, a GitHub Gist, …) and point `--logo-mask-source` at its absolute `https://` URL (block A above, with your URL substituted).
- Drawing requirements: single-colour silhouette with transparent background (only the alpha channel is used — the displayed colour always comes from the theme), viewBox aspect ≈ **2.36:1** (same as the original 640×271), no transparent padding, no external references. Full checklist: [`./logos/README.md`](./logos/README.md).
- Path note: a relative `url()` inside a custom-property value is resolved when the property is *used* via `var()` — against the stylesheet that **uses** it (`lite/theme.css`), not the one that declares it. Here both files share the `lite/` directory, so the resolution is identical either way; but a relative path you write in your **own** Custom CSS resolves against the Jellyfin page and will 404 — use absolute URLs (see above).

### Fallback & escape hatch

- **Mask URL 404s (missing SVG):** a failing `mask-image` hides the element it applies to (current CSS Masking behaviour — [w3c/fxtf-drafts #130](https://github.com/w3c/fxtf-drafts/issues/130)). Pure CSS cannot detect whether a mask loaded, so “hide the PNG while the SVG works” and “auto-show the PNG when the SVG is missing” are mutually exclusive. Remix mode chooses correct display: if the SVG is ever missing, the logo area is **empty** (not the PNG). **Manual fallback** = switch to mode B or C (one block).
- **Mask is `none` (no SVG applied):** if `--logo-mask-source` computes to `none` (unset, or explicitly set to `none`) while `--logo-remix-color` is an opaque colour, the overlay has no mask and renders as an **unclipped solid rectangle** covering the whole logo area. This is standard behaviour — not a browser quirk. Fix it by setting `--logo-remix-color: transparent` **and** switching to a PNG mode (B or C).
- The logo is **fully decoupled from colour schemes**: the base layer reads `--logo-base-image` (never the active scheme’s `--logo`), so switching schemes — including `jellyflix` — does not change the logo.

## 5. Optional CSS components

| Component | Lite file | Default | Purpose |
|---|---|---|---|
| `theme.css` | `lite/theme.css` | entry | main stylesheet; auto-imports the rest below; includes the Jellyfin 12+ dashboard fixes inline |
| `loginpage.css` | `lite/loginpage.css` | auto | login screen styling |
| `icontext.css` | (upstream, reused as-is) | **on** | detail-page buttons show text (Trailer / Mark-as-watched / Shuffle / More / User-rating); Play keeps its icon |
| `ratings.css` | `lite/ratings.css` | **on** | coloured parental ratings (needs `scripts/rating.js`) |
| logo remix | `lite/logo-remix.css` + `lite/logos/` | **on** | logo mode switch & variables (see §4) |
| `12_fixes.css` | `lite/12_fixes.css` | optional | Jellyfin 12+ fixes, for setups that import components separately (already inline in `theme.css`) |
| `misc.css` | `lite/misc.css` | optional | card-grid hover overlays, mobile OSD tweaks |
| `slideshow.css` | `lite/slideshow.css` | optional | custom home / slideshow plugins |
| `progress_bar.css` | `lite/progress_bar.css` | optional | progress bar styling |
| `bg_scroll.css` | `lite/bg_scroll.css` | optional | static-compatible module: same enlarged backdrop crop, infinite scroll animation removed |
| `colors/*`, `fonts/*`, `scripts/*`, `indicators.css`, `sidebar.css`, `scrollbar.css`, `card_buttons.css`, `streamberry_logo.css`, `arr-tags-links.css` | (upstream, reused) | as needed | no effects; used as-is |

**Enable / disable**:
- **Self-hosted:** edit the `@import` block at the top of `lite/theme.css` — comment a line out to disable a bundled component (the `@import` lines must stay at the top of the file).
- **CDN users:** the components bundled into `lite/theme.css` (loginpage, icontext, ratings, logo remix) load through the entry import and cannot be “un-imported” from Custom CSS. To change how they look, add higher-precedence overrides in Custom CSS **after** the theme import (later rules / `!important` win). The optional standalone files (`misc.css`, `slideshow.css`, `progress_bar.css`, `bg_scroll.css`, `12_fixes.css`) are imported separately only when you need them.

## 6. Companion scripts

The scripts live in [`../scripts/`](../scripts/) (upstream, unchanged). They provide functionality that CSS alone cannot.

- **`rating.js`** — companion to `ratings.css`. Jellyfin does not expose the content rating natively, so this script stamps a `rating` attribute onto `.mediaInfoOfficialRating`; the colour rules key off it. **Without it the badges stay grey** (`#607D8B`, same fallback as the original theme). Inject it with any JS injection method (e.g. the [Jellyfin JavaScript Injector](https://github.com/n00bcodr/Jellyfin-JavaScript-Injector) plugin).
- **`theme_selector.js`** — colour-scheme dropdown in profile settings (see §3.3).
- **`title.js`** — changes the browser tab title to “Jellyfish”.
- **`loginimage.js`** — shows the user’s avatar on the manual login screen instead of the name (⚠️ hides the username field — only use when all users are visible on the login screen).
- **`dashboard_theming.js`** — dashboard theming (⚠️ use at your own risk: Jellyfin removed dashboard custom CSS in 10.11; requires extra plugins, see the script’s own README).

Injection details and warnings: [`../scripts/README.md`](../scripts/README.md).

## 7. Compatibility

- **Jellyfin**: targets **10.11+** and retains the upstream theme’s **Jellyfin 12** styling support. Selectors are unchanged from upstream, so the compatibility surface is the same as the original theme.
- **Browsers**: the logo remix uses CSS `mask-image` (with `-webkit-` prefix) — supported by all modern browsers; §4 covers both failure modes (empty area when the SVG 404s, solid rectangle when the mask is `none`).
- **Reduced motion**: preserved `prefers-reduced-motion` fallbacks (e.g. ratings badge hover).
- **What changed vs. the original** (summary): blurs, animations, `will-change` and large glow shadows removed; transitions narrowed; visuals stay equivalent except motion/blur. Full change list, static metrics and acceptance records: [`../AUDIT.md`](../AUDIT.md).

## 8. Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| Theme does not apply | Check the import URL is exactly `https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css` and keep **one** entry import. When self-hosting, the directory structure (`fonts/`, `colors/`, `logos/`, `lite/`) must be intact for the relative imports. Look at the browser Network tab for 404s and hard-refresh. |
| Logo area is empty (remix mode) | The mask SVG is missing or its URL 404s. Default install: `lite/logos/jellyfish-remixed.svg` must exist (**not** repo-root `logos/`). If you overrode `--logo-mask-source` with a relative URL in Custom CSS, that 404s by design — use an absolute URL (§4). Switch to a PNG mode as a manual fallback. |
| Solid colour rectangle instead of a logo | `--logo-mask-source` computes to `none` while `--logo-remix-color` is an opaque colour — the overlay renders unmasked. Set `--logo-remix-color: transparent` and switch to a PNG mode (§4). |
| Coloured ratings show grey | `scripts/rating.js` is not injected — install it (§6). Grey is the expected fallback without it. |
| Colour switch has no effect | The override must be imported **after** `lite/theme.css` (§3.1); the dynamic selector script points at upstream CDN URLs (§3.3). |
| Fonts / icons 404 | Relative imports need the surrounding directories; when self-hosting, serve `fonts/` (woff2) with the correct MIME type (`font/woff2`) and, if cross-origin, the right CORS headers. |
| Dark block behind detail-page content | The translucent block (`--secondary-background-transparent`, 20% of the primary colour) is the **original design** (layered over Jellyfin base styling), kept identical in Lite by decision. |
| You still see the original theme’s effects | Only import `lite/` URLs — never mix in the root `theme.css` (the un-optimised one). |

## 9. License & attribution

- Upstream: [https://github.com/n00bcodr/Jellyfish](https://github.com/n00bcodr/Jellyfish) — Copyright (C) 2025 **n00bcodr** — [DBAD Public License](../LICENSE.md) (upstream, unchanged).
- This Lite variant **retains the original attribution and license**; every file header notes the variant. No upstream file has been modified.
- When redistributing, follow the DBAD license (keep attribution; do not rename-and-sell as-is, etc.).
- Internal documentation (change list, static metrics, acceptance records): [`../AUDIT.md`](../AUDIT.md) (Chinese).
