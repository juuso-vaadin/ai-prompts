# Layouts: Tailwind utility classes

Use this approach when the project has Tailwind configured. Tailwind classes are applied via `addClassNames(...)` like any other CSS class.

**Required:** Tailwind CSS support in Vaadin requires adding `com.vaadin.experimental.tailwindCss=true` to `src/main/resources/vaadin-featureflags.properties`.

## Required setup when using Tailwind with Vaadin

Two files must be present for Tailwind to use Aura/Vaadin design tokens correctly.

### 1. `tailwind.config.js` — map Tailwind to Aura CSS variables

Place this file at `src/main/resources/META-INF/resources/tailwind.config.js` and import it at the top of `styles.css`:

```css
@config "./tailwind.config.js";
```

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{html,js,ts,jsx,tsx,java}'],
  theme: {
    extend: {
      // ─── Typography ───────────────────────────────────────────────
      fontSize: {
        xs: ['var(--aura-font-size-xs)', { lineHeight: 'var(--aura-line-height-xs)' }],
        s:  ['var(--aura-font-size-s)',  { lineHeight: 'var(--aura-line-height-s)' }],
        m:  ['var(--aura-font-size-m)',  { lineHeight: 'var(--aura-line-height-m)' }],
        l:  ['var(--aura-font-size-l)',  { lineHeight: 'var(--aura-line-height-l)' }],
        xl: ['var(--aura-font-size-xl)', { lineHeight: 'var(--aura-line-height-xl)' }],
      },
      lineHeight: {
        xs: 'var(--aura-line-height-xs)',
        s:  'var(--aura-line-height-s)',
        m:  'var(--aura-line-height-m)',
        l:  'var(--aura-line-height-l)',
        xl: 'var(--aura-line-height-xl)',
      },

      // ─── Spacing — gap ────────────────────────────────────────────
      gap: {
        xs: 'var(--vaadin-gap-xs)',
        s:  'var(--vaadin-gap-s)',
        m:  'var(--vaadin-gap-m)',
        l:  'var(--vaadin-gap-l)',
        xl: 'var(--vaadin-gap-xl)',
      },

      // ─── Spacing — padding ────────────────────────────────────────
      padding: {
        xs: 'var(--vaadin-padding-xs)',
        s:  'var(--vaadin-padding-s)',
        m:  'var(--vaadin-padding-m)',
        l:  'var(--vaadin-padding-l)',
        xl: 'var(--vaadin-padding-xl)',
        'block-container':  'var(--vaadin-padding-block-container)',
        'inline-container': 'var(--vaadin-padding-inline-container)',
      },

      // ─── Colors ───────────────────────────────────────────────────
      colors: {
        container: 'var(--vaadin-background-container)',
        red:    'var(--aura-red)',
        orange: 'var(--aura-orange)',
        yellow: 'var(--aura-yellow)',
        green:  'var(--aura-green)',
        blue:   'var(--aura-blue)',
        purple: 'var(--aura-purple)',
        'red-text':    'var(--aura-red-text)',
        'orange-text': 'var(--aura-orange-text)',
        'yellow-text': 'var(--aura-yellow-text)',
        'green-text':  'var(--aura-green-text)',
        'blue-text':   'var(--aura-blue-text)',
        'purple-text': 'var(--aura-purple-text)',
        text: {
          secondary: 'var(--vaadin-text-color-secondary)',
          disabled:  'var(--vaadin-text-color-disabled)',
        },
        border: {
          secondary: 'var(--vaadin-border-color-secondary)',
        },
        accent:            'var(--aura-accent-color)',
        'accent-contrast': 'var(--aura-accent-contrast-color)',
        'accent-text':     'var(--aura-accent-text-color)',
      },

      // ─── Shadows ──────────────────────────────────────────────────
      boxShadow: {
        xs: 'var(--aura-shadow-xs)',
        s:  'var(--aura-shadow-s)',
        m:  'var(--aura-shadow-m)',
      },

      // ─── Border radius ────────────────────────────────────────────
      borderRadius: {
        sm: 'var(--vaadin-radius-s)',
        md: 'var(--vaadin-radius-m)',
        lg: 'var(--vaadin-radius-l)',
      },
    },
  },
  plugins: [],
}
```

### 2. `@theme` block in your global stylesheet — connect Tailwind spacing to Vaadin tokens

Add this to your main stylesheet (e.g. `styles.css`).
This maps Tailwind's built-in spacing scale keys (`xs`, `s`, `m`, `l`, `xl`) to Vaadin padding tokens, so utilities like `p-m`, `gap-s`, `m-l` resolve to the correct design-system values:

```css
@theme {
  --spacing-xs: var(--vaadin-padding-xs);
  --spacing-s:  var(--vaadin-padding-s);
  --spacing-m:  var(--vaadin-padding-m);
  --spacing-l:  var(--vaadin-padding-l);
  --spacing-xl: var(--vaadin-padding-xl);
}
```

With this in place, use semantic spacing keys that resolve to design tokens:

```java
layout.addClassNames("p-m", "gap-s");
```

**Note:** Class names must be present literally in source files.
The Tailwind compiler scans files at build time and only generates CSS for class names it finds statically.
Dynamic construction (`"p-" + size`) will not work.

## Decision table

| Layout need | Preferred class(es) |
|---|---|
| Vertical stacking | `VerticalLayout` (preferred) or `flex flex-col` |
| Horizontal stacking | `HorizontalLayout` (preferred) or `flex flex-row` |
| Padding (all sides) | `p-xs` / `p-s` / `p-m` / `p-l` / `p-xl` |
| Padding per axis | `px-m py-s` |
| Padding per side | `pt-s pr-m pb-s pl-m` |
| Gap | `gap-xs` / `gap-s` / `gap-m` / `gap-l` / `gap-xl` |
| Gap on rows / columns only | `gap-y-s` / `gap-x-m` |
| Width | `w-full` / `w-64` |
| Height | `h-full` / `h-screen` |
| Min/max width | `min-w-0` / `max-w-md` |
| Flex direction | `flex-row` / `flex-col` |
| Flex wrap | `flex-wrap` / `flex-nowrap` |
| Justify content | `justify-between` / `justify-center` / `justify-end` |
| Align items | `items-center` / `items-baseline` / `items-start` |
| Align self override | `self-end` / `self-center` |
| Flex grow / shrink | `grow` / `shrink-0` / `flex-1` |
| 2-D grid | `grid grid-cols-3 gap-m` |
| Grid column span | `col-span-2` |
| Position | `sticky top-0` / `absolute` / `relative` |
| Overflow clip | `overflow-hidden` |
| Text truncation | `truncate` (= `overflow-hidden text-ellipsis whitespace-nowrap`) |
| Hide / show | `hidden` / `block` / responsive: `hidden md:block` |
| Font size | `text-xs` / `text-s` / `text-m` / `text-l` / `text-xl` |
| Text color — secondary | `text-text-secondary` |
| Background color — container | `bg-container` |
| Accent color | `bg-accent text-accent-contrast` |
| Badge / status color | `bg-green text-green-text` (or red/orange/yellow/blue/purple) |
| Border radius | `rounded-sm` / `rounded-md` / `rounded-lg` |
| Shadow | `shadow-xs` / `shadow-s` / `shadow-m` |

## Examples

```java
// ✅ Semantic spacing with Aura tokens
HorizontalLayout header = new HorizontalLayout();
header.setSpacing(false);
header.setPadding(false);
header.addClassNames("w-full", "px-m", "py-s", "justify-between", "items-center");

