# Jellyfish Lite

> **[English](./README.md) · [简体中文](./README.zh-CN.md)**

> Jellyfish Lite is a **performance-lightened variant** of the [Jellyfish theme](https://github.com/n00bcodr/Jellyfish) by **n00bcodr** (DBAD Public License — see [`../LICENSE.md`](../LICENSE.md)). All functionality is preserved; decorative motion and expensive rendering effects are trimmed to reduce rendering pressure. The upstream files in the repository root are **untouched** — this `lite/` directory is a standalone, drop-in replacement theme.

---

## 1. Overview

Jellyfish Lite keeps every feature of the original theme (layout, colours, fonts, interactions, plugin compatibility) while removing or narrowing the things that cost rendering performance:

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

> ⚠️ **Availability**: this fork is not published yet — the `lite/` files have not been pushed, so the CDN URL above will **404 until the repository is pushed and public**. For local testing, serve the repository with any static server (e.g. `python3 -m http.server` in the repo root) and import e.g. `http://<your-host>:8000/lite/theme.css`. (jsDelivr’s `gh` proxy serves any *public* repository without a release; it becomes usable once this repository is public.)

Keep the repository directory structure intact — every internal `@import` (fonts, icons, login page, default colour scheme, icontext, ratings, logo remix) is **relative** to the entry file and resolves against the same origin. Do not copy `lite/theme.css` somewhere on its own; it needs `fonts/`, `colors/`, `logos/`, `lite/` around it.

The only external dependency is the Druidblack metadata-icon CDN, identical to the original theme (unchanged).

## 3. Colour schemes

All 15 original schemes are preserved untouched (`colors/*.css`, pure `:root` CSS variables, no effects):

`aurora` · `banana` · `coal` · `coral` · `forest` · `grass` · `jellyblue` · `jellyflix` · `jellypurple` · `lavender` · `midnight` · `mint` · `ocean` · `peach` · `watermelon`

- **Default**: `jellyblue` (imported in `lite/theme.css`).
- Switching a scheme only swaps CSS variables — **`lite/theme.css` is never re-imported**, so the performance optimisations always stay active.

### 3.1 Server-wide (global)

Edit the colour `@import` line in `lite/theme.css`:

```css
@import url("../colors/ocean.css");   /* global default becomes Ocean */
```

### 3.2 Per user (Display settings)

Import `lite/theme.css` first, then add the target scheme **after** it (later-imported `:root` variables win):

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/colors/peach.css"); /* this device uses Peach */
```

(Self-hosted? Replace the URLs with your own static server paths.)

### 3.3 Dynamic dropdown (script)

[`../scripts/theme_selector.js`](../scripts/theme_selector.js) adds a theme dropdown to the user’s profile settings (including a random-daily option). Notes:

- the script embeds **upstream** CDN links (`n00bcodr/Jellyfish`) — if you self-host a fork, update the URLs inside the script (`scripts/` are upstream files, untouched by Lite);
- it only loads the pure-variable scheme files and **never re-imports any theme stylesheet**, so switching via the dropdown cannot pull in the un-optimised original `theme.css`.

## 4. Logo modes

### How the logo works

The logo is a **PNG image** loaded through the `--logo` variable that every `colors/*.css` defines. It appears at three surfaces: the header / page-title logo (`.pageTitleWithDefaultLogo`), the login-page logo (`#loginPage h1::before`), and the Jellyfin 12+ top-bar logo (MUI toolbar). The item detail-page logo uses the item’s own artwork and is untouched.

Lite adds a two-layer mechanism: the **base layer** (original PNG, driven by `--logo-base-image`) and an **overlay layer** (a pseudo-element that paints `--logo-remix-color` clipped by your SVG, used as a `mask-image`). The variables live in `lite/logo-remix.css`; the element rules live at the end of `lite/theme.css`.

> There is **no configuration UI** — switching modes means editing the three `:root` variables in `lite/logo-remix.css` (copy-paste blocks are provided at the bottom of that file).

### The three modes

Replace the three `:root` lines in `lite/logo-remix.css` with one of the following:

**A. Nico Remix (default)** — theme-coloured silhouette, original PNG hidden (no double logo):

```css
--logo-base-image: none;
--logo-mask-source: url("logos/jellyfish-remixed.svg");
--logo-remix-color: var(--primary-accent-color);
```

**B. Original Jellyfish PNG**:

```css
--logo-base-image: url("../logos/jellyfish.png");
--logo-mask-source: none;
--logo-remix-color: transparent;
```

**C. Streamberry PNG**:

```css
--logo-base-image: url("../logos/streamberry.png");
--logo-mask-source: none;
--logo-remix-color: transparent;
```

### Using your own SVG (Nico Remix)

- Place your SVG at **`./logos/jellyfish-remixed.svg`** (i.e. `lite/logos/jellyfish-remixed.svg`) — **not** the repo-root `logos/` directory, which holds the original PNGs only. Relative URLs in custom-property values resolve against the stylesheet that declares them (`lite/`), so a file in the root `logos/` dir will 404.
- Drawing requirements: single-colour silhouette with transparent background (only the alpha channel is used — the displayed colour always comes from the theme), viewBox aspect ≈ **2.36:1** (same as the original 640×271), no transparent padding, no external references. Full checklist: [`./logos/README.md`](./logos/README.md).
- Once the SVG is in place, **no CSS edit is needed** — Nico Remix is the default mode.

### Fallback & escape hatch

- CSS cannot detect whether a mask loaded (failing masks hide their element — [w3c/fxtf-drafts #130](https://github.com/w3c/fxtf-drafts/issues/130)), so “hide the PNG while the SVG works” and “auto-show the PNG when the SVG is missing” are mutually exclusive. Remix mode chooses correct display: if the SVG is ever missing, the logo area is **empty** (not the PNG). **Manual fallback** = switch to mode B or C (one block).
- If a deviant browser renders a failed mask as an unclipped solid rectangle, set `--logo-remix-color: transparent` to neutralise the overlay (and switch to a PNG mode to display the original PNG).
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

**Enable / disable**: edit the `@import` block at the top of `lite/theme.css` (comment a line out to disable), or add per-user imports **after** the `lite/theme.css` import in Custom CSS (later imports win for variables).

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
- **Browsers**: the logo remix uses CSS `mask-image` (with `-webkit-` prefix) — supported by all modern browsers; the escape hatch in §4 covers deviant rendering.
- **Reduced motion**: preserved `prefers-reduced-motion` fallbacks (e.g. ratings badge hover).
- **What changed vs. the original** (summary): blurs, animations, `will-change` and large glow shadows removed; transitions narrowed; visuals stay equivalent except motion/blur. Full change list, static metrics and acceptance records: [`../AUDIT.md`](../AUDIT.md).

## 8. Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| Theme does not apply | Only **one** entry import (`lite/theme.css`); the directory structure (`fonts/`, `colors/`, `logos/`, `lite/`) must be intact for the relative imports. Check the browser Network tab for 404s and hard-refresh. |
| Logo area is empty (remix mode) | The SVG is missing or at the wrong path — it must be `lite/logos/jellyfish-remixed.svg`, **not** repo-root `logos/`. Restore the file, or switch to a PNG mode (§4) as a manual fallback. |
| Solid colour rectangle instead of a logo | A deviant browser rendered the failed mask as unclipped — set `--logo-remix-color: transparent` (§4). |
| Coloured ratings show grey | `scripts/rating.js` is not injected — install it (§6). Grey is the expected fallback without it. |
| Colour switch has no effect | The per-user override must be imported **after** `lite/theme.css` (§3.2); the dynamic selector script points at upstream CDN URLs (§3.3). |
| Fonts / icons 404 | Relative imports need the surrounding directories; when self-hosting, serve `fonts/` (woff2) with the correct MIME type (`font/woff2`) and, if cross-origin, the right CORS headers. |
| Dark block behind detail-page content | The translucent block (`--secondary-background-transparent`, 20% of the primary colour) is the **original design** (layered over Jellyfin base styling), kept identical in Lite by decision. |
| You still see the original theme’s effects | Only import `lite/` URLs — never mix in the root `theme.css` (the un-optimised one). |

## 9. License & attribution

- Upstream: [https://github.com/n00bcodr/Jellyfish](https://github.com/n00bcodr/Jellyfish) — Copyright (C) 2025 **n00bcodr** — [DBAD Public License](../LICENSE.md) (upstream, unchanged).
- This Lite variant **retains the original attribution and license**; every file header notes the variant. No upstream file has been modified.
- When redistributing, follow the DBAD license (keep attribution; do not rename-and-sell as-is, etc.).
- Internal documentation (change list, static metrics, acceptance records): [`../AUDIT.md`](../AUDIT.md) (Chinese).
