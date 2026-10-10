
## Goal

Expand bookmarks into automatically saved, live-linked Brave windows. Close pages to free RAM, keep their URLs on the Mac’s SSD, and reopen only what is needed. Initial Design workload goal: roughly 15 windows × 30 tabs. Aka keep track of ~500 tabs

Here, **window/instance** means a Brave window in one browser profile. A **tree** is its named saved collection. Each tab keeps its identity and original tree linkage until removed.
**Full-tree mode** means opening the entire tree as its associated Brave window. Newly created tabs belong to that tree, and manually closing its own tabs, eg Ctrl+W, removes their saved entries. Closing every tab manually deletes the entire tree.
**Partial-open/borrowed mode** means opening or moving one/several linked tabs separately or into another window. Navigation updates their original entries; closing them preserves those entries. Moving tabs does not delete entries or change their owner tree.

## Must have

- Shall be a Brave/chrome plugin
- Automatically save a brave instances tabs when i press the X on a brave instance, which just closes the instance but not the browser itself. 
If the plugin is enabled it will store that instance in its tree/link nodes from now on and i can reopen them in the future.
- Give each saved window a stable identity and editable name. Show its tabs and allow deleting saved trees or individual entries.
- Save locally on disk: URL, title, tab order, and enough identity/link information to reopen and update the same records. Saved pages must not remain running just to preserve them.
- Use a push/event-driven system, not polling. React when Brave pushes tab changes or closure events; keep the latest state saved so tab/window closure and browser shutdown do not depend on a final save alone. Keep performance in mind; do not repeatedly ping for changes.
- Maintain one current state per tree: additions, removals, reordering, and navigation replace its previous state. Do not endlessly append duplicate snapshots or old URLs for one instance. Its always the most current tabs open for that instance/linked tabs.
- Be able to work in a tab from a saved brave instance but only part of it, so only one tab open. But still updating the tree for that instance, while working in the individual tab
- Reopen a whole tree as its associated window, or select one/several tabs to open separately or in an existing window. Have a button/option for that.
- Individually reopened tabs remain linked to their original saved entries. Navigating YouTube → LinkedIn updates that entry’s URL and title in the original tree.
- If a linked tab moves into another automatically saved window, it keeps its original owner. The second tree ignores it entirely: it stores neither a copy nor a reference, and reopening that second tree later does not reopen the borrowed tab.
- Give each tab a unique, stable identifier across all instances. Store that tab only once, link it to only one tree, and allow only one live instance of that tab. Identical URLs are supported as separate tabs with different identifiers.
- If I request an already-open linked tab in another instance, move the existing tab into the requested instance instead of duplicating it or switching focus to its current window. Always preserve its identity and original tree linkage.
- Opening a closed full tree creates a new Brave instance, moves any already-open tabs belonging to it into that instance, and opens the remaining saved tabs. Move existing tabs rather than closing and reloading them.
- Opening an already-open full tree focuses its existing instance instead of creating another one.
- Have a **Resync all tabs** button for an open full tree. It moves borrowed tabs back into that tree's instance. Borrowed tabs stay where I moved them until I request this; do not continuously move them back automatically.
- No trees inside trees,

- The plugin/software must be able to communicate across Brave instances to move linked tabs between them.
- Preserve saved state and tab ownership through normal Brave shutdown, force quit, and restart. When Brave restores its windows normally, reconnect the restored tabs/windows to their saved identities, updating live browser identifiers as needed. Preserve borrowed-tab linkages and support resync afterwards. The plugin must not change Brave's normal startup restoration or open extra tabs/windows itself.
- Use local disk storage only, no cloud. Chrome storage.local is acceptable; document its location/path on the Mac. Persistence after deleting the extension is not required for now.

Future note: allow changing which tree owns a tab and editing the tree arrangement. For now, moving a live tab between windows never changes its tree ownership. No tree-ownership editing in this version.

## The close/delete distinction

| Action | Required result |
| --- | --- |
| Close a whole tree window with X | Keep its latest remaining entries saved, including its tabs borrowed elsewhere; mark the window closed. |
| Close an owned tab while editing the fully reopened tree | Remove that tab from that tree’s current membership. Borrowed tabs follow the partial-open rule. |
| Close every tab manually in full-tree mode, including the last with Ctrl+W | Delete the entire tree. This is different from closing the window with X. |
| Close a tab opened individually/from a subset | Keep its original saved entry at its latest URL. |
| Navigate a linked tab in another window | Update only its original saved entry. The host tree ignores the borrowed tab. |
| Close that other window | Save its own tabs, excluding borrowed tabs; preserve borrowed entries in their original trees. |
| Move a linked tab between windows | Preserve its identity and original owner; do not treat the move as deletion. |
| Resync all tabs in the original full-tree window | Move its borrowed tabs back into that window. |
| Explicitly delete a saved entry/tree in the manager | Remove it deliberately, rather than treating it as an ordinary window close. |

Example: Tree A has 30 tabs. Open only its YouTube tab in window B, browse to LinkedIn, then close that tab. A still has 30 entries, with LinkedIn replacing YouTube. B’s unrelated tabs are unaffected. Reopen all of A, close five tabs, add two, then close A’s window: A now contains the latest 27 tabs.

Example: A's full-tree window is open. Borrow one of its tabs into B: the existing tab moves to B but still belongs to A, and B does not save it. Opening A again focuses A's existing window. Press **Resync all tabs** in A to move the borrowed tab back without reloading it.
