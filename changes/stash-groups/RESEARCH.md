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
1. Allow the user to stash an entire window as multiple bookmark folders inside the Stash Home: one folder for each Tab Group, and a window-named folder for the ungrouped tabs of that window.
2. Allow the user to unstash any stashed folder directly in the currently active window, rather than opening a new window.
3. This should be triggered by a new UI button and omnibox command.

### 1. Stashing a Window's Groups as Separate Folders

**Current Behavior:**
When a window is stashed, Winger creates a single bookmark folder named after the window. The window's properties (like `{"private": true}`) are appended to the title. Each tab in the window becomes a bookmark within this single folder. The tab's properties (including its `group` info) are appended to the bookmark's title.

**Proposed Changes:**
Since Tab Groups are not yet surfaced as addressable rows in the UI, we cannot directly stash a *single* group. Instead, we introduce an experimental, opt-in setting (e.g., `stash_groups_as_folders`) that alters how *windows* are stashed.

- **Stash Mechanics**: When stashing a window (`stashWindow` in `background/stash.stash.js`), if the setting is enabled, Winger will iterate through the tabs and identify distinct Tab Groups. For each group found, it will create a *separate*, top-level bookmark folder in the Stash Home, rather than nesting them all under one window folder. Ungrouped tabs are stored in a separate folder, same as if the new setting was disabled; it only affects the behavior of tabs inside groups.
- **Properties**: Metadata will continue to be stored exactly as it is today (in the bookmark titles). We do not need to clutter the folder names with JSON annotations for group properties, as the folder name will just be the group name, and the tab bookmarks inside it will already have the group metadata encoded in them.

### 2. Unstashing a Folder Here (Current Window)

**Current Behavior:**
Right now, `unstashFolder` in `background/stash.unstash.js` reads a folder, creates a new Window (using any encoded `ProtoWindow` properties), and opens the bookmarks in that new window. Group metadata is parsed from the bookmark titles to recreate the groups.

**Proposed Changes:**
- **Trigger**: The user clicks a new "Unstash here" icon button on a stashed folder row, .
- **Unstash Mechanics**: We modify `unstashFolder` (or add an `unstashHere` function) to:
  1. Retrieve the tabs (bookmarks) in the folder.
  2. Open these tabs in the *current* active window (instead of calling `browser.windows.create()`).
  3. Let the existing restore algorithm handle grouping: since the bookmarks contain group metadata in their titles, the existing logic (`StashProp.Tab.postOpen`) will automatically parse it and use `browser.tabs.group` and `browser.tabGroups.update` to recreate the groups natively in the current window.

### 3. Edge Cases in the "Unstash Here" Flow

An important consideration when moving from "Unstash to new window" to "Unstash here" is how the existing `unstashFolder` algorithm handles tab duplication and group collisions in the active window.

*(Note: Handling these edge cases is a non-goal for the MVP. This section is purely informational.)*

**Current Unstash Algorithm Behavior:**
- **Tab Identity:** Winger does not currently check for "identity" (e.g., matching URLs or existing tabs) when unstashing. It blindly iterates through the folder's bookmarks and creates a *new tab* for each bookmark via `browser.tabs.create`. Thus, if the active window already has a tab open with the same URL, Winger will simply create a duplicate tab.
- **Group Collisions:**
  - `StashProp.Tab.postOpen` -> `Groups.restore` collects the stored `protoGroup` metadata from the bookmarks being unstashed.
  - It relies on the *old* `groupId` (saved in the bookmark's JSON metadata) to figure out which tabs belong together so it can group them in a single API call.
  - It then calls `browser.tabs.group({ tabIds: [...] })` *without* passing a `groupId`.
  - According to MDN documentation, passing a `groupId` to `browser.tabs.group` will add the tabs to that existing group. However, since the stashed `groupId` is old, there is no guarantee that a group with that ID still exists—or worse, the ID might now belong to an entirely different group in an unfocused window.
  - Because no `groupId` is provided to the API call, Firefox natively creates a *brand new group* in the destination window and Winger subsequently updates it with the stashed color and title.
  - Therefore, unstashing a group into a window that *already has* a group with the exact same name and color will result in two separate groups with identical visual properties.

**Conclusion for Phase 1:**
For the MVP, this behavior is perfectly fine. "Unstash here" will safely create duplicate tabs and duplicate groups without overwriting or interfering with the existing tabs/groups in the active window. Future iterations could involve investigating a more intelligent "merge" strategy if the user desires to deduplicate tabs or merge into existing groups. For example, if a group with the same `groupId`, `name`, and `color` already exists in the current window, Winger could pass this `groupId` to the `browser.tabs.group` method to merge them; otherwise, it would create a new group as usual.

### 4. API & Extension Requirements

- **Permissions**: Winger already declares the `"tabGroups"` permission in `manifest.json`.
- **Commands & Shorthands**:
  - Add an "Unstash here" action button (with a new icon, e.g., `icons/unstash-here.svg`) in the popup UI for stashed rows (`Template.$folder` in `popup/row.js`).

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