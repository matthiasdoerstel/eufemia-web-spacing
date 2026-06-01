---
name: eufemia-web-spacing
description: Use for Figma layout work in DNB/Eufemia files when creating new screens, refactoring spacing, standardizing hierarchy, or spacing nested components. Apply hierarchical spacing using the Gestalt principle of proximity so outer containers get proportionally more spacing than inner elements, binding to the Eufemia Web library's semantic spacing tokens (`page/content-width`, `page/content-padding`, `page/layout-gap-*`, `page/responsive-layout-gap-*`) — never to raw primitives or hardcoded values.
---

# Eufemia Web Spacing

Apply Gestalt-proximity hierarchy to DNB/Eufemia layouts, binding to the **actual semantic spacing tokens published by the Eufemia Web library** — not an invented scale, not locally-recreated variables, and never the resolved primitive value.

> **Cardinal rule: bind the semantic token, not the value.** Eufemia's spacing chain is `semantic token → size primitive → number` (e.g. `page/responsive-layout-gap-sm` → `size/8` → `8`). Always bind the *semantic* token. Binding the `size/*` primitive — or worse, a hardcoded number — throws away the intent and won't follow the system. If a lookup resolves a token through an alias chain, bind the variable a designer would *name*, not the alias target you landed on.

> 🏷️ **Naming your layers well makes the output noticeably better.** This skill infers each element's role (page, content area, section, card, field, label, input) partly from layer names. Clear, descriptive names like `Content Area`, `Section: Profile`, `Field`, or `Input` let it map the right token to the right level. Generic names (`Frame 1`, `Group 2`) force it to guess from structure alone, and misleading names can lead to the wrong spacing. **Name your layers for what they are, and you'll get a more accurate hierarchy.**

## When to use
- Creating new screens or layouts in a DNB/Eufemia Figma file that need intentional, consistent spacing communicating visual hierarchy.
- Refactoring spacing in an existing Eufemia design to improve readability and grouping.
- Building components that nest inside larger containers and need spacing that scales with depth.
- Standardizing spacing across a file to align with the Gestalt principle of proximity — related items close together, separate groups further apart.

## Before you apply — required checks
Run these against the current selection **before binding any spacing**. Both checks pause the workflow.

### 1. Official library components — warn loudly, testing only
Detect whether the selection **is, or contains, an official Eufemia library component**:
- any `COMPONENT` or `COMPONENT_SET` node, **or**
- any `INSTANCE` whose `mainComponent.remote === true` (an instance of a published library component), **or**
- you are working inside the Eufemia Web library file itself (file key `cdtwQD8IJ7pTeE45U148r1`).

If so, **STOP and surface this warning clearly before doing anything else:**

> ⚠️ **This skill is NOT intended for official Eufemia library components.**
> Changing the spacing of a shared component affects **every product that consumes it**. This is permitted **for testing purposes ONLY** — never as a real change to the official library.

Only continue after the user **explicitly confirms it is for testing**. Otherwise stop and suggest they apply the skill to a local copy or a plain frame instead.

### 2. Figma Slots — ask before including them
Check the selection and its subtree for **Figma Slot nodes** (`node.type === 'SLOT'`). Slots are a newer Figma feature; if the running API doesn't expose the type, skip this check silently.

If any slots are found, **ask the user** whether the spacing hierarchy should also be applied to the slots (and their contents), and only include them if they say yes. Treat each slot's content as its own nesting branch when applying the proximity walk.

Quick detection script:
```js
const sel = figma.currentPage.selection;
const slots = [];
const libComponents = [];
for (const n of sel) {
  if (n.type === "COMPONENT" || n.type === "COMPONENT_SET") libComponents.push(n.id);
  if (n.type === "INSTANCE") {
    const mc = await n.getMainComponentAsync();
    if (mc && mc.remote) libComponents.push(n.id);
  }
  const found = n.findAll ? n.findAll(d => d.type === "SLOT") : [];
  if (n.type === "SLOT") slots.push(n.id);
  slots.push(...found.map(s => s.id));
}
return { libComponents, slots };
```

