# Layouts: Vaadin APIs + plain CSS

Use this approach when the project does **not** use LumoUtility classes or Tailwind. It uses
Vaadin layout component Java APIs for structure, and plain CSS with `--vaadin-*` custom
properties for what the APIs cannot express.

## Java API first — decision table

| Layout need | Java API |
|---|---|
| Vertical stacking | `VerticalLayout` |
| Horizontal stacking | `HorizontalLayout` |
| Default theme spacing | `layout.setSpacing(true)` — uses `--vaadin-gap` |
| Custom spacing value | `layout.setSpacing("var(--vaadin-gap-l)")` — accepts any CSS value |
| No spacing | `layout.setSpacing(false)` |
| Default theme padding | `layout.setPadding(true)` — uses `--vaadin-padding` |
| Custom padding value | `layout.setPadding("var(--vaadin-padding-m)")` |
| No padding | `layout.setPadding(false)` |
| Cross-axis alignment | `layout.setAlignItems(...)` / `layout.setDefaultHorizontalComponentAlignment(...)` |
| Main-axis distribution | `layout.setJustifyContentMode(...)` |
| One child fills remaining space | `layout.expand(child)` |
| Per-child alignment override | `layout.setAlignSelf(Alignment.END, child)` |
| Width / height | `component.setWidth("...")`, `component.setHeight("...")` |
| Fill parent | `component.setSizeFull()` |
| Min/max constraints | `component.setMinWidth(...)`, `setMaxWidth(...)` |
| Flex direction (custom) | `flex.setFlexDirection(FlexLayout.FlexDirection.ROW)` |
| Flex wrap | `flex.setFlexWrap(FlexLayout.FlexWrap.WRAP)` |
| Scrollable region | `new Scroller(content)` + `setScrollDirection(...)` |
| Responsive form columns | `FormLayout` + `setResponsiveSteps(...)` |
| Resizable split panels | `SplitLayout` |

### Available spacing tokens (`--vaadin-gap-*`, `--vaadin-padding-*`)
`xs`, `s`, `m`, `l`, `xl` — pick the size that matches the Figma value.

```java
// ✅ Spacing/padding with the layout Java API
HorizontalLayout header = new HorizontalLayout();
header.setSpacing("var(--vaadin-gap-l)");
header.setPadding("var(--vaadin-padding-m)");
header.setWidthFull();
header.setAlignItems(FlexComponent.Alignment.CENTER);
header.setJustifyContentMode(FlexComponent.JustifyContentMode.BETWEEN);
header.expand(titleLabel);

// ✅ Per-child flex-grow
VerticalLayout sidebar = new VerticalLayout();
sidebar.expand(contentArea);

// ❌ Don't use the style API for things the layout API covers
layout.getStyle().set("display", "flex");
layout.getStyle().set("gap", "16px");      // → setSpacing("var(--vaadin-gap-m)")
layout.getStyle().set("padding", "16px");  // → setPadding("var(--vaadin-padding-m)")
```

## Falling back to CSS

When the layout API can't express a structural need, add a scoped CSS class with
`addClassName("descriptive-name")` and write the rule in `styles.css`. Use `--vaadin-*`
custom properties rather than hard-coded values.

| Need | CSS |
|---|---|
| Custom gap on a non-Vaadin-layout container | `gap: var(--vaadin-gap-m);` |
| 2-D grid (rows AND columns) | `display: grid; grid-template-columns: ...; gap: var(--vaadin-gap-m);` |
| Aspect ratio | `aspect-ratio: 16 / 9;` |
| Sticky / absolute positioning | `position: sticky; top: 0;` |
| Clip overflow (no scroll) | `overflow: hidden;` |
| Text truncation | `overflow: hidden; text-overflow: ellipsis; white-space: nowrap;` |

```java
Div grid = new Div();
grid.addClassName("dashboard-grid");
```
```css
.dashboard-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: var(--vaadin-gap-m);
}
```

## Scroller, not `overflow: auto`

For any region that should scroll, use `Scroller` — not CSS `overflow: auto`. Reserve CSS
`overflow: hidden` for clipping (no scrollbar).

```java
Scroller scroller = new Scroller(content);
scroller.setScrollDirection(Scroller.ScrollDirection.VERTICAL);
scroller.setSizeFull();
```

## CSS class naming

- Kebab-case, role-based: `order-summary-card`, `toolbar-actions`
- Not visual: ❌ `blue-background`, `padding-top-16`

## Known gap

Arbitrary spacing values that don't map to an `xs`/`s`/`m`/`l`/`xl` token must be hard-coded
for now. If the project uses Aura, the entire scale can be tuned via `--aura-base-size`. If
this is a frequent need, the project should consider switching to the Tailwind approach.