<img width="900" height="180" alt="code_gen_banner" src="https://github.com/user-attachments/assets/5256577f-4651-4e8f-8f69-4d0c5eee922f" />


# Figma to Vaadin agent skills
Collection of AI skills for translating Figma designs to Vaadin applications.

This repository contains three complementary skills that work with Figma MCP and Vaadin MCP to streamline the design-to-code workflow:

### 1. Figma to Vaadin (UI implementation)
**Purpose:** Translate Figma designs to well-structured Vaadin Flow code

**Use this skill when you need to:**
- Implement UI components from Figma designs
- Convert design layouts to Vaadin views
- Generate semantic and accessible Vaadin code

**Workflow:**
1. Extract design context from Figma selection using Figma MCP
2. Check component annotations for implementation guidance
3. Fetch relevant Vaadin documentation via Vaadin MCP
4. Generate proper Vaadin components with correct themes

**Example prompt:**
```
Implement the selection in Figma. Follow the provided guidelines.
```

### 2. Figma to Lumo Theme
**Purpose:** Map Figma design tokens to Lumo CSS variables

**Use this skill when you need to:**
- Apply custom theme from Figma to your Vaadin app
- Generate CSS variable declarations for Lumo
- Customize colors, typography, spacing from design system

**Workflow:**
1. Extract design tokens from Figma using `get_variable_defs`
2. Categorize tokens by type (colors, typography, spacing, etc.)
3. Map to corresponding Lumo CSS variables  
4. Generate `styles.css` with only non-default values

**Example prompt:**
```
Apply the Figma design tokens to Lumo theme. Follow the provided guidelines.
```

### 3. Figma to Aura Theme
**Purpose:** Map a Figma Aura design system to the Vaadin Aura theme

**Use this skill when you need to:**
- Configure the Aura theme (Vaadin's default theme from 25.0 onwards) to match a Figma design
- Generate Aura CSS custom properties (accent color, background, density, radius, etc.) from Figma variables
- Support light/dark mode variants defined in Figma

**Workflow:**
1. Extract Figma variables from all available modes (light/dark) using `use_figma` and `get_variable_defs`
2. Map variables to Aura's higher-level properties — not a 1:1 copy, since Aura derives many values from a small set of inputs
3. Infer visual properties (density, radius, surface level) from `get_design_context`
4. Generate the Aura theme CSS file with only non-default values

**Example prompt:**
```
Set up the Aura theme to match this Figma design. Follow the provided guidelines.
```

## How to Use

### Prerequisites
- AI coding assistant with MCP support (GitHub Copilot, Claude Code, or similar)
- Desktop version of Figma
- Connection to Figma MCP and Vaadin MCP servers

### Setup
The agent skill definitions are found in `skills/`. The skill structure is generic, but different tools may load the skills slightly differently.

Figma MCP can be used **remotely** (link to Figma file) or **locally** (Figma Desktop app). Remove one or the other from the `mcp.json` file before using the skills.
Project contains a sample MCP server configuration file for VS Code in the `.vscode` directory.

### Usage
1. Make a selection in Figma.
2. Right click to see context menu and select "Copy link to selection". Alternatively you can copy the URL with node ID from browser's address bar.
3. In your code editor, use the appropriate slash command in the AI chat and paste the URL pointing to a Figma node:
   - `/figma-to-vaadin` for UI code implementation
   - `/figma-to-lumo-theme` for configuring Lumo theme
   - `/figma-to-aura-theme` for configuring Aura theme
4. The AI agent will follow the skill guidelines to generate accurate code or styles.

#### Example prompt
```
/figma-to-vaadin Create new view based on https://www.figma.com/design/ExAMpLeID/My-Figma-File?node-id=1-234&t=DABCINP9G1B1d8Wi-1
```

## Technical Details

Both skills follow a consistent workflow:
- Always extract design information from Figma MCP first
- Fetch relevant Vaadin documentation before generating code
- Focus on accuracy over speed
- Skip tests and validations (manual verification is expected)

## Useful Links
- [Vaadin MCP Documentation](https://mcp.vaadin.com/docs/)
- [Figma MCP Developer Documentation](https://developers.figma.com/docs/figma-mcp-server)
