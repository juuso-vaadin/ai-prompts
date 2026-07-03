---
name: figma-to-vaadin
description: >
  Translate Figma designs into Vaadin Flow (Java) UI code using the Figma MCP and Vaadin MCP.
  Use this skill whenever the user wants to implement a Figma frame, screen, or component as
  Vaadin Java code — even if they just say "implement this design", "generate Vaadin code from
  Figma", "convert this frame to Java", or paste a Figma URL. Does NOT apply to React, HTML,
  web components, or other frontend frameworks — only Vaadin Flow (Java). Does NOT apply to
  design-only tasks such as editing Figma files or generating Figma components. Does NOT
  configure themes or visual design tokens — that is a separate skill.
compatibility: Requires a Figma MCP server and the Vaadin MCP server
---

# Figma to Vaadin Implementation

## Scope

This skill produces Vaadin Flow (Java) code that reproduces the **layout and component
structure** of a Figma design. It does not configure global theme tokens, brand colors, or
typography — that belongs to a separate theme configuration skill.

The main failure mode this skill guards against is jumping straight to code from a guess.
Gather enough context — the design, its annotations, and the real Vaadin API — before writing
anything.

## Workflow

Create TODOs from these steps and follow them in order.

### 1. Fetch design context

`get_design_context` on the given node is the primary source — it has the most detailed
component information; check `data-name` for component type, and note theme/variant hints and
text styles. If the response is truncated (very large or deeply nested frames), fall back to
`get_metadata` for the layer hierarchy, then call `get_design_context` on the specific child
nodes you need.

### 2. Check component annotations

For each component instance, apply these in order: recommended Vaadin component, theme
variants, accessibility requirements, implementation notes, documentation links. Annotations
override guesses from layer names.

If a Figma component still doesn't map clearly to one Vaadin component after checking
annotations, ask: "Should this be a [ComponentA] or [ComponentB]? The Figma shows
[description]." Don't guess.

### 3. Research each component (mandatory)

Never rely on memorized Vaadin knowledge — API surfaces and feature-flag status change between
versions.

- `search_vaadin_docs` to find candidates, record `file_path`
- `get_full_document` for **every** component before implementing — search results are
  previews, not enough on their own
- `get_component_java_api` for the exact Java method signatures — use this whenever you
  need to know which methods a component exposes (slot setters, theme variants, sizing)

If a compile error suggests a method doesn't exist, re-read the component's Java API docs
before guessing at a fix. Don't search local `.m2` jars for source, and don't run anything to
"just try it" — the docs are the authoritative source.

### 4. Resolve project preferences, once

Before implementing, resolve the preferences below. Check `.agent-context` in the project root
first — if a value is already recorded, use it and don't ask again. For anything unresolved,
ask the user (ideally in one combined question), then append all the answers to
`.agent-context` so future runs don't ask again:

```
layout-approach: vaadin-css
architecture: single-view
sample-data: generate-sample
verification: skip
```

| Preference | Values | Auto-detect | Otherwise |
|---|---|---|---|
| `layout-approach` | `lumo-utility` / `vaadin-css` / `tailwind` | App class has both `@StyleSheet(Lumo.STYLESHEET)` and `@StyleSheet(Lumo.UTILITY_STYLESHEET)` → `lumo-utility`. `vaadin-featureflags.properties` has `com.vaadin.experimental.tailwindCss=true` → `tailwind`. Aura-themed apps have no equivalent signal — ask. | "Which layout approach should I use — Vaadin layout APIs + plain CSS (always works, no setup), Lumo Utility classes (requires `@StyleSheet(Lumo.UTILITY_STYLESHEET)`), or Tailwind (only if already configured)?" |
| `architecture` | `single-view` / `composed-components` | No reliable signal | "Should I build this as one view class with private helper methods, or split it into reusable components (e.g. a separate detail/edit form that fires its own save/cancel events)?" |
| `sample-data` | `generate-sample` / `use-existing-data` | Check whether the project already has a repository, service, or entity matching the data shown in the design | "Should I generate small sample data for this view, or is there existing data/service in the project I should wire it to instead?" |
| `verification` | `skip` / `verify` / `verify-and-fix` | No reliable signal | "After implementing, should I skip testing, run visual verification against the Figma design, or run visual verification and automatically apply one round of fixes based on the findings?" |

