# Layouts: Tailwind utility classes

Use this approach when the project has Tailwind configured. Tailwind classes are applied via
`addClassNames(...)` like any other CSS class. Tailwind handles arbitrary spacing values
gracefully via its `[...]` bracket syntax.

## Decision table

| Layout need | Tailwind classes |
|---|---|
| Vertical stacking | `VerticalLayout` (preferred) or `flex flex-col` |
| Horizontal stacking | `HorizontalLayout` (preferred) or `flex flex-row` |
| Padding (all sides) | `p-4` / `p-6` / `p-[24px]` |
| Padding per axis | `px-4 py-2` |
| Padding per side | `pt-2 pr-4 pb-2 pl-4` |
| Gap | `gap-2` / `gap-4` / `gap-[18px]` |
| Gap on rows / columns only | `gap-y-2` / `gap-x-4` |
| Width | `w-full` / `w-64` / `w-[600px]` |
| Height | `h-full` / `h-screen` / `h-[400px]` |
| Min/max width | `min-w-0`, `max-w-md`, `max-w-[800px]` |
| Flex direction | `flex-row` / `flex-col` |
| Flex wrap | `flex-wrap` / `flex-nowrap` |
| Justify content | `justify-between` / `justify-center` / `justify-end` |
| Align items | `items-center` / `items-baseline` / `items-start` |
| Align self override | `self-end` / `self-center` |
| Flex grow / shrink | `grow` / `shrink-0` / `flex-1` |
| 2-D grid | `grid grid-cols-3 gap-4` |
| Grid column span | `col-span-2` |
| Position | `sticky top-0` / `absolute` / `relative` |
| Overflow clip | `overflow-hidden` |
| Text truncation | `truncate` (= `overflow-hidden text-ellipsis whitespace-nowrap`) |
| Hide / show | `hidden` / `block` / responsive: `hidden md:block` |

### Tailwind spacing scale
`0`, `0.5`, `1`, `1.5`, `2`, `2.5`, `3`, `4`, `5`, `6`, `8`, `10`, `12`, `16`, `20`, `24`, ...
Each unit is `0.25rem` (4px) by default. Use bracket syntax `[24px]` for arbitrary values
when no scale value matches the Figma design.

## Examples

```java
// ✅ Layout structure with Tailwind
HorizontalLayout header = new HorizontalLayout();
header.setSpacing(false);
header.setPadding(false);
header.addClassNames("w-full", "px-6", "py-3", "justify-between", "items-center");

// ✅ Bracket syntax for exact Figma values
Div card = new Div();
card.addClassNames("p-[18px]", "gap-[14px]");   // when 18px / 14px don't match the scale

// ✅ Per-child override
content.addClassNames("flex-1");                // child fills remaining space

// ✅ 2-D grid
Div grid = new Div();
grid.addClassNames("grid", "grid-cols-3", "gap-4");

// ✅ Responsive
sidebar.addClassNames("hidden", "md:block", "md:w-64");

// ❌ Don't use the style API when Tailwind covers it
layout.getStyle().set("padding", "16px");
layout.getStyle().set("display", "flex");
```

## Spacing on `VerticalLayout` / `HorizontalLayout`

These components apply default spacing and padding. When using Tailwind for explicit control,
disable the defaults first:

```java
HorizontalLayout row = new HorizontalLayout();
row.setSpacing(false);
row.setPadding(false);
row.addClassNames("gap-2", "p-4");
```

## Scroller, not `overflow-auto`

For scrollable regions, prefer `Scroller`. Use Tailwind's `overflow-hidden` for clipping
without scroll. Use `overflow-auto` / `overflow-y-auto` only on simple cases where pulling in
`Scroller` would be over-engineering.

```java
Scroller scroller = new Scroller(content);
scroller.setScrollDirection(Scroller.ScrollDirection.VERTICAL);
scroller.setSizeFull();
```

## Avoid margin between siblings

Use `gap-*` on the parent rather than margin on each child:

```java
// ❌ Avoid
child.addClassNames("mb-4");

// ✅ Prefer
parent.addClassNames("flex", "flex-col", "gap-4");
```