## The Eufemia Web spacing tokens
Source library: **💻 Eufemia - Web**, file key `cdtwQD8IJ7pTeE45U148r1`. Spacing lives in the **`spacing`** collection (modes: `desktop` / `tablet` / `mobile`). Bind these — do **not** bind the underlying `size/*` primitives.

### Page-frame tokens (the outer shell)
| Token | Binds to | Desktop | Mobile | Key |
|---|---|---|---|---|
| `page/content-width` | frame **width** | 1128 | 375 | `7abe66ef049f8b64397120c16b76f335c8c95c60` |
| `page/content-padding` | page **padding** (all sides) | 96 | 16 | `9015f8ac9dac3e16ca9bb024ffcd92e7c281415c` |

Apply `content-width` to the outermost page frame's width **always, regardless of its previous width**, and `content-padding` to that frame's padding. Everything *inside* uses the gap scale below.

### Internal rhythm — gap scale (two flavours)
| Step | px (constant / responsive→mobile) | `layout-gap-*` key (constant) | `responsive-layout-gap-*` key |
|---|---|---|---|
| xs | 4 / 4→2 | `d235e127119d9c4c33d4e31dfe2bce71c71d52b4` | `031d7c9f4bb2f11d2adaf02db9edd9a4ba6efb3c` |
| sm | 8 / 8→4 | `b204bbba1da9d2e2b6bb451a674cb51ee22d9d1c` | `fa2b411b43ec509319db986b8a3d7bd98c994b2a` |
| me | 16 / 16→8 | `713a78d1219922711995485aa28fe6127eb05cc7` | `4cb6fa0910d0cf43c7b6be0270d87456502d0c66` |
| la | 24 / 24→16 | `5fc6dbe7ebb5c37ac9fcae9b2434b55d34e1d355` | `05723ee53f0cfd29a0e22871d104604e2e9952cf` |
| xl | 32 / 32→24 | `14134a4b65c738308070b28923f95a2f31f4241c` | `36022c0edcbd39e65d797df0fee2b174bccc0d5f` |
| 2xl | 40 / 40→32 | `88ffc90b1f8e7c4e89d02cf47b8e6300a5bb208b` | `905bf4c076b386bc8730ac8b8ab981d701235137` |
| 3xl | 48 / 48→40 | `c7e77ef9a84120a401081ba8b2ec19a5b9403e38` | `880e617ada2fddc11c5f4c05a60f695ecb2e2b85` |

**Which flavour to use:**
- **`page/responsive-layout-gap-*`** — desktop/tablet hold, mobile compresses one step. **Default** for normal page/section/field layout that should breathe less on small screens.
- **`page/layout-gap-*`** — identical across all modes. Use **only** when spacing must stay constant regardless of breakpoint.

There is no 12px (or any non-token value) in this system — the scale is 4/8/16/24/32/40/48.

### Discovery & gotchas
- The semantic tokens are **aliases into a `size` primitive collection** (`page/responsive-layout-gap-sm` → `size/8`). The `size/*` primitives are the resolved values — **never bind them**; bind the named `page/...-gap-*` token.
- `get_variable_defs` on a node returns `{}` when the node has **no bound variables** — it does **not** list what's available in the file. Don't conclude "no tokens exist" from an empty result.
- `getLocalVariableCollectionsAsync` on the library file returns only `brand` + `spacing`; the `size` collection isn't listed, but its variables still resolve via alias and import fine by key.
- `teamLibrary.getAvailableLibraryVariableCollectionsAsync()` only lists libraries **enabled** in the target file. `importVariableByKeyAsync(key)` works regardless — it doesn't require the library to be enabled.
- Import → bind pattern: `const v = await figma.variables.importVariableByKeyAsync(key); node.setBoundVariable("itemSpacing", v);`

