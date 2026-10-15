<p align="center">
  <img src="./lite/logos/jellyfish-remixed.svg" alt="Jellyfish Lite — Remixed by Nico" width="460">
</p>

<p align="center">
  <a href="./README.md">English</a> | <a href="./README.zh-CN.md">简体中文</a>
</p>

<p align="center">
  A lighter, smoother Jellyfin experience.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Jellyfin-10.11%2B-AA5CC3?logo=jellyfin&logoColor=white" alt="Jellyfin 10.11+">
  <img src="https://img.shields.io/badge/Theme-CSS-00A4DC" alt="CSS Theme">
  <img src="https://img.shields.io/badge/Color%20Schemes-15-8A63D2" alt="15 Color Schemes">
</p>

---

## About

**Jellyfish Lite** is a performance-focused remix of [Jellyfish](https://github.com/n00bcodr/Jellyfish), the Jellyfin theme created by [n00bcodr](https://github.com/n00bcodr).

The original Jellyfish theme brings a distinctive visual style to Jellyfin. Jellyfish Lite builds on that foundation with a different priority: **keeping the experience beautiful while making everyday browsing feel more responsive.**

Instead of redesigning the interface, Lite focuses on reducing expensive CSS effects and unnecessary animations while preserving the original layouts, color schemes, and familiar interactions.

**Same jellyfish. Less turbulence.** 🪼

## What's Different?

### Lighter Visual Effects

Jellyfish Lite reduces CSS rendering overhead by removing or simplifying:

- Expensive backdrop blur effects
- Continuous background animations
- Large decorative glow effects
- Unnecessary hover animations and transitions
- Persistent rendering hints such as `will-change`

Essential interface feedback and functional transitions are retained where appropriate.

### Familiar Design

Lite aims to preserve the original Jellyfish experience:

- Original layouts and responsive styling
- All 15 original color schemes
- Existing fonts, icons, and visual identity
- Compatibility with supported optional theme components

The goal is refinement, not a visual overhaul.

### A Little Nico Remix

Jellyfish Lite includes a custom SVG logo designed by Nico.

The logo uses CSS masking to adapt to the selected theme's accent color. Logo selection is independent of the color scheme, with three available modes:

- **Nico Remix** — the custom theme-colored SVG logo
- **Original Jellyfish** — the original Jellyfish logo
- **Streamberry** — the alternative Streamberry logo

### Small Quality-of-Life Changes

- Detail-page action labels enabled by default
- Colored parental rating styles included
- A single CSS entry point for the main theme
- Optional components available separately

**Note:** Colored parental ratings require the companion `rating.js` script to apply their colors. Without the script, the fallback appearance is used.

## Quick Installation

Open your Jellyfin administrator dashboard and navigate to:

**Dashboard → General → Custom CSS Code**

Add:

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
```

Save your changes and refresh Jellyfin.

That's it! The main Lite theme, default color scheme, and included styles are loaded through this single import.

For individual user settings, alternative colors, logo modes, and optional components, see the detailed documentation.

> **Version note:** The URL above follows the latest `main` branch. For a stable installation, use a tagged release when one becomes available.

## Customization

Jellyfish Lite retains all 15 original Jellyfish color schemes:

`aurora` · `banana` · `coal` · `coral` · `forest` · `grass` · `jellyblue` · `jellyflix` · `jellypurple` · `lavender` · `midnight` · `mint` · `ocean` · `peach` · `watermelon`

The default is **JellyBlue**.

You can also select a different logo independently of your chosen color scheme.

For configuration instructions, optional CSS modules, companion scripts, and troubleshooting, see:

**[Detailed Documentation](./lite/README.md)** · **[中文详细说明](./lite/README.zh-CN.md)**

## Compatibility

Jellyfish Lite primarily targets **Jellyfin 10.11+** and retains the upstream theme's Jellyfin 12 styling support.

Compatibility can vary depending on the Jellyfin client, installed plugins, and customizations.

The theme has been manually tested in a Jellyfin environment, with smoother scrolling observed in poster grids and episode lists compared with the original theme. These observations are not intended as universal performance benchmarks.

The Jellyfin administration dashboard has separate styling limitations inherited from Jellyfin itself.

## Documentation

| Resource | Description |
|---|---|
| [Detailed Guide](./lite/README.md) | Installation, configuration, and optional features |
| [中文使用说明](./lite/README.zh-CN.md) | Detailed documentation in Simplified Chinese |
| [Technical Audit](./AUDIT.md) | CSS optimization details and development notes |
| [Original README](./README.upstream.md) | Original Jellyfish documentation |
| [Original Jellyfish](https://github.com/n00bcodr/Jellyfish) | Upstream project |

## Credits & License

Jellyfish Lite is an unofficial community remix of **[Jellyfish by n00bcodr](https://github.com/n00bcodr/Jellyfish)**.

The original theme was inspired by other community projects, including Flow, Zesty, and Ultrachromic. Credit for the original design and its upstream contributions belongs to their respective creators.

**Remix and performance optimizations:** Nico, with AI-assisted development.

**Custom Lite logo:** Nico.

The original project's license is preserved in [LICENSE.md](./LICENSE.md). Please review the upstream **DBAD Public License** before redistributing or modifying the project.

This project is not affiliated with or officially endorsed by Jellyfin or the original Jellyfish author.

---

<p align="center">
  <strong>Jellyfish Lite — Remixed by Nico</strong>
</p>

<p align="center">
  Made for smoother scrolling and happier jellyfish. 🪼
</p>