// ✅ Colors and typography from Aura tokens
Div badge = new Div("Warning");
badge.addClassNames("bg-orange", "text-orange-text", "text-s", "rounded-sm", "p-xs");

// ✅ Accent button-like styling
Div cta = new Div("Submit");
cta.addClassNames("bg-accent", "text-accent-contrast", "rounded-md", "p-s");

// ✅ Per-child override
content.addClassNames("flex-1");   // child fills remaining space

// ✅ 2-D grid using token gap
Div grid = new Div();
grid.addClassNames("grid", "grid-cols-3", "gap-m");

// ✅ Responsive
sidebar.addClassNames("hidden", "md:block", "md:w-64");

// ❌ Don't use the style API when Tailwind covers it
layout.getStyle().set("padding", "16px");
layout.getStyle().set("display", "flex");

// ❌ Don't use raw numeric scale when a token key exists
layout.addClassNames("p-4");   // use p-m or p-s instead
```

## Spacing on `VerticalLayout` / `HorizontalLayout`

These components apply default spacing and padding. When using Tailwind for explicit control, disable the defaults first:

```java
HorizontalLayout row = new HorizontalLayout();
row.setSpacing(false);
row.setPadding(false);
row.addClassNames("gap-s", "p-m");
```

## Avoid margin between siblings

Use `gap-*` on the parent rather than margin on each child:

```java
// ❌ Avoid
child.addClassNames("mb-4");

// ✅ Prefer
parent.addClassNames("flex", "flex-col", "gap-s");
```
