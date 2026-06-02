# Changelog

All notable changes to **eufemia-web-spacing**. The skill reads the entry for
the latest version on the `stable` channel and shows it when offering an update.

## 1.4.0

- **Update mechanism reworked for safety.** The skill now tracks a dedicated
  **release channel (`stable` branch)** instead of `main`, so in-progress
  experiments never reach installs.
- **Detect is now separated from apply.** The daily check is a read-only HTTPS
  read of the channel's `VERSION` (and `CHANGELOG.md`) — no `git fetch`, nothing
  privileged, and it works for copy-paste installs too. A `git pull --ff-only`
  runs only after an explicit yes.
- **Changelog shown on update.** Designers see *what* changed, not just *that*
  something changed, before consenting.
- **State files relocated** from `~/.claude/...` to a co-located, gitignored
  `.state/` directory — no assumption about `~/.claude` vs `~/.raiwork`, and
  untouched by fast-forward updates.
- Install docs updated to clone the `stable` channel and cover both Raiwork and
  Raicode skill paths.

## 1.3.1

- Baseline: Gestalt-proximity spacing hierarchy bound to Eufemia Web semantic
  tokens; component-internals and Figma Slot guards; single-pass read + apply.
