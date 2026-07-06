# Stash and Unstash Logic Tasks

These tasks implement the core functionality in the background scripts.

**Prerequisites**: Familiarity with `background/stash.stash.js`, `background/stash.unstash.js`, `popup/request.js`, and `background/background.message.js`.

### Task 1: Stash Groups as Separate Folders (`stash.stash.js`)
1. Open `background/stash.stash.js`.
2. Locate the logic where a window is stashed (`stashWindow` or `runStashTasks`).
3. Fetch the value of the `stash_groups_as_folders` setting using `Storage.getValue('stash_groups_as_folders')` (import `Storage` if necessary).
4. **Behavior Modification**:
   - If `stash_groups_as_folders` is `true`, instead of dumping all tabs into a single folder:
     - Group the tabs by their `groupId` (or use the group data fetched during `StashProp.Tab.prepare`).
     - For each group found, create a separate bookmark folder.
     - **Important**: Create a separate folder for ungrouped tabs *only* if there are ungrouped tabs in the window.
     - The folder name for a group should be the group's title (or a fallback name if unnamed). The metadata for tabs (like group properties) remains exactly the same as before (encoded in the bookmark titles).

### Task 2: Refactor Unstash Logic (`stash.unstash.js`)
1. Open `background/stash.unstash.js`.
2. Update the `unstashNode` function signature to accept `toNewWindow`:
   ```javascript
   export async function unstashNode(nodeId, remove = true, toNewWindow = true)
   ```
3. Update the `unstashFolder` function to accept `toNewWindow`:
   ```javascript
   async function unstashFolder(folder, remove, toNewWindow)
   ```
4. **Logic Branching in `unstashFolder`**:
   - If `toNewWindow` is `true`, execute the existing logic (`browser.windows.create()`, etc.).
   - If `toNewWindow` is `false` (the "Unstash Here" path):
     - Do not call `browser.windows.create()`.
     - Get the current active window using `browser.windows.getLastFocused()`.
     - Pass this existing window to `populateWindow(window, bookmarks)`. (Note: `populateWindow` does not take a `name` argument).
     - The existing `populateWindow` and `StashProp.Tab.postOpen` logic should automatically handle recreating the tabs and groups in the current window.

### Task 3: Route "Unstash Here" Action
1. Open `popup/request.js`.
2. Locate the `action` function or wherever button clicks are routed to background commands.
3. Ensure that when the `.unstashHere` button is clicked, a specific command or message (e.g., `{ command: 'unstashHere', ... }`) is constructed and sent to the background script.
4. Open `background/background.message.js`.
5. Intercept the `unstashHere` command in the `action` message handler.
6. Call `Stash.Main.unstashNode(request.folderId, request.remove, false)` passing `toNewWindow = false`.
