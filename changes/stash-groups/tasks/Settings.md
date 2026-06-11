# Settings Tasks

These tasks involve adding the `stash_groups_as_folders` setting so users can opt-in to the new group stashing behavior.

**Prerequisites**: Familiarity with `storage.js` and `page/settings.html`.

### Task 1: Update Storage Defaults
1. Open `storage.js`.
2. Locate the `STORED_PROPS` object.
3. Add a new property `stash_groups_as_folders: false,` right after `stash_nameless_with_title: false,` or in the general stash settings block.
4. Ensure it follows the existing codebase conventions (lower_snake_case for settings).

### Task 2: Update Settings Page UI
1. Open `page/settings.html`.
2. Find the `<fieldset>` block for Stash settings (usually identifiable by legends or existing options like `stash_nameless_with_title`).
3. Add a new checkbox for this setting:
   ```html
   <label>
       <input type="checkbox" id="stash_groups_as_folders">
       Stash tab groups as separate folders
   </label>
   ```
4. If there is a `settings.js` script that needs explicit binding for new settings, verify if it automatically binds by ID (which is common). If it does, no further action is needed in JS for the UI binding.

### Task 3: Background Validation (If any)
1. Verify if `storage.js` `getPopupConfig` or other initialization functions need to explicitly expose `stash_groups_as_folders`. If it's only read by `stash.stash.js` directly via `Storage.getValue('stash_groups_as_folders')`, you might not need to expose it in `getPopupConfig`.
