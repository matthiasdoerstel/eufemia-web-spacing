# eufemia-web-spacing

A **Raicode** skill that helps **DNB designers apply the Eufemia spacing system** to their Figma layouts.

It uses the Gestalt principle of proximity — related items close together, separate groups further apart — and binds every value to the **Eufemia Web library's semantic spacing tokens** (`page/content-width`, `page/content-padding`, `page/responsive-layout-gap-*`). It never binds raw `size/*` primitives or hardcoded numbers.

## What it does

- Maps a design's nesting depth (page → content area → section → field → label/input) and assigns one step up the spacing scale per level, so hierarchy reads without dividers.
- Binds the page frame's **width** to `content-width` and its **padding** to `content-padding`.
- Binds every internal level to the **responsive** gap tokens (`page/responsive-layout-gap-*`), which hold on desktop/tablet and compress one step on mobile.
- Imports the tokens straight from the Eufemia Web library by key — works even if the library isn't enabled in the file.

## Install

Clone into your Raicode skills directory:

```bash
git clone https://github.com/matthiasdoerstel/eufemia-web-spacing.git ~/.claude/skills/eufemia-web-spacing
```

Then invoke it in Raicode with `/eufemia-web-spacing` (or let it trigger automatically on Eufemia layout/spacing work). The first run shows a short onboarding; run `/eufemia-web-spacing help` anytime to see it again.

### Staying up to date
The skill checks this repo for updates once a day (only when you run it). If a newer version exists, it tells you and offers to update — it never pulls silently. You can also force a check with `/eufemia-web-spacing update`. Auto-update requires the git clone above (a copy-paste install won't self-update). Local edits are preserved: updates use `--ff-only`, so if your copy has diverged it'll stop and let you resolve it.

## Requirements

- Raicode (CLI) or Raiwork (desktop) with the **official remote Figma MCP server** (`use_figma`, `get_metadata`; `get_screenshot` only for optional visual checks). It does **not** use `figma-console`, `pencil`, or any other Figma integration.
- Access to the **💻 Eufemia - Web** Figma library.

## The token model

The skill's golden rule: **bind the semantic token, not the resolved value.** Eufemia's chain is `semantic token → size primitive → number` (e.g. `page/responsive-layout-gap-sm` → `size/8` → `8`). Always bind the named `page/...` token so the design follows the system.

See [`SKILL.md`](./SKILL.md) for the full token tables (with import keys), the responsive gap scale, discovery gotchas, and worked examples.

## License

MIT