## Instructions
0. **Run the pre-flight checks** (see "Before you apply" above): warn on official library components (testing only, explicit confirmation required) and ask about Figma Slots. Don't bind anything until these are cleared.
1. **Map the nesting depth** from outermost to innermost (e.g. page → content area → section/card → field → label/input), using layer names to identify each element's role. If names are generic or missing, fall back to structural depth — and tell the designer that naming layers will improve the result. If slots were approved in the pre-flight, treat each slot's content as its own branch.
2. **Discover the spacing tokens.** Check whether the Eufemia Web library is enabled in the file (`teamLibrary.getAvailableLibraryVariableCollectionsAsync`). If the tokens aren't already available, **import them by key** from the Eufemia Web library with `figma.variables.importVariableByKeyAsync(key)` (keys in the tables above). **Never create local mirror variables and never bind `size/*` primitives.**
3. **Bind the page-frame shell:** outermost frame `width` → `content-width`, `padding` (all four sides) → `content-padding`.
4. **Assign the internal hierarchy** using the proximity walk — one step up the gap scale per level, innermost → outermost, defaulting to the **responsive** flavour:
   - label ↔ input → `responsive-layout-gap-sm`
   - field ↔ field & section-title ↔ fields → `responsive-layout-gap-me`
   - section padding & between-sections → `responsive-layout-gap-la`
   - content-area padding → `responsive-layout-gap-xl`
   - page-internal gap (title ↔ content) → `responsive-layout-gap-3xl`
   - If two adjacent levels would collide on the same step, bump the outer one up a step so each level is perceptibly different.
5. **Apply via `use_figma`:**
   - Padding → `setBoundVariable("paddingTop"|"paddingRight"|"paddingBottom"|"paddingLeft", token)` on auto-layout frames (maps to `Section`/`Card`/`Space` in code).
   - Gap → `setBoundVariable("itemSpacing", token)` on auto-layout frames (maps to `Flex.Stack`/`Flex.Container` gaps in code).
   - Width → `setBoundVariable("width", contentWidthToken)`.
   - When widening a frame to `content-width`, set the child chain to `layoutSizingHorizontal = "FILL"` so content stretches cleanly.
6. **Validate.** Read back `node.boundVariables` to confirm each property points at the *named semantic token* (not a `size/*` primitive or null). Screenshot to check the proximity hierarchy reads: outer groups clearly separated, inner items cohesive. Note: the REST screenshot resolves the collection's default (desktop) mode, while the plugin runtime may report another mode — a width of 1128 vs 375 just reflects which mode is active, not a bug.
7. **Summarize** the token bound at each level (token name + px), and flag anything that had to fall back (e.g. a constant `layout-gap` where responsive wasn't wanted).

## Example

**Request:** "A settings page with content area containing form sections."

| Level | Element | Token | Desktop px |
|-------|---------|-------|------------|
| page frame | width / outer padding | `content-width` / `content-padding` | 1128 / 96 |
| page internal | title ↔ content | `responsive-layout-gap-3xl` | 48 |
| content area | padding | `responsive-layout-gap-xl` | 32 |
| section | padding & between sections | `responsive-layout-gap-la` | 24 |
| field group | field ↔ field, title ↔ fields | `responsive-layout-gap-me` | 16 |
| innermost | label ↔ input | `responsive-layout-gap-sm` | 8 |

**Result:** tightest spacing (sm, 8px) groups labels with inputs, me (16) separates fields, la (24) distinguishes sections, xl (32) frames the content area, and the page shell uses `content-width` + `content-padding`. Because everything binds the responsive tokens, the whole layout compresses correctly in mobile mode, and it hands off cleanly to `Flex.Stack`, `Section`, and `Card` in code.

## Common edge cases
- **Tokens not enabled in the file:** Import them by key from the Eufemia Web library (`importVariableByKeyAsync`) — this works even if the library isn't enabled in the file. Do **not** recreate them locally.
- **Tempted to bind a number:** Stop. Find the semantic token. If you only have a resolved value, that's a sign you grabbed the alias target (`size/*`) instead of the token — go back up the chain.
- **Spacing must not change on mobile** (e.g. an icon-to-label gap inside a control): use the constant `page/layout-gap-*` flavour instead of responsive.
- **Adjacent levels collide on the same step:** bump the outer level up one step. Never split the difference with a non-token value.
- **Borders / divider lines (1–2px):** keep the layout on grid by subtracting the line thickness from the neighbouring element's padding rather than adding an off-grid value.
- **Deeply nested (5+ levels):** the scale tops at 3xl (48); cap the outer levels there and compress inner levels while keeping the relative progression.
- **Existing inconsistent / off-grid spacing:** audit current values, flag every non-token value, then remap from the innermost level outward — don't nudge individual values in isolation.
