---
name: figma-prototype
description: Generate a Figma plugin (code.js + manifest.json) from a UX spec file and a Figma reference file. Use when the user says "figma prototype", "create Figma screens", "generate prototype", "build Figma frames", "prototype this", or when called by ux-review after PM approves prototype creation.
---

# Figma Prototype Generator

Generate a ready-to-run Figma plugin from a UX spec file. The plugin creates all specified screens with correct brand tokens, exact copy, and wired prototype connections — ready to run in Figma desktop in under 30 seconds.

## Why a Plugin, Not Direct API

The Figma REST API is read-only for design nodes. The Figma MCP is also read-only. The only supported path to programmatically create Figma frames is the Figma Plugin API — which requires a one-time manual run in Figma desktop. This skill generates that plugin.

## Inputs Required

1. **UX spec file path** — a markdown file containing screen list, exact copy, layout descriptions, and prototype flow connections
2. **Figma reference file key** — the key from the URL of an existing Figma file to extract brand tokens from (e.g. `bq3p5LfmYuMG5K5p7pEL4M`)
3. **Figma PAT** — personal access token for the REST API (`X-Figma-Token`). Ask the user if not already provided.
4. **Feature slug** — used to name the output directory (e.g. `partial-auth`)
5. **Output Figma file URL** — the Figma file where the plugin will be run (for run instructions)

If any input is missing, ask before proceeding. Do not guess file keys or tokens.

## Step 1: Read the UX Spec

Read the UX spec file fully. Extract:

- List of all screens (names, purpose, trigger conditions)
- Exact copy for each screen: nav title, headline, body, CTA labels
- Layout descriptions (content blocks, icon placement, stacked CTAs)
- Prototype connections: which tap/action navigates to which screen, any auto-navigate with timeout
- Option variants (e.g. Option A vs Option B) — note which is preferred
- Business flow paths to annotate (success, decline, abandonment, etc.)

## Step 2: Extract Brand Tokens from Reference File

Call the Figma REST API to read the reference file:

```
GET https://api.figma.com/v1/files/{file_key}?geometry=paths
Header: X-Figma-Token: {PAT}
```

Then fetch the key reference frame (e.g. the main review/confirmation screen node):

```
GET https://api.figma.com/v1/files/{file_key}/nodes?ids={node_id}&geometry=paths
Header: X-Figma-Token: {PAT}
```

Extract and record:
- Frame width and height (default: 390×844 if not found)
- Background fill color
- Primary button fill color
- Primary text color
- Secondary text color
- Accent/teal/active color
- List/card background color
- Divider/border color
- Font family (usually "Inter")

Convert all colors to Figma RGB format (0–1 range: divide hex values by 255).

**Rule**: Never hardcode colors you haven't read from the reference file. If a specific token can't be found, note it as `[UNVERIFIED]` and use a reasonable fallback, calling it out in the output.

## Step 3: Generate code.js

Write the full Figma plugin `code.js`. Structure:

### Constants block

```javascript
// Brand tokens extracted from reference file
const C = {
  primary:       { r: x/255, g: x/255, b: x/255 }, // #xxxxxx
  white:         { r: 1,     g: 1,     b: 1       },
  textPrimary:   { r: x/255, g: x/255, b: x/255 }, // #xxxxxx
  textSecondary: { r: x/255, g: x/255, b: x/255 }, // #xxxxxx
  // ... all extracted tokens
};
const FONT = "Inter"; // or extracted font name
const W = 390; const H = 844; // or extracted dimensions
```

### Helper functions

- `makeFrame(name, x, y)` — creates a frame at position, sets size and background fill
- `addText(parent, content, opts)` — adds a text node with font, size, color, alignment
- `addRect(parent, opts)` — adds a filled rectangle (for buttons, dividers, cards)
- `addButton(parent, label, opts)` — adds a CTA button (rect + centered text label)
- `wire(fromNode, toNode, type, delay?)` — adds a prototype reaction

### Prototype wiring

