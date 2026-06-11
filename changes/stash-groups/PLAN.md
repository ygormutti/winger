# MVP Implementation Plan for Stashing Groups & Unstash Here

This document outlines the high-level implementation plan for the Phase 1 features of Tab Group support in Winger, as detailed in `RESEARCH.md`. The goal is to introduce:
1. **Stash Groups as Folders**: An opt-in setting allowing windows with tab groups to be stashed as separate bookmark folders (one for each group, plus one for ungrouped tabs).
2. **Unstash Here**: A new action to unstash a folder's bookmarks directly into the currently active window, bypassing the creation of a new window and relying on the existing restore algorithm for native grouping.

## Task Breakdown

The work has been categorized into four distinct areas, detailed in their respective task markdown files within the `changes/stash-groups/tasks/` directory. These tasks are designed to be completed by a junior programmer or autonomous agent.

### 1. Settings (`tasks/Settings.md`)
- Add the `stash_groups_as_folders` setting to the extension's default storage and settings UI (in `storage.js` and `page/settings.html`).

### 2. UI (`tasks/UI.md`)
- Create a new "Unstash Here" button alongside the standard "Unstash" button in the popup row templates (`popup/popup.html`, `popup/row.js`).
- Add the corresponding SVG icon (`icons/unstash-here.svg`).
- Ensure the button is properly hidden/shown for stashed items in `popup/popup.css`.

### 3. Stash & Unstash Logic (`tasks/Stash.md`)
- **Stash**: Modify `stashWindow` and `runStashTasks` in `background/stash.stash.js` to read the new setting. If true, partition tabs by group and stash them into separate folders in the Stash Home.
- **Unstash Here**: Refactor the existing unstash logic (`background/stash.unstash.js`) to support unstashing directly to the active window. Introduce a `toNewWindow` parameter (defaulting to `true`) and allow the popup request to pass `false` when triggered by the new button. Route the popup UI actions to these background scripts.

### 4. Documentation & Testing (`tasks/DocsAndTesting.md`)
- As Winger has no automated test suite, create clear manual verification instructions to test the new functionality.
- Update `README.md` or any relevant help documentation to describe the new settings and buttons.

## Important Notes for Implementation
- Ensure there are no slash commands or shorthands created for the new "Unstash here" functionality (unlike the standard unstash which doesn't have them either).
- Do not touch Phase 2 requirements (e.g. nested tab group UI in popup). Edge cases regarding duplicate groups/tabs on unstash are acceptable for this MVP.