Read the matching layout reference before writing any layout code:
`lumo-utility` → `references/layouts-lumo-utility.md`
`vaadin-css` → `references/layouts-vaadin-css.md`
`tailwind` → `references/layouts-tailwind.md`

### 5. Implement

- Use Vaadin components, not generic HTML; prefer the component API over the element/style API
  (e.g. `textField.setReadOnly(true)`, not `.getElement().setAttribute("readonly", "")`)
- Apply theme variants via Java API (`addThemeVariants`)
- Use the layout patterns from the chosen reference
- Pick correct heading levels from text styles
- Add accessibility attributes where needed (e.g. `setAriaLabel` on icon-only buttons)

If `architecture: composed-components` — split the view into a container plus reusable
sub-components (e.g. a details/edit form as its own class). Sub-components fire custom
`ComponentEvent`s (e.g. `SaveEvent`, `CancelEvent`) that the container listens for and acts on,
rather than the container reaching into the sub-component's fields directly.

If `sample-data: generate-sample`:
- Define it in a `private` helper method (e.g. `createSampleOrders()`)
- 3–5 items max, or enough to match what the design visually shows (e.g. a scrolling grid) if
  that density is core to the layout
- Realistic values (`"Alice Johnson"`, not `"Item 1"`)
- Add `// Sample data — replace with real service call` comment
- Prefer `List.of(...)` for immutable collections

If `sample-data: use-existing-data`, wire the view to the existing repository/service/entity
instead of inventing new sample data.

### 6. Test

This skill's own job — writing code — is done by the end of Step 5. Don't run terminal
commands, open a browser, or take screenshots yourself; what happens next depends on the
`verification` preference resolved in Step 4:

- **`skip`** — stop here.
- **`verify`** — invoke the `vaadin-visual-verification` skill, passing it the Figma URL (or
  `fileKey`/`nodeId`) used for this view and the route it was implemented at. Present its
  prioritized findings to the user as-is; don't act on them yet.
- **`verify-and-fix`** — invoke `vaadin-visual-verification` the same way, then apply exactly
  **one** round of fixes addressing its findings, highest severity first. Tell the user what was
  changed and why. Don't loop back into a second verification pass automatically — if the user
  wants to confirm the fixes, that's a new verification run.

## Universal component patterns

These apply regardless of the styling approach.

```java
// ✅ Component API over element/style API
textField.setReadOnly(true);
button.addThemeVariants(ButtonVariant.LUMO_TERTIARY);
avatar.addThemeVariants(AvatarVariant.LUMO_LARGE);
iconButton.setAriaLabel("Close");
input.setLabel("Label");                      // HasLabel API, not a separate Span

// ✅ Sizing via component API
layout.setSizeFull();
layout.setWidth("600px");

// ❌ Never use the style API for things the component API handles
textField.getElement().setAttribute("readonly", "");
button.getElement().getStyle().set("background", "transparent");
layout.getStyle().set("width", "600px");
avatar.getStyle().set("--vaadin-avatar-size", "48px");
```

## Gotchas

`VerticalLayout` defaults:
- Padding ON — call `setPadding(false)` if not wanted
- Width 100% of parent
- `alignItems` START — children do not stretch horizontally; call `setAlignItems(STRETCH)` or `setWidthFull()` per child to fill the width
- `justifyContentMode` controls the vertical (main) axis

