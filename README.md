# eufemia-web-spacing

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill for applying intentional, hierarchical spacing to **DNB / Eufemia** Figma layouts.

It uses the Gestalt principle of proximity — related items close together, separate groups further apart — and binds every value to the **Eufemia Web library's semantic spacing tokens** (`page/content-width`, `page/content-padding`, `page/layout-gap-*`, `page/responsive-layout-gap-*`). It never binds raw `size/*` primitives or hardcoded numbers.

## What it does

- Maps a design's nesting depth (page → content area → section → field → label/input) and assigns one step up the spacing scale per level, so hierarchy reads without dividers.
- Binds the page frame's **width** to `content-width` and its **padding** to `content-padding`.
- Defaults to the **responsive** gap tokens (which compress on mobile) and falls back to the constant `layout-gap` tokens only when spacing must stay identical across breakpoints.
- Imports the tokens straight from the Eufemia Web library by key — works even if the library isn't enabled in the file.

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/matthiasdoerstel/eufemia-web-spacing.git ~/.claude/skills/eufemia-web-spacing
```

Then invoke it in Claude Code with `/eufemia-web-spacing` (or let it trigger automatically on Eufemia layout/spacing work).

## Requirements

- Claude Code with the Figma MCP server (`use_figma`, `get_screenshot`, `get_metadata`).
- Access to the **💻 Eufemia - Web** Figma library (file key `cdtwQD8IJ7pTeE45U148r1`).

## The token model

The skill's golden rule: **bind the semantic token, not the resolved value.** Eufemia's chain is `semantic token → size primitive → number` (e.g. `page/responsive-layout-gap-sm` → `size/8` → `8`). Always bind the named `page/...` token so the design follows the system.

See [`SKILL.md`](./SKILL.md) for the full token tables (with import keys), the layout-gap vs. responsive-layout-gap rule, discovery gotchas, and worked examples.

## License

MIT
