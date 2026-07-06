# Documentation and Testing Tasks

Since Winger currently does not have an automated test suite, validation for this feature relies on careful manual testing. Additionally, user documentation must be updated to reflect the new feature.

### Task 1: Update Documentation
1. Review `README.md`.
2. Add a short section explaining the new "Stash tab groups as separate folders" setting.
3. Explain the new "Unstash Here" button that appears on stashed folders, allowing users to restore stashed tabs into their current active window rather than opening a new window.

### Task 2: Manual Testing Protocol
Follow these steps to ensure the new feature works flawlessly:

**Test 1: Normal Stashing (Setting Disabled)**
1. Ensure `stash_groups_as_folders` is unchecked in the settings page.
2. Create a window with 2 ungrouped tabs and 2 tabs inside a group.
3. Stash the window.
4. Verify that in the Firefox bookmarks (Stashed Windows folder), *only one* folder was created for the window, containing all 4 tabs.

**Test 2: Group Stashing (Setting Enabled)**
1. Enable `stash_groups_as_folders` in the settings page.
2. Create a window with 2 ungrouped tabs and 2 tabs inside a named group.
3. Stash the window.
4. Verify that in the bookmarks, *two* separate folders were created: one named after the window (containing the 2 ungrouped tabs) and one named after the group (containing the 2 grouped tabs).

**Test 3: Group Stashing Edge Case (No Ungrouped Tabs)**
1. With the setting enabled, create a window where *all* tabs are inside a group (0 ungrouped tabs).
2. Stash the window.
3. Verify that only the group folder is created, and no empty window folder is created.

**Test 4: Unstash Here**
1. With some stashed folders available (from previous tests).
2. Open the Winger popup and view stashed items.
3. Verify the new "Unstash Here" button is visible alongside the standard "Unstash" button on stashed rows.
4. Click "Unstash Here" on one of the folders.
5. Verify that the tabs from the folder open in your *current* window.
6. Verify that if those tabs belonged to a group, the group is recreated natively in the current window.
7. Verify that the stashed folder is correctly deleted after unstashing (if standard behavior is to delete on unstash).
8. Verify that standard "Unstash" (to a new window) still works correctly.
