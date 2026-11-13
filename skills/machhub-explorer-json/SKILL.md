---
name: machhub-explorer-json
description: Guide for users and AI on how to fill in `_explorer.json` for the Designer extension's Explorer panel (MACHHUB 2) — a Workspace's folders and which folder each item is filed in. Covers what an upload creates, moves and saves, what it leaves alone, and moving a Workspace to another server.
license: MPL-2.0
---

# Filling in `_explorer.json` — What to Type

The Explorer file is one Workspace's folders and the items filed in them, stored at `_explorer.json` in the project root and synced from the **Explorer** panel in the Designer extension. It uses paths and names and holds no record ids, so a file from one server applies to another.

> **Needs MACHHUB 2.** On a MACHHUB 1 server the panel says so and offers nothing.

```json
{
  "folders": ["Line 1", "Line 1/Ovens", "Quality"],
  "items": [
    { "folder": "Line 1/Ovens", "type": "workflow", "name": "Oven check", "definition": "name: Oven check\non:\n  manual: {}\njobs: ..." },
    { "folder": "", "type": "artifact", "name": "Shift page", "content": { "$schema": "artifact/v1", "title": "Shift page" } },
    { "folder": "Line 1", "type": "report", "name": "Weekly", "content": { "page": {}, "blocks": [] } },
    { "folder": "Line 1", "type": "collection", "name": "oven_readings" }
  ]
}
```

---

## `folders`

Each entry is a folder's path of names from the Workspace root, joined with `/`.

- A folder name cannot contain `/`, and cannot be empty.
- The root is `""` and is never listed.
- Folders above a listed one may be left out; an upload makes them anyway. A download always lists them.
- Empty folders are kept: list them like any other.

## `items`

| Field | What to put | Required? |
|---|---|---|
| `folder` | The folder's path, or `""` for the root | **Required** |
| `type` | `collection`, `report`, `artifact`, `screen`, `process` or `workflow` | **Required** |
| `name` | The item's name. With `type`, this is how the server finds it | **Required** |
| `definition` | A workflow's YAML, as one string | Workflows only |
| `content` | An artifact's document, or a report's `{template?, page, blocks}` | Artifacts and reports only |

**The name must match inside the content.** A workflow definition's `name` and an artifact's `content.title` must equal the item's `name`. A report's title is its `name`.

**What carries content.** Collections, processes and screens are placed only, and content on them refuses the whole file. Their contents travel through their own panels; see [Related skills](#related-skills).

---

## What an upload does

The panel first asks the server what the file would change and shows every action in the **MACHHUB Explorer** output. Nothing happens until you choose **Apply**.

| In the file | On the server |
|---|---|
| A folder it lacks | Made, parents first |
| An item filed elsewhere | Moved to this folder |
| A workflow or artifact with changed content | Saved as a new draft |
| A report with changed content | Saved as its next version, **live for readers at once** |
| A workflow, artifact or report it lacks | Created from its content; a workflow is an unpublished draft |
| A collection, process or screen it lacks | Skipped, with the reason |

**What an upload leaves alone:**

- Nothing is deleted. Removing a folder or item from the file leaves it on the server.
- Access entries are not in the file and do not change.
- No workflow is published, and its on/off switch stays as it is. A changed workflow runs its old version until someone publishes the draft in MACHHUB.

Uploading the same file twice changes nothing the second time.

**Skips.** A skip does not stop the rest. Besides a missing collection, process or screen, these are skipped: an item in a folder that could not be made, a folder your key cannot edit the parent of, and a name two items of that type share on the server. Read the reason in the output.

**Refused whole.** A file that cannot be applied at all is refused with a message, and nothing is done:

- an empty or over-long folder name
- an unknown `type`
- the same item listed twice
- content on a type that carries none
- more than 2,000 folders or 5,000 items

**Access.** The server reads and applies the file as the developer key's own access entries, or else as the user who made the key, folder by folder and item by item. A download lists only what the key can view, and an upload needs edit access on what it changes.

---

## Moving a Workspace to another server

Upload in this order, so the items the file places already exist:

1. **Namespaces**, **Collections** and **Processes**, from their own panels.
2. Screens, which the Designer does not carry: make them on the server first.
3. The **Explorer** file.

Any skip names the panel to go back to. Fix it and upload the Explorer file again.

---

## Things that bite

❌ **Removing an item from the file to delete it.** It stays on the server. Delete it in MACHHUB.

❌ **Renaming an item in the file.** Items are found by name, so a new name is a different item. A workflow, artifact or report is created alongside the old one; a collection, process or screen is skipped. Rename it in MACHHUB, then download.

❌ **Expecting a changed workflow to run after upload.** It is a draft until someone publishes it in MACHHUB.

⚠️ **Ids inside content are kept as written.** An artifact a report shows, or a collection an artifact reads, may not resolve on another server. Fix those references after a move.

⚠️ **A file marked modified after an upload.** The server may store content in a slightly different form from what you wrote. **Compare JSON with Server** shows the difference; a download settles it.

---

## Using it from the Designer extension

The file has one row in the **Explorer** panel, with its folder and item counts and a colored status (synced, modified, or server only).

- **Download** writes the server's document to the file as it comes, sorted and indented so it diffs cleanly in a repository. It holds no secrets; a workflow's webhook secret is not part of its definition.
- **Upload** shows the plan, then applies it when you choose **Apply**.
- **Compare JSON with Server** (the icon on the row) opens a side-by-side diff. Order of folders, items and keys is ignored.

The safe workflow is **Download → edit → Compare → Upload**.

---

## Related skills

- `machhub-namespace-json` — UNS topic trees, synced from the **Namespaces** panel
- `machhub-collection-json` — collection schemas, synced from the **Collections** panel
- `machhub-sdk-processes` — processes, synced from the **Processes** panel
- `machhub-groups-json` — group assignments, synced from the **Permissions** panel
