# Tab Groups Support in Winger - Research Document

## Overview and Vision

With the introduction of the native `tabGroups` API in Firefox (v137+), Winger can be extended to manage not just windows, but also Tab Groups natively.

The long-term vision is to allow Tab Groups to be first-class citizens in Winger's popup UI:
- Nested under windows, represented as collapsible containers.
- Navigable using the keyboard.
- Supporting the same quick actions as windows ("Send", "Bring", "Stash").
- Supporting moving tabs between groups.

This research document details the architecture and APIs necessary to build this vision, split into two phases: the **Current Scope** (Stashing/Unstashing groups) and **Future Research** (Popup UI integration and tab movements).

---

## Phase 1: Research (Current Scope)

### Goal
1. Allow the user to stash an entire Tab Group as a separate, top-level bookmark folder inside the Stash Home (instead of just burying the group info in the bookmark titles under a window stash).
2. Allow the user to unstash any stashed folder directly into a Tab Group in the currently active window, rather than opening a new window.
3. This should be triggered by a new UI button in the panel and/or an omnibox command.

### 1. Stashing a Window's Groups as Separate Folders

**Current Behavior:**
When a window is stashed, Winger creates a single bookmark folder named after the window. The window's properties (like `{"private": true}`) are appended to the title. Each tab in the window becomes a bookmark within this single folder. The tab's properties (including its `group` info) are appended to the bookmark's title.

**Proposed Changes:**
Since Tab Groups are not yet surfaced as addressable rows in the UI, we cannot directly stash a *single* group. Instead, we introduce an experimental, opt-in setting (e.g., `stash_groups_as_folders`) that alters how *windows* are stashed.

- **Stash Mechanics**: When stashing a window (`stashWindow` in `background/stash.stash.js`), if the setting is enabled, Winger will iterate through the tabs and identify distinct Tab Groups. For each group found, it will create a *separate*, top-level bookmark folder in the Stash Home, rather than nesting them all under one window folder. Ungrouped tabs can either go into a "misc" folder or be handled according to user preference.
- **Properties**: We extend `StashProp.Window` in `background/stash.prop.js` (or create a new `StashProp.Group`) to encode group properties into the folder title for these group-specific folders. The JSON annotation on the folder should look something like: `{"group": {"id": 1, "color": "blue", "title": "My Group"}}`.

### 2. Unstashing a Folder as a Group

**Current Behavior:**
Right now, `unstashFolder` in `background/stash.unstash.js` reads a folder, creates a new Window (using any encoded `ProtoWindow` properties), and opens the bookmarks in that new window.

**Proposed Changes:**
- **Trigger**: The user clicks a new "Unstash to Group" icon button on a stashed folder row, or uses a modified command (e.g., `/unstashgroup` or a shift-modifier combination).
- **Unstash Mechanics**: We modify `unstashFolder` (or add `unstashToGroup`) to:
  1. Retrieve the tabs in the bookmark folder.
  2. Open these tabs in the *current* window (instead of calling `browser.windows.create()`).
  3. Group the newly created tab IDs using `browser.tabs.group()`.
  4. If the folder itself had group properties encoded in its title (e.g. color, title), we apply those to the new group using `browser.tabGroups.update()`.

### 3. API & Extension Requirements

- **Permissions**: Winger already declares the `"tabGroups"` permission in `manifest.json`.
- **Commands & Shorthands**:
  - Add a `/stashgroup` (`/sg`) command in `popup/omnibox.js` to stash the currently active group (if the user is focused on a tab that belongs to a group).
  - Add an "Unstash to Group" action button (with a new icon, e.g., `icons/unstash-group.svg`) in the popup UI for stashed rows (`Template.$folder` in `popup/row.js`).
- **Compatibility**: Ensure fallback behaviors if the user's Firefox version does not support tabGroups (though the user should be on v137+).

---

## Phase 2: Future Research (Handoff)

### Goal
Integrate Tab Groups fully into the popup UI, making them visually nested under windows, and allowing tab movements (Send/Bring) across groups.

### 1. Surfacing Groups in the Popup UI

- **Data Retrieval**: `browser.tabGroups.query({ windowId: winfo.id })` can fetch all groups for a given window.
- **UI Adjustments (`popup/row.js`)**:
  - Windows that contain groups should act as collapsible containers.
  - Groups should be rendered as a new custom element type (e.g., `<group-row>`) or a heavily styled `<window-row>` that appears indented.
  - The group rows need to display the group's `title`, `color` (perhaps as a colored dot icon), and the count of tabs within the group.
  - To support expand/collapse, we can toggle a `collapsed` CSS class on the parent window row and hide/show the child group rows.

### 2. Keyboard Navigation Updates

- **Current Navigation (`popup/omnibox.js` and CSS)**: Arrow keys traverse the list.
- **Proposed Navigation**:
  - `Up/Down`: Move to the previous/next visible item (be it a window or a group). If a window is collapsed, skip its groups.
  - `Right`: If the focused item is a window with groups, expand it (show child groups).
  - `Left`: If the focused item is an expanded window, collapse it. If the focused item is a group, move focus to its parent window and collapse it.
  - Ensure the omnibox filter mechanism accurately matches window names *and* group names, perhaps automatically expanding windows whose groups match the search query.

### 3. Modifying "Send" and "Bring" for Groups

- **Targeting Groups**: When the destination of a "Send" or "Bring" action is a group row rather than a window row.
- **API Execution**:
  - In `background/action.js`, `moveTabs` currently targets an `index` in a destination `windowId`.
  - When targeting a group, we still move tabs to the `windowId` (if it's a different window), but we must follow up by explicitly calling `browser.tabs.group({ tabIds: movedTabIds, groupId: targetGroupId })`.
  - Conversely, we may need an "Ungroup" action if the user wants to pull tabs *out* of a group and dump them into the general pool of a window.

### 4. Moving Selected Tabs to Groups

- The "Send" / "Bring" mechanics naturally accommodate moving tabs. If a user selects 3 tabs in Window A and uses "Send" on Group X in Window B, Winger will:
  1. Move the 3 tabs to Window B.
  2. Add those 3 tabs to Group X.
- The UI must clearly indicate whether the user is targeting the parent window or a specific group within it.

### Conclusion

The `tabGroups` API provides all the necessary primitives (`group`, `ungroup`, `get`, `query`, `update`) to implement these features without extensive workarounds. The primary challenge lies in seamlessly updating Winger's highly optimized DOM manipulation (`popup/row.js`, `filter.js`) and keyboard navigation to support the nested hierarchy of Windows > Groups > Tabs, and correctly encoding/decoding Group state in Winger's Stash engine.