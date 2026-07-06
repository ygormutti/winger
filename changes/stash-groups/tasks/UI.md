# UI Tasks

These tasks involve adding the "Unstash Here" button to the UI of stashed folders.

**Prerequisites**: Familiarity with `icons/unstash.svg`, `popup/popup.html`, `popup/row.js`, and `popup/popup.css`.

### Task 1: Create Icon
1. Copy `icons/unstash.svg` to `icons/unstash-here.svg`.
2. Modify the pre-existing unstash icon (`unstash.svg`) by adding a plus sign centered inside the white rounded rectangle.

### Task 2: Add Button to HTML Template
1. Open `popup/popup.html`.
2. Locate the `<window-row id="currentWindowRow">` element.
3. Next to the existing `<button class="windowAction stash" ...>` add a new button for "Unstash here". We only add it here because `Template.$folder` is cloned from `Template.$window` (which comes from `currentWindowRow`).
   - Example: `<button class="unstashHere tabAction" title="Unstash here" hidden><img src="../icons/unstash-here.svg"></button>`.

### Task 3: Handle the Button in Row Hydration
1. Open `popup/row.js`.
2. Add `.unstashHere` to the `CELL_SELECTORS` set.
3. In `FolderRow.init()`, ensure the `.unstashHere` button is configured for stashed rows. Set the title to "Unstash Here" and explicitly remove the hidden/disabled attributes (using logic similar to how `.stash` is handled, e.g., removing it from `disableElement` or explicitly managing its state).
4. For non-stashed rows (`WindowRow.init()`), the `.unstashHere` button will be removed from the DOM automatically if not handled differently, or should be explicitly removed if it isn't part of the standard `CELL_SELECTORS` cleanup.

### Task 4: CSS Updates
1. Open `popup/popup.css`.
2. Add necessary styles for `.unstashHere`. Tie its icon visibility to the `.stashed` class, similar to `.stash`, so that it follows codebase conventions.
3. Example:
   ```css
   window-row.stashed & {
       background-image: url("../icons/unstash-here.svg");
   }
   ```
   (Note: Adjust the exact nesting based on where `.unstashHere` is placed relative to `window-row` in `popup.css`).
