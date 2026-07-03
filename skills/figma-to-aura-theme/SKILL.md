---
name: figma-to-aura-theme
description: Map a Figma Aura design system to Vaadin Aura theme CSS configuration. Use when the user provides a Figma URL and wants to configure the Aura theme to match it. Triggers on requests like "set up Aura theme from Figma", "configure Aura to match my design", "generate Aura CSS from Figma", or when a Figma URL is combined with any Aura theming request. Applies when the target app uses the Aura theme (`@StyleSheet(Aura.STYLESHEET)`, Vaadin's default from 25.0 onwards) or has no theme configured yet. Does NOT apply to apps still on the classic Lumo theme (`@StyleSheet(Lumo.STYLESHEET)`, no Aura import) — use figma-to-lumo-theme for those instead.
---

# Figma to Aura Theme

Extract visual styling from a Figma design and translate it into a Vaadin Aura theme CSS file.

## Overview

Aura is a higher-level theming system than Lumo. It uses a small set of properties (accent color, background, surface level, density, radius, font, color scheme) that internally derive many Lumo variables. Because of this, Figma's design variables do **not** map 1:1 to Aura CSS properties — Figma encodes computed values, while Aura works from abstract inputs.

**The goal is not to copy Figma values verbatim — it is to find the Aura configuration that produces a result matching the Figma design.**

## Required Workflow

Create TODOs based on these steps.

- Step 1: Extract Figma variables from all available modes
- Step 2: Map variables to Aura properties
- Step 3: Infer visual properties from `get_design_context`
- Step 4: Generate the Aura CSS file

---

### Step 1: Extract Figma Variables from All Available Modes

**Start with `get_variable_defs`** on a representative node. It returns values for the file's
current/default mode only — it takes no mode parameter, and there's no documented way to switch
modes from outside and have it pick that up. If the file only has one mode, that's all you need.

If the file has **multiple modes** (e.g. light/dark), use `use_figma` instead, and read each
variable's value across all of its modes in one pass (`variable.valuesByMode`, keyed by mode) —
rather than trying to toggle the active mode and re-call `get_variable_defs`, which isn't how mode
selection works. Before your first `use_figma` call in this workflow, load its required `figma-use`
guidance (skill or MCP resource) — its own instructions mark this mandatory.

**Color scheme rule:**
- If the file has **both light and dark modes** → always implement both; set `color-scheme: light dark`
- If only **one mode** exists → implement that mode only; set `color-scheme: light` or `color-scheme: dark`

Variables to capture from each mode:

| Figma Variable | Light Mode Maps To | Dark Mode Maps To |
|---|---|---|
| `aura-accent-color` | `--aura-accent-color-light` | `--aura-accent-color-dark` |
| `aura-background-color` | `--aura-background-color-light` | `--aura-background-color-dark` |
| `vaadin-user-color-0` … `vaadin-user-color-9` | `--vaadin-user-color-0` … `--vaadin-user-color-9` | (same values, single declaration) |
| `Components/Field border tint` | `--vaadin-input-field-border-color` | (same or mode-specific) |
| `aura-border-color` | Informs input field border color | — |
| Font variable (e.g. `lumo-font-family`) | `--aura-font-family` | — |
| Font size variable (e.g. `lumo-font-size-m`) | `--aura-base-font-size` | — |

> **Note:** The Figma Aura design system may label some variables with `lumo-` prefixes (e.g. `lumo-font-family`, `lumo-font-size-m`). These are equivalent to their Aura counterparts and should be mapped to Aura CSS properties. Ignore any `lumo-` variables that have no Aura equivalent.

---

### Step 2: Map Figma Variables to Aura Properties

Aura uses named colors from a predefined list — do not use raw hex values from Figma unless no named color matches closely (see color matching below).

#### Accent Color

The `aura-accent-color` variable from Figma is the **light mode** accent. Aura requires separate light and dark accent values.

1. Compare the light mode hex to the Aura **Light Accent Colors** table in `property-values.md`
2. Find the closest named color by hue (keep the same color family — blue stays blue, green stays green)
3. Use the **paired dark variant** of the same named color for `--aura-accent-color-dark`

```css
/* Example: Figma aura-accent-color = #3266e4 → closest named = Default blue */
--aura-accent-color-light: #3266e4;   /* or omit if it IS the default */
--aura-accent-color-dark: #3266e4;    /* paired dark variant */
```

Only set these if they differ from the Aura defaults (`#3266e4` for both).

#### Background Color

The `aura-background-color` variable from Figma is the **light mode** background. Aura requires separate light and dark background values.

1. Compare the light mode hex to the Aura **Light Background Colors** table in `property-values.md`
2. Find the closest named background by hue and tone
3. Use the **paired dark variant** of the same named background for dark mode

If the background is tinted by the accent color (vibrant/colorful design), use the Accent background formula:
```css
--aura-background-color-light: oklch(from var(--aura-accent-color-light) 0.9 calc(c * 0.3) h);
--aura-background-color-dark: oklch(from var(--aura-accent-color-dark) 0.18 calc(c * 0.3) h);
```

Only set these if they differ from the Aura defaults (`#F4F5F7` / `#151922`).

#### Font Family

Map the font variable to `--aura-font-family`. Prefer fonts from the curated list in `property-values.md`. Add the corresponding Google Fonts `@import` at the top of the CSS file.

```css
/* Figma: lumo-font-family = "Instrument Sans" → not in curated list → omit or use closest */
/* Figma: lumo-font-family = "Inter" → use Inter */
--aura-font-family: 'Inter', var(--aura-font-family-system);
```

If the font from Figma is not in the curated list, omit `--aura-font-family` (defaults to Instrument Sans).

#### Font Size

The Figma font-size-m variable maps to `--aura-base-font-size`. Use the closest value from the allowed set: `13`, `14` (default), `15`, `16`.

Only set if it differs from the default (`14`).

#### User Colors

`vaadin-user-color-0` through `vaadin-user-color-9` map directly. Only set values that differ from the Aura defaults. If Figma has fewer than 10 user colors defined, only set the ones present.

```css
html {
  --vaadin-user-color-0: #3266e4;
  --vaadin-user-color-1: #00d2cd;
  /* ... */
}
```

#### What Does NOT Map Directly

These Aura properties have no corresponding Figma variable — infer them visually in Step 4:

| Aura Property | How to Infer |
|---|---|
| `--aura-base-size` | Component density from `get_design_context` |
| `--aura-base-radius` | Border radius from component screenshots |
| `--aura-surface-level` | Surface elevation/layering from the design |
| `--aura-surface-opacity` | Whether surfaces are semi-transparent or opaque |
| `--aura-contrast-level` | Text/border contrast from the design |
| `--aura-app-layout-inset` | Whether the app layout has an inset/margin |
| `--aura-overlay-surface-opacity` | Dialog/overlay rendering from screenshots |

---

### Step 3: Infer Visual Properties from Figma

Call `get_design_context` on a representative frame (preferably an application shell or dashboard view). Use the screenshot and code hints to infer the properties listed above.

Look for:
- **Border radius:** Check button, input, and card corner rounding. Map to Aura's discrete values (`-1`, `0`, `3`, `4`, `7`)
- **Density:** Check component heights and spacing. Map to `12` (compact), `16` (default), `20` (spacious)
- **Surface level:** Check if cards/panels appear elevated, flat, or deeply layered. Map to `-0.5`, `1`, or `2`
- **App layout inset:** Check if the main content area has a gap/margin from the viewport edge. `0px` = no inset
- **Color scheme for nav vs content:** If the side nav is dark and content is light, set `--aura-content-color-scheme: light` alongside `color-scheme: dark`

For **input field styling**, check if the Figma design shows custom input field borders or backgrounds. If the `Components/Field border tint` or `aura-border-color` variables are meaningful (non-zero, non-default), configure input field properties:

```css
html {
  --vaadin-input-field-border-color: <value>;
  --vaadin-input-field-border-width: 1px; /* required when border-color is set */
}
```

---

### Step 4: Generate the Aura CSS File

Follow the file creation workflow:

1. Locate `styles.css` (default: `/src/main/resources/META-INF/resources/styles.css`)
2. Choose a descriptive filename for the theme CSS file
3. Create the CSS file in the same directory
4. Add `@import "filename.css";` at the top of `styles.css`

**Only include properties that differ from Aura defaults.** Do not set properties that match the default values. Use the default values documented in this skill (or looked up via the Vaadin MCP's `get_theme_css_properties` with `theme: "aura"`, and the app's Vaadin version) — don't guess a default from memory; a wrong assumption produces a CSS declaration that looks theme-driven but is actually just silently re-asserting (or subtly missing) the real default.

**CSS structure:**

```css
/* Font import — only if using a Google Font */
@import url('https://fonts.googleapis.com/css2?family=...');

html {
  /* Color scheme */
  color-scheme: light dark;

  /* Accent colors — only if different from default #3266e4 */
  --aura-accent-color-light: #009966;
  --aura-accent-color-dark: #34D399;

  /* Background — only if different from defaults */
  --aura-background-color-light: #ffffff;
  --aura-background-color-dark: #18181b;

  /* Typography — only non-defaults */
  --aura-font-family: 'Inter', var(--aura-font-family-system);
  --aura-base-font-size: 15;

  /* Layout and visual style — only non-defaults */
  --aura-base-radius: 4;
  --aura-base-size: 16;
  --aura-surface-level: 1;
  --aura-app-layout-inset: 0px;

  /* User colors — only if customized */
  --vaadin-user-color-0: #3266e4;
  --vaadin-user-color-1: #00d2cd;
}

/* Input field styling — only if border is customized */
html {
  --vaadin-input-field-border-color: #2941702e;
  --vaadin-input-field-border-width: 1px;
}
```

**For mixed mode (dark nav, light content):**
```css
html {
  color-scheme: dark;
  --aura-content-color-scheme: light;
}
```

---

## Color Matching Reference

When matching a Figma hex to a named Aura color, compare hues:

| Hue range | Named Aura Color |
|---|---|
| Red ~0° | Red |
| Orange ~25° | Orange |
| Amber/Yellow ~40-55° | Amber / Yellow |
| Yellow-Green ~75° | Lime |
| Green ~130-150° | Green / Emerald |
| Teal ~170° | Teal |
| Cyan ~185° | Cyan |
| Sky ~200° | Sky |
| Blue ~215-225° | Blue (Default) |
| Indigo ~240° | Indigo |
| Violet ~265° | Violet |
| Purple ~280° | Purple |
| Fuchsia ~295° | Fuchsia |
| Pink ~320° | Pink |
| Rose ~345° | Rose |
| Neutral / Black/White | Neutral |

Always pair the light and dark variants of the **same named color**.