`HorizontalLayout` defaults:
- Padding OFF
- Width shrinks to content — call `setWidthFull()` if it should fill the parent
- `alignItems` STRETCH — children stretch vertically to fill the layout height (a `Button` next to a `TextField` will silently grow)
- `justifyContentMode` controls the horizontal (main) axis
- A layout child's minimum size defaults to its content size; this causes unexpected scrollbars in `Scroller` / `TabSheet`; fix with `component.setMinWidth("0")` or `setMinHeight("0")`

- For purely visual containers prefer `FlexLayout` — it avoids all of the above defaults
- `flex-shrink` is on by default — a fixed-size child shrinks when placed next to a `setWidthFull()` sibling; call `layout.setFlexShrink(component, 0)` to prevent it, or use `layout.setFlexGrow(fullSizeComponent, 1)` instead of `setWidthFull()` to avoid the conflict altogether
- `setWidthFull()` on a child in a content-hugging `HorizontalLayout` expands the layout rather than fitting it; use `setAlignItems(STRETCH)` instead
- A layout child's minimum size defaults to its content size; this causes unexpected scrollbars in `Scroller` / `TabSheet`; fix with `component.setMinWidth("0")` or `setMinHeight("0")`. The same default also applies one level up: a component like `MasterDetailLayout` or `Scroller` placed as the `expand()`ed child of a `VerticalLayout` (or a CSS Grid area) can resist shrinking below its content's natural height even with `setSizeFull()`. If a view overflows the page instead of scrolling internally, add `setMinHeight("0")` to that expanded child itself, not just to a `Scroller` nested further inside it
- `RadioButtonGroup` / `CheckboxGroup` default orientation is theme-dependent: horizontal in Lumo, **vertical in Aura**. If the Figma layer is named/laid out horizontally and the project uses Aura, add `addThemeVariants(RadioGroupVariant.AURA_HORIZONTAL)` / `CheckboxGroupVariant.AURA_HORIZONTAL` — otherwise the group silently renders as a vertical stack
- Feature-flag status changes between versions — don't assume a component needs one from memory; check `search_vaadin_docs("feature flags")` then `get_full_document` on the result
- Never use CSS `margin` to space out a Vaadin layout component from its container — margin sits outside the component's measured box, which breaks `setSizeFull()`/`expand()` height math (a component can measure "correct" while still visually overflowing its parent). Add spacing instead via padding on a wrapping layout, or by targeting the component's own shadow-DOM part with `::part(...)` (e.g. `vaadin-master-detail-layout::part(detail) { padding: ...; }`)
- When writing custom CSS, use real theme CSS custom properties — look them up with the Vaadin MCP (`get_theme_css_properties`) rather than inventing a plausible-sounding variable name with a hardcoded `var(--name, fallback)` fallback. If the name doesn't actually exist, the fallback silently becomes the real value and never tracks the theme (e.g. `var(--vaadin-background-color-secondary, #f9fafb)` — that property doesn't exist; the real one is `--vaadin-background-container`)
- `VerticalLayout`/`HorizontalLayout`/`FlexLayout` already set `box-sizing: border-box` themselves, so padding on them is safe by default. Only plain elements — a custom CSS rule targeting a `Div`, another non-layout component, or a shadow-DOM `::part(...)` — need `box-sizing: border-box` added explicitly when the rule also sets `padding`; without it, padding adds to the element's declared width/height instead of being carved out of it, so a component sized with `setWidth()`/`setSizeFull()` ends up visually larger than intended

## Quick reference: Figma → Vaadin

| Figma | Vaadin |
|---|---|
| Vertical auto layout | `VerticalLayout` |
| Horizontal auto layout | `HorizontalLayout` |
| Free / absolute layout | `FlexLayout` |
| Form / labelled fields | `FormLayout` |
| Master-detail | `MasterDetailLayout` |
| Button | `Button` |
| Text Field | `TextField` |
| Grid / Table | `Grid` |
| Avatar | `Avatar` |
| Card | `Card` (v24.8+) |
| Badge / status label | `Badge` |
| Text layer | `com.vaadin.flow.component.html.Span` |
| Heading 3 | `com.vaadin.flow.component.html.H3` |