```javascript
function wire(fromNode, toNode, type = "ON_CLICK", delay = 0) {
  fromNode.reactions = [...(fromNode.reactions || []), {
    action: {
      type: "NODE",
      destinationId: toNode.id,
      navigation: "NAVIGATE",
      transition: null,
      preserveScrollPosition: false
    },
    trigger: delay > 0 ? { type: "AFTER_DELAY", delay } : { type: type }
  }];
}
```

### Screen creation functions

One function per screen. Each function:
1. Calls `makeFrame` with the screen name
2. Adds status bar placeholder at top
3. Adds navigation bar with back arrow + title from UX spec
4. Adds content block per UX spec layout (icon, headline, body, amount highlight)
5. Adds CTAs (primary first, secondary below if applicable)
6. Adds home indicator at bottom
7. Returns the frame node (needed for wiring)

### Section backgrounds

Add labeled backgrounds to visually separate Option A / Option B variants:

```javascript
const bg = figma.createRectangle();
bg.name = "── Option B: Post-Review (Preferred) ──";
bg.resize(totalWidth + padding, H + labelHeight);
bg.x = startX - padding / 2;
bg.y = -labelHeight;
bg.fills = [{ type: "SOLID", color: sectionColor, opacity: 0.05 }];
figma.currentPage.insertChild(0, bg); // behind all frames
```

### Main execution block

```javascript
(async () => {
  await figma.loadFontAsync({ family: FONT, style: "Regular" });
  await figma.loadFontAsync({ family: FONT, style: "SemiBold" });
  await figma.loadFontAsync({ family: FONT, style: "Bold" });

  // Create section backgrounds first (z-index 0)
  // Create all screens
  // Wire all prototype connections per UX spec
  // Scroll viewport to show all screens: figma.viewport.scrollAndZoomIntoView([...allFrames])

  figma.notify("✓ Created N screens: Screen1, Screen2, ...");
  figma.closePlugin();
})();
```

## Step 4: Generate manifest.json

```json
{
  "name": "{Feature Display Name} Prototype",
  "id": "{feature-slug}-prototype-tgo-{year}",
  "api": "1.0.0",
  "main": "code.js",
  "editorType": ["figma"]
}
```

Use the feature slug for the id. Year = current year.

## Step 5: Save Files

Save both files to `product-docs/{feature-slug}-figma-plugin/`:

- `product-docs/{feature-slug}-figma-plugin/code.js`
- `product-docs/{feature-slug}-figma-plugin/manifest.json`

The `product-docs/` directory is at `~/transfergo/product-docs/` (one level above the monolith repo, not inside it).

## Step 6: Output Run Instructions

After saving, output:

```
## Plugin Ready

Files saved to:
  product-docs/{feature-slug}-figma-plugin/code.js
  product-docs/{feature-slug}-figma-plugin/manifest.json

### How to run in Figma (~30 seconds)
1. Open your output Figma file: {output_file_url}
2. Menu → Plugins → Development → Import plugin from manifest…
3. Select: ~/transfergo/product-docs/{feature-slug}-figma-plugin/manifest.json
4. Menu → Plugins → Development → {plugin name} → Run

### Screens created
Option B (preferred):
  - [screen list with descriptions]
Option A:
  - [screen list with descriptions]

Prototype connections wired: [count]
Flow annotations: [count]

### Items requiring Figma confirmation after running
- [list any [CONFIRM] items from the UX spec that need verifying against live frames]
```

## Template Reference

Use `/Users/michal.szczotka/transfergo/product-docs/partial-auth-figma-plugin/` as a structural reference for a well-formed plugin. Read `code.js` from that directory if you need to verify the pattern for screen layout, prototype wiring, or section annotations.

## Rules

- Always read the reference Figma file before writing `code.js` — never guess brand tokens
- Frame dimensions default to 390×844 unless the reference file says otherwise
- All prototype connections defined in the UX spec must be wired before the plugin closes
- The plugin must close with `figma.notify()` listing screens created
- Do NOT attempt to create Figma nodes via the Figma MCP or REST API — only the plugin approach works for design node creation
- If the UX spec has `[CONFIRM]` annotations, list them in the run instructions as items to verify after running
