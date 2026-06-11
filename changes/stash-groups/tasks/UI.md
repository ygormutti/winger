# UI Tasks

These tasks involve adding the "Unstash Here" button to the UI of stashed folders.

**Prerequisites**: Familiarity with `icons/unstash.svg`, `popup/popup.html`, `popup/row.js`, and `popup/popup.css`.

### Task 1: Create Icon
1. Copy `icons/unstash.svg` to `icons/unstash-here.svg`.
2. Modify the pre-existing unstash icon (`unstash-here.svg`) by adding a plus sign centered inside the white rounded rectangle.

### Task 2: Add Button to HTML Template
1. Open `popup/popup.html`.
2. Locate the `<window-row id="currentWindowRow">` and `<window-row id="newWindowRow">` elements.
3. Next to the existing `<button class="stash tabAction" ...>` (which functions as the unstash button when the row is a stashed item), add a new button for "Unstash here".
   - Example: `<button class="unstashHere tabAction" title="Unstash here" disabled><img src="../icons/unstash-here.svg"></button>`.
4. Hide it or handle it appropriately so it doesn't appear on non-stashed items.

### Task 3: Handle the Button in Row Hydration
1. Open `popup/row.js`.
2. Add `.unstashHere` to the `CELL_SELECTORS` set.
3. In `FolderRow.init()`, ensure the `.unstashHere` button is properly hydrated for stashed rows. Since it's a `tabAction` in the template, you might need to ensure its disabled state is managed correctly (or just remove the disabled attribute for folder rows, similar to how `.stash` is handled).
4. For non-stashed rows (`WindowRow.init()`), the `.unstashHere` button should be removed from the DOM entirely so it doesn't show up on active windows.

### Task 4: CSS Updates
1. Open `popup/popup.css`.
2. Add necessary styles for `.unstashHere`. You will likely want to tie its visibility to the `.stashed` class, similar to `.stash`, so that it only shows up for stashed items.
3. Example:
   ```css
   window-row:not(.stashed) .unstashHere {
       display: none;
   }
   ```
