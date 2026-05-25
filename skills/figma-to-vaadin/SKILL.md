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
compatibility: Requires Figma MCP server and Vaadin MCP server
metadata:
  mcp-server: figma
---

# Figma to Vaadin Implementation

## Scope

This skill produces Vaadin Flow (Java) code that reproduces the **layout and component
structure** of a Figma design. It does NOT configure global theme tokens, brand colors, or
typography — that belongs to a separate theme configuration skill.

Sample data is allowed and encouraged to make components visible. Real services and business
logic are out of scope.

## Workflow

Create TODOs from these steps and follow them in order.

### 1. Read the design via `get_design_context`
- Most detailed component information; check `data-name` for component type
- Note theme/variant hints and text styles

### 2. Check component annotations
For each component instance, apply these in order: recommended Vaadin component, theme
variants, accessibility requirements, implementation notes, documentation links. Annotations
override guesses from layer names.

### 3. Read structure via `get_metadata`
- Layer names may not match Vaadin component names
- Map the hierarchy and identify layout patterns

### 4. Research each component (mandatory)
- `search_vaadin_docs` to find candidates, record `file_path`
- `get_full_document` for **every** component before implementing — search results are
  previews, not enough on their own
- `get_component_java_api` for the exact Java method signatures — use this whenever you
  need to know which methods a component exposes (e.g. slot setters, theme variants, sizing)

### 5. Pick the layout styling approach

One of three values: `lumo-utility`, `vaadin-css`, `tailwind`.

**Resolve in this order — stop at the first match:**

1. **`.agent-context` in the project root has `layout-approach: <value>`** → use it.
2. **App class has BOTH `@StyleSheet(Lumo.STYLESHEET)` AND `@StyleSheet(Lumo.UTILITY_STYLESHEET)`** → `lumo-utility`.
3. **Otherwise — ask the user.** Do not guess. Do not default. Present all three:
   > "Which layout approach should I use?
   > - **Vaadin layout APIs + plain CSS** — always works, no setup
   > - **Lumo Utility classes** — requires `@StyleSheet(Lumo.UTILITY_STYLESHEET)` in your app class
   > - **Tailwind** — only if Tailwind is already configured"

**After resolving:**
- Write the decision to `.agent-context` so it's not asked again:
  ```
  layout-approach: vaadin-css
  ```
- Read the matching reference before writing any layout code:
  `lumo-utility` → `references/layouts-lumo-utility.md`
  `vaadin-css` → `references/layouts-vaadin-css.md`
  `tailwind` → `references/layouts-tailwind.md`

### 6. Implement
- Use Vaadin components, not generic HTML
- Apply theme variants via Java API (`addThemeVariants`)
- Use the layout patterns from the chosen reference
- Pick correct heading levels from text styles
- Add accessibility attributes where needed

### 7. Do not run anything
- No terminal commands, no browser, no screenshots, no tests
- **Never search for Java source files** (e.g., `find ~/.m2 -name "*.java"`) to discover
  API methods. The Vaadin docs MCP (`get_component_java_api`, `get_full_document`) is the
  authoritative source for all component APIs — if a method isn't documented there, ask the
  user rather than hunting source files.
- If a compile error suggests a method doesn't exist (e.g., `add()` undefined on a
  component), re-read the component's Java API docs via `get_component_java_api` before
  guessing or running any commands.

## Sample data

Make components visually meaningful with small, realistic sample data:

- Define in a `private` helper method (e.g., `createSampleOrders()`)
- 3–5 items max
- Realistic values (`"Alice Johnson"`, not `"Item 1"`)
- Add `// Sample data — replace with real service call` comment
- Prefer `List.of(...)` for immutable collections

## Universal component patterns

These apply regardless of the styling approach chosen in Step 5.

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

Other traps:
- For purely visual containers with no flex semantics, prefer `Div` — it avoids all of the above defaults
- `flex-shrink` is on by default — a fixed-size child shrinks when placed next to a `setWidthFull()` sibling; call `layout.setFlexShrink(component, 0)` to prevent it, or use `layout.setFlexGrow(fullSizeComponent, 1)` instead of `setWidthFull()` to avoid the conflict altogether
- `setWidthFull()` on a child in a content-hugging `HorizontalLayout` expands the layout rather than fitting it; use `setAlignItems(STRETCH)` instead
- A layout child's minimum size defaults to its content size; this causes unexpected scrollbars in `Scroller` / `TabSheet`; fix with `component.setMinWidth("0")` or `setMinHeight("0")`

## When to ask for clarification

- Multiple Vaadin components could fit the visual design
- Figma component name doesn't map clearly to a Vaadin component

Ask: "Should this be a [ComponentA] or [ComponentB]? The Figma shows [description]"

## Quick reference: Figma → Vaadin

| Figma | Vaadin |
|---|---|
| Vertical auto layout | `VerticalLayout` |
| Horizontal auto layout | `HorizontalLayout` |
| Free / absolute layout | `FlexLayout` |
| Form / labelled fields | `FormLayout` |
| Master-detail | `MasterDetailLayout` ¹ |
| Button | `Button` |
| Text Field | `TextField` |
| Grid | `Grid` |
| Avatar | `Avatar` |
| Card | `Card` (v24.8+) |
| Badge / status label | `Badge` ² |
| Text layer | `com.vaadin.flow.component.html.Span` |
| Heading 3 | `com.vaadin.flow.component.html.H3` |

¹ **MasterDetailLayout requires a feature flag.** Add this line to
`src/main/resources/vaadin-featureflags.properties` (create the file if it doesn't exist):
```
com.vaadin.experimental.masterDetailLayoutComponent=true
```

² **Badge requires a feature flag.** Add this line to `vaadin-featureflags.properties`:
```
com.vaadin.experimental.badgeComponent=true
```