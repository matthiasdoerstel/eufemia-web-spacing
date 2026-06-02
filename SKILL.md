---
name: eufemia-web-spacing
description: Use for Figma layout work in DNB/Eufemia files when creating new screens, refactoring spacing, standardizing hierarchy, or spacing nested components. Apply hierarchical spacing using the Gestalt principle of proximity so outer containers get proportionally more spacing than inner elements, binding to the Eufemia Web library's semantic spacing tokens (`page/content-width`, `page/content-padding`, `page/responsive-layout-gap-*`) — never to raw primitives or hardcoded values.
---

# Eufemia Web Spacing

Apply Gestalt-proximity hierarchy to DNB/Eufemia layouts, binding to the **actual semantic spacing tokens published by the Eufemia Web library** — not an invented scale, not locally-recreated variables, and never the resolved primitive value.

> **Cardinal rule: bind the semantic token, not the value.** Eufemia's spacing chain is `semantic token → size primitive → number` (e.g. `page/responsive-layout-gap-sm` → `size/8` → `8`). Always bind the *semantic* token. Binding the `size/*` primitive — or worse, a hardcoded number — throws away the intent and won't follow the system. If a lookup resolves a token through an alias chain, bind the variable a designer would *name*, not the alias target you landed on.

> 🏷️ **Naming your layers well makes the output noticeably better.** This skill infers each element's role (page, content area, section, card, field, label, input) partly from layer names. Clear, descriptive names like `Content Area`, `Section: Profile`, `Field`, or `Input` let it map the right token to the right level. Generic names (`Frame 1`, `Group 2`) force it to guess from structure alone, and misleading names can lead to the wrong spacing. **Name your layers for what they are, and you'll get a more accurate hierarchy.**

## Onboarding (first run & `help`)
**Do this before anything else, every invocation:**

1. **Show the onboarding if needed.** Show the orientation below when **either**:
   - the user invoked the skill with `help`, `--help`, `?`, or `onboard`, **or**
   - the first-run marker file is missing. Check it with: `test -f ~/.claude/.eufemia-web-spacing-onboarded`.
2. **After showing it on a first run, write the marker** so it won't auto-show again: `touch ~/.claude/.eufemia-web-spacing-onboarded`. (Don't write the marker when the user merely asked for `help` — only on the automatic first run.)
3. If the marker exists and the user didn't ask for help, **skip onboarding** and go straight to the work.

**Orientation to show:**

> 👋 **Eufemia Web Spacing** — applies an intentional, responsive spacing hierarchy to your Figma *page layout* using the Eufemia Web design tokens.
>
> **What it does:** reads your layout's nesting (page → content area → section → field → label/input) and binds Eufemia spacing tokens so related things sit close and groups stay distinct — the page frame gets `content-width` + `content-padding`, everything inside gets the responsive `layout-gap` scale.
>
> **How to use it:** select the page/layout frame you want to space, then run the skill. It works best when the Eufemia Web library is available in your file and your **layers are named** for what they are (`Content Area`, `Section`, `Field`, `Input`).
>
> **What to expect:** it walks the spacing scale one step per nesting level (8 → 16 → 24 → 32 → 48 on desktop, compressing on mobile), binds real library tokens (never hardcoded numbers), and screenshots the result to verify the hierarchy reads.
>
> **Good to know:** it's a *layout* tool — pages full of components are fine; it spaces the frames *around* them. It will **not** edit a component's internals (that's testing-only and it'll warn you), and it'll **ask** before touching any Figma Slots.
>
> Run `/eufemia-web-spacing help` anytime to see this again.

## Auto-update (throttled daily check)
The skill keeps itself current from its GitHub repo:

```
https://github.com/matthiasdoerstel/eufemia-web-spacing
```

**Run this gate near the start of every invocation**, using the **skill's own directory** (the "Base directory for this skill" shown when the skill loads) as `$DIR` and the repo above as `$REPO`.

