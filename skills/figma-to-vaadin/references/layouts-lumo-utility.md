# Layouts: LumoUtility classes

Use this approach when the project already imports `LumoUtility` classes or the user requests
them. `LumoUtility` provides ready-made class names for spacing, padding, sizing, alignment,
flex, and grid — applied via `addClassNames(...)`.

## Decision table

| Layout need | Approach |
|---|---|
| Vertical stacking | `VerticalLayout` |
| Horizontal stacking | `HorizontalLayout` |
| Padding | `addClassNames(LumoUtility.Padding.MEDIUM)` |
| Padding on one side | `addClassNames(LumoUtility.Padding.Bottom.LARGE)` |
| Horizontal padding only | `addClassNames(LumoUtility.Padding.Horizontal.MEDIUM)` |
| Vertical padding only | `addClassNames(LumoUtility.Padding.Vertical.MEDIUM)` |
| Spacing between children | `addClassNames(LumoUtility.Gap.MEDIUM)` |
| Gap on rows / columns only | `LumoUtility.Gap.Row.MEDIUM` / `LumoUtility.Gap.Column.MEDIUM` |
| Margin (avoid — use padding on parent) | `LumoUtility.Margin.*` (last resort) |
| Width | `addClassNames(LumoUtility.Width.FULL)` or `setWidthFull()` |
| Height | `addClassNames(LumoUtility.Height.SCREEN)` or `setHeightFull()` |
| Min/max width | `LumoUtility.MinWidth.NONE`, `LumoUtility.MaxWidth.SCREEN_MEDIUM` |
| Flex direction override | `LumoUtility.FlexDirection.ROW` / `COLUMN` |
| Flex wrap | `LumoUtility.FlexWrap.WRAP` |
| Justify content | `LumoUtility.JustifyContent.BETWEEN` / `CENTER` / `END` |
| Align items | `LumoUtility.AlignItems.CENTER` / `BASELINE` / `START` |
| Align self override | `LumoUtility.AlignSelf.END` |
| Flex grow | `LumoUtility.Flex.GROW` |
| Display | `LumoUtility.Display.FLEX` / `GRID` / `BLOCK` / `NONE` |
| 2-D grid | `LumoUtility.Display.GRID` + `LumoUtility.Grid.Column.COLUMNS_3` |
| Position | `LumoUtility.Position.STICKY` / `ABSOLUTE` |
| Overflow clip | `LumoUtility.Overflow.HIDDEN` |
| Text truncation | `LumoUtility.TextOverflow.ELLIPSIS` + `Overflow.HIDDEN` + `Whitespace.NOWRAP` |

### Size scale

LumoUtility sizes follow `XSMALL`, `SMALL`, `MEDIUM`, `LARGE`, `XLARGE`. Pick the closest match
to the Figma value.

## Examples

```java
// ✅ Layout structure with LumoUtility
HorizontalLayout header = new HorizontalLayout();
header.addClassNames(
    LumoUtility.Padding.Horizontal.LARGE,
    LumoUtility.Padding.Vertical.MEDIUM,
    LumoUtility.JustifyContent.BETWEEN,
    LumoUtility.AlignItems.CENTER,
    LumoUtility.Width.FULL
);

// ✅ Mixing LumoUtility with layout Java API is fine
VerticalLayout content = new VerticalLayout();
content.setSpacing(false);                       // disable default spacing
content.addClassNames(LumoUtility.Gap.LARGE);    // apply explicit gap

// ✅ Per-child override
content.add(sidebar);
sidebar.addClassNames(LumoUtility.AlignSelf.END);

// ✅ 2-D grid via LumoUtility
Div grid = new Div();
grid.addClassNames(
    LumoUtility.Display.GRID,
    LumoUtility.Grid.Column.COLUMNS_3,
    LumoUtility.Gap.MEDIUM
);

// ❌ Don't use the style API when LumoUtility covers it
layout.getStyle().set("padding", "16px");
layout.getStyle().set("display", "flex");
layout.getStyle().set("justify-content", "space-between");
```

## Scroller, not `Overflow.AUTO`

For scrollable regions, use `Scroller` rather than `LumoUtility.Overflow.AUTO`. Reserve
`Overflow.HIDDEN` for clipping (no scrollbar).

```java
Scroller scroller = new Scroller(content);
scroller.setScrollDirection(Scroller.ScrollDirection.VERTICAL);
scroller.setSizeFull();
```

## Avoid margin

Vaadin's component model favors padding on the parent over margin on the child. Reach for
`LumoUtility.Margin.*` only when no other option exists.

```java
// ❌ Avoid
child.addClassNames(LumoUtility.Margin.Bottom.MEDIUM);

// ✅ Prefer — padding/gap on the parent
parent.addClassNames(LumoUtility.Gap.MEDIUM);
// or
parent.setSpacing("var(--vaadin-gap-m)");
```

## Combining with `setSpacing` / `setPadding`

`VerticalLayout` and `HorizontalLayout` apply default spacing and padding. When using
`LumoUtility` for explicit control, disable the defaults first to avoid double-spacing:

```java
HorizontalLayout row = new HorizontalLayout();
row.setSpacing(false);
row.setPadding(false);
row.addClassNames(LumoUtility.Gap.SMALL, LumoUtility.Padding.MEDIUM);
```