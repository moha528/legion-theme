# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

An **Obsidian theme** (`name: "GitHub Theme"` in `manifest.json`) styled after the GitHub UI. There is **no build system, no package manager, no test suite** — the deliverable is just two files:

- `manifest.json` — Obsidian theme metadata (name, version, minAppVersion, author). Bump `version` on every release; existing commit convention: `chore: update theme version in the manifest.json`.
- `theme.css` — the entire theme: a single CSS file that begins with a `/* @settings ... */` YAML block and is followed by ~900 lines of CSS rules.

`imgs/` holds README screenshots only. `LICENSE` is MIT.

## Architecture: the two halves of `theme.css`

Reading just CSS rules is misleading — the file has two tightly coupled parts that must stay in sync:

1. **YAML settings block (top of file, inside `/* @settings ... */`)** — consumed by the [Style Settings](https://github.com/mgmeyers/obsidian-style-settings) Obsidian plugin. Each entry of type `class-toggle` declares an ID (e.g. `callout-on`, `h1-underline`, `colorblind_protan-deutan`, `kanban-on`) that the plugin will toggle as a class on `<body>`.
2. **CSS rules block (rest of file)** — uses those same IDs as `body.<id> ...` selectors to gate styles. Examples in the current file: `body.callout-on .callout { ... }`, `body.headers-one-color { ... }`, `body.kanban-on .kanban-plugin { ... }`, `body.colorblind_protan-deutan.theme-dark { ... }`.

**Critical invariant**: any new user-toggleable feature requires both (a) a `class-toggle` entry in the YAML block AND (b) matching `body.<id>` selectors below. The IDs must be identical character-for-character. Removing one without the other silently breaks the feature.

`variable-themed-color` entries in the YAML expose CSS variables (e.g. `--h-color-theme`, `--h1-color-theme`) with separate light/dark defaults — these are referenced by selectors under `.theme-light` / `.theme-dark` (around lines 711 and 792).

Colorblind variants (`colorblind_protan-deutan`, `colorblind_tritan`) override color variables both for `.theme-dark` and `.theme-light` separately — keep both sides updated when changing palette.

## Local development & testing

Obsidian loads themes from `<vault>/.obsidian/themes/<ThemeFolder>/`. The folder must contain `theme.css` and `manifest.json`. For iterative work, symlink the repo into a test vault so edits are picked up live:

```bash
# from inside <vault>/.obsidian/themes/
ln -s /path/to/this/repo "GitHub Theme"
```

Then in Obsidian: Settings → Appearance → reload theme (or toggle off/on) to see changes. Install the **Style Settings** plugin to exercise the YAML toggles.

There is no linter, no formatter, no CI configured — keep changes minimal and follow surrounding indentation/style.

## Release flow

1. Edit `theme.css` and/or `manifest.json`.
2. Bump `version` in `manifest.json`.
3. Commit (style: lowercase imperative, e.g. `feat: add headers underline`, `fix: ...`, `chore: update theme version in the manifest.json`).
4. Create a GitHub release whose tag matches the new `manifest.json` version — Obsidian's community theme store pulls `theme.css` and `manifest.json` from the latest release.