1. **Decide whether to check.** Force a check if the user invoked `update` (manual). Otherwise throttle to once per day: read `~/.claude/.eufemia-web-spacing-update-check`; if its contents equal today's date (`date +%F`), **skip the check entirely** and proceed to the work.
2. **Check (only if not throttled).** The skill dir must be a git clone; if `$DIR/.git` doesn't exist, skip silently (copy-paste install — nothing to update against). Otherwise fetch the **pinned repo explicitly** (don't rely on whatever `origin` happens to point at):
   ```bash
   REPO="https://github.com/matthiasdoerstel/eufemia-web-spacing.git"
   git -C "$DIR" fetch --quiet "$REPO" main 2>/dev/null
   LOCAL=$(git -C "$DIR" rev-parse HEAD)
   REMOTE=$(git -C "$DIR" rev-parse FETCH_HEAD)
   echo "$(date +%F)" > ~/.claude/.eufemia-web-spacing-update-check   # record the check
   ```
3. **If `LOCAL` != `REMOTE`, notify and offer — do NOT pull silently.** Show the remote version (`git -C "$DIR" show FETCH_HEAD:VERSION`) and a one-line summary, then ask if they want to update now. On confirmation:
   ```bash
   git -C "$DIR" pull --ff-only "$REPO" main
   ```
   - `--ff-only` is deliberate: if it fails, the local copy has diverged (local edits). Don't force — tell the user and let them resolve.
   - The pull rewrites this SKILL.md, so the **new version takes effect on the next invocation**, not the current run. Say so.
4. If `LOCAL` == `REMOTE`, say nothing (or, on a manual `update`, confirm it's already current).

Keep the check quiet and fast — never block the actual spacing work on it, and never auto-pull without the user's OK.

## When to use
- Creating new screens or layouts in a DNB/Eufemia Figma file that need intentional, consistent spacing communicating visual hierarchy.
- Refactoring spacing in an existing Eufemia design to improve readability and grouping.
- Building components that nest inside larger containers and need spacing that scales with depth.
- Standardizing spacing across a file to align with the Gestalt principle of proximity — related items close together, separate groups further apart.

## Before you apply — required checks
Run these against the current selection **before binding any spacing**. Both checks pause the workflow.

### 1. Don't apply layout tokens *to a component* — testing only
This skill applies **layout** spacing to **layout containers** (the page, sections, cards, field wrappers — the frames that arrange content). A page that *contains* Eufemia component instances (buttons, inputs, etc.) is completely normal and **expected** — that's exactly what layout tokens are for. You space the layout frames *around* the instances; you never touch the instances' internals. **The mere presence of components on a page is not a problem.**

The problem is only when the skill is pointed **at a component itself** — i.e. the request is to apply layout spacing *to* a Eufemia component or its internal structure. Trigger the warning only when:
- the selected/target node **is** a `COMPONENT` or `COMPONENT_SET` (a main component, not an instance sitting in a layout), **or**
- the user explicitly asks to apply the skill **to a Eufemia component** (e.g. "space the internals of this Button"), **or**
- you'd be binding tokens **inside** a component's or instance's definition rather than on the surrounding layout.

In those cases, **STOP and surface this warning clearly:**

> ⚠️ **This skill is NOT intended to space the internals of Eufemia components.**
> It applies *layout* tokens to *page layout*. A component's internal spacing is owned by the design system — overriding it affects every product that consumes the component. Doing this is permitted **for testing purposes ONLY**.

Only continue after the user **explicitly confirms it is for testing**. Otherwise, redirect to the surrounding layout frames (or a local copy) instead.

If the selection is a layout frame that simply *holds* component instances → **no warning, proceed normally**: bind tokens to the layout frames and leave every instance's internals untouched.

### 2. Figma Slots — ask before including them
Check the selection and its subtree for **Figma Slot nodes** (`node.type === 'SLOT'`). Slots are a newer Figma feature; if the running API doesn't expose the type, skip this check silently.

If any slots are found, **ask the user** whether the spacing hierarchy should also be applied to the slots (and their contents), and only include them if they say yes. Treat each slot's content as its own nesting branch when applying the proximity walk.

Quick detection script:
```js
const sel = figma.currentPage.selection;
const slots = [];
// Only flag components that ARE the target — NOT instances living inside a layout.
const targetIsComponent = sel.some(n => n.type === "COMPONENT" || n.type === "COMPONENT_SET");
for (const n of sel) {
  if (n.type === "SLOT") slots.push(n.id);
  const found = n.findAll ? n.findAll(d => d.type === "SLOT") : [];
  slots.push(...found.map(s => s.id));
}
// targetIsComponent → fire the testing-only warning (check 1).
// Instances inside the layout are fine and intentionally ignored here.
return { targetIsComponent, slots };
```

## The Eufemia Web spacing tokens
Source library: **💻 Eufemia - Web**, file key `cdtwQD8IJ7pTeE45U148r1`. Spacing lives in the **`spacing`** collection (modes: `desktop` / `tablet` / `mobile`). Bind these — do **not** bind the underlying `size/*` primitives.

### Page-frame tokens (the outer shell)
| Token | Binds to | Desktop | Mobile | Key |
|---|---|---|---|---|
| `page/content-width` | frame **width** | 1128 | 375 | `7abe66ef049f8b64397120c16b76f335c8c95c60` |
| `page/content-padding` | page **padding** (all sides) | 96 | 16 | `9015f8ac9dac3e16ca9bb024ffcd92e7c281415c` |

Apply `content-width` to the outermost page frame's width **always, regardless of its previous width**, and `content-padding` to that frame's padding. Everything *inside* uses the gap scale below.

### Internal rhythm — responsive gap scale
Everything inside the page shell binds the **`page/responsive-layout-gap-*`** tokens: desktop/tablet hold, mobile compresses one step. All internal spacing is responsive — there is no constant variant in this skill.

| Step | px (desktop→mobile) | `responsive-layout-gap-*` key |
|---|---|---|
| xs | 4→2 | `031d7c9f4bb2f11d2adaf02db9edd9a4ba6efb3c` |
| sm | 8→4 | `fa2b411b43ec509319db986b8a3d7bd98c994b2a` |
| me | 16→8 | `4cb6fa0910d0cf43c7b6be0270d87456502d0c66` |
| la | 24→16 | `05723ee53f0cfd29a0e22871d104604e2e9952cf` |
| xl | 32→24 | `36022c0edcbd39e65d797df0fee2b174bccc0d5f` |
| 2xl | 40→32 | `905bf4c076b386bc8730ac8b8ab981d701235137` |
| 3xl | 48→40 | `880e617ada2fddc11c5f4c05a60f695ecb2e2b85` |

There is no 12px (or any non-token value) in this system — the scale is 4/8/16/24/32/40/48.

### Discovery & gotchas
- The semantic tokens are **aliases into a `size` primitive collection** (`page/responsive-layout-gap-sm` → `size/8`). The `size/*` primitives are the resolved values — **never bind them**; bind the named `page/...-gap-*` token.
- `get_variable_defs` on a node returns `{}` when the node has **no bound variables** — it does **not** list what's available in the file. Don't conclude "no tokens exist" from an empty result.
- `getLocalVariableCollectionsAsync` on the library file returns only `brand` + `spacing`; the `size` collection isn't listed, but its variables still resolve via alias and import fine by key.
- `teamLibrary.getAvailableLibraryVariableCollectionsAsync()` only lists libraries **enabled** in the target file. `importVariableByKeyAsync(key)` works regardless — it doesn't require the library to be enabled.
- Import → bind pattern: `const v = await figma.variables.importVariableByKeyAsync(key); node.setBoundVariable("itemSpacing", v);`

## Instructions
0. **Onboarding gate** (see "Onboarding" above): show the orientation on first run (or on `help`), write the marker on first run, then continue.
0a. **Auto-update gate** (see "Auto-update" above): run the throttled daily check (or forced check on `update`); notify and offer to update if behind, then continue.
0b. **Run the pre-flight checks** (see "Before you apply" above): warn **only if the skill is being pointed at a component itself** (testing only, explicit confirmation) — component *instances* sitting inside the layout are normal, don't warn — and ask about Figma Slots. Don't bind anything until these are cleared.
1. **Map the nesting depth** in a single read from outermost to innermost (e.g. page → content area → section/card → field → label/input), using layer names to identify each element's role. If names are generic or missing, fall back to structural depth — and tell the designer that naming layers will improve the result. If slots were approved in the pre-flight, treat each slot's content as its own branch.
2. **Plan the token-per-level assignment** (no Figma roundtrip — pure reasoning) using the proximity walk, one step up the responsive gap scale per level, innermost → outermost:
   - label ↔ input → `responsive-layout-gap-sm`
   - field ↔ field & section-title ↔ fields → `responsive-layout-gap-me`
   - section padding & between-sections → `responsive-layout-gap-la`
   - content-area padding → `responsive-layout-gap-xl`
   - page-internal gap (title ↔ content) → `responsive-layout-gap-3xl`
   - If two adjacent levels would collide on the same step, bump the outer one up a step so each level is perceptibly different.
3. **Apply everything in ONE `use_figma` script** — this is the speed-critical step. Don't split discovery, import, and binding into separate tool calls; each call is a roundtrip. In a single execution:
   - **Import only the keys you actually need, in parallel** with `Promise.all` (never serially, never one tool call per key). Pattern:
     ```js
     const KEYS = { sm:"…", me:"…", la:"…", xl:"…", "3xl":"…", contentWidth:"…", contentPadding:"…" };
     const pairs = await Promise.all(
       Object.entries(KEYS).map(async ([n, k]) => [n, await figma.variables.importVariableByKeyAsync(k)])
     );
     const v = Object.fromEntries(pairs); // v.sm, v.contentWidth, …
     ```
     Never create local mirror variables and never bind `size/*` primitives.
   - **Bind the page shell:** outermost frame `width` → `v.contentWidth`, all four paddings → `v.contentPadding`. When widening to content-width, set the child chain `layoutSizingHorizontal = "FILL"` so content stretches cleanly.
   - **Bind the internal hierarchy** from the plan in step 2, in the same pass:
     - Padding → `setBoundVariable("paddingTop"|"paddingRight"|"paddingBottom"|"paddingLeft", token)` on auto-layout frames (maps to `Section`/`Card`/`Space` in code).
     - Gap → `setBoundVariable("itemSpacing", token)` on auto-layout frames (maps to `Flex.Stack`/`Flex.Container` gaps in code).
   - **Return the `boundVariables` readback** for every touched node in the same script's return value, so validation needs no extra roundtrip.
4. **Validate.** Use the readback returned by step 3 to confirm each property points at the *named semantic token* (not a `size/*` primitive or null). Then screenshot once to check the proximity hierarchy reads: outer groups clearly separated, inner items cohesive. Note: the REST screenshot resolves the collection's default (desktop) mode, while the plugin runtime may report another mode — a width of 1128 vs 375 just reflects which mode is active, not a bug.
5. **Summarize** the token bound at each level (token name + px).

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
- **Page is full of component instances:** That's fine and expected — apply layout tokens to the surrounding layout frames and leave the instances' internals alone. The only thing that triggers the testing-only warning is pointing the skill *at a component itself* (see check 1).
- **Tempted to bind a number:** Stop. Find the semantic token. If you only have a resolved value, that's a sign you grabbed the alias target (`size/*`) instead of the token — go back up the chain.
- **Adjacent levels collide on the same step:** bump the outer level up one step. Never split the difference with a non-token value.
- **Borders / divider lines (1–2px):** keep the layout on grid by subtracting the line thickness from the neighbouring element's padding rather than adding an off-grid value.
- **Deeply nested (5+ levels):** the scale tops at 3xl (48); cap the outer levels there and compress inner levels while keeping the relative progression.
- **Existing inconsistent / off-grid spacing:** audit current values, flag every non-token value, then remap from the innermost level outward — don't nudge individual values in isolation.
