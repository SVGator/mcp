# Tools reference

Every tool the SVGator MCP server exposes, with its full input and output contract. All tools
return JSON both as structured content (validated against the advertised `outputSchema`) and as a
text block with the same JSON. Errors come back as tool errors of the form
`{ "error": "<message>", "code": <http-status> }` — the messages are written to be actionable,
so an assistant can read the constraint and retry with a corrected call.

**Id conventions:** projects are `pi_…`, assets are `as_…`, folders are `fd_…`, render jobs are
`ri_…`, teams are `tm_…`. Element ids inside a project are addressed as `#<id>` (see
[Editing projects](editing.md) for the full path grammar).

| Tool | Purpose | Auth |
| --- | --- | --- |
| [`ping`](#ping) | Connectivity diagnostic | none |
| [`get_profile`](#get_profile) | Account name, email, plan, teams | required |
| [`list_projects`](#list_projects) | List your projects | required |
| [`get_project`](#get_project) | One project's summary + settings | required |
| [`get_skeleton`](#get_skeleton) | Token-light structure map of a project | required |
| [`read_part`](#read_part) | Read one element / property / value | required |
| [`edit_part`](#edit_part) | Apply one validated edit | required |
| [`create_project`](#create_project) | Create a project from full JSON | required |
| [`replace_project`](#replace_project) | Overwrite a project with full JSON | required |
| [`duplicate_project`](#duplicate_project) | Copy a project (optionally into a folder / retitled) | required |
| [`move_project`](#move_project) | Move a project to a folder and/or rename it | required |
| [`list_assets`](#list_assets) | List your reusable assets | required |
| [`get_asset`](#get_asset) | One asset's summary + document | required |
| [`create_asset`](#create_asset) | Create an asset from a document | required |
| [`update_asset`](#update_asset) | Overwrite an asset's document | required |
| [`list_folders`](#list_folders) | List your folders | required |
| [`get_folder`](#get_folder) | One folder's summary | required |
| [`create_folder`](#create_folder) | Create a folder | required |
| [`update_folder`](#update_folder) | Rename a folder | required |
| [`describe_shape`](#describe_shape) | Authoring reference for one building block | none |
| [`export_project`](#export_project) | Export to svg / lottie / react-native / flutter / video / gif | required |
| [`get_render`](#get_render) | Poll an async video/gif render job | required |

---

## `ping`

Diagnostic connectivity check; requires no authentication. Use it to confirm the server is
reachable and the transport works.

**Inputs:** none.

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `message` | string | Always `"pong"` |
| `server` | string | Server name |
| `version` | string | Server version |
| `time` | string | ISO-8601 server time |

---

## `get_profile`

The authenticated customer profile. Useful for limit-aware UX before exporting.

**Inputs:** none.

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `name` | string | Account name |
| `email` | string | Account email |
| `picture` | string \| null | Avatar URL |
| `plan` | object | `{ name, access_level }` |
| `teams` | array | Teams the account belongs to |

---

## `list_projects`

Lists the authenticated user's projects, newest first. Returns lean rows plus a total count for
pagination.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `limit` | integer 1–1000 | no | Max projects to return (default 100) |
| `offset` | integer ≥ 0 | no | Pagination offset |
| `search` | string | no | Filter by title substring |
| `team_id` | string | no | Restrict to a team (`tm_…`) |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `projects` | array | Rows of `{ id, title, preview, created, updated, editor_url }` |
| `total` | number | Total matching projects (for pagination) |
| `offset` | number | The offset this page starts at |

`editor_url` is a deep link that opens the project in the SVGator editor
(`https://app.svgator.com/editor#/…`); it is `null` when a row has no id.

---

## `get_project`

A single project's summary: metadata plus its saved settings. Does **not** return the full
project JSON — use [`get_skeleton`](#get_skeleton) / [`read_part`](#read_part) to inspect content.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id, e.g. `pi_abc123` |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `id`, `title` | string | Project metadata |
| `folder` | object \| null | The folder the project belongs to (`{ id: "fd_…", title }`); null when not in any folder |
| `preview` | string \| null | Preview image URL |
| `created`, `updated` | number | Timestamps |
| `settings` | object | Saved export settings, canvas size, animation timeline |
| `editor_url` | string | Deep link to open the project in the SVGator editor |

---

## `get_skeleton`

A cheap, token-light **structure map** of a project — step 1 of the editing loop (navigate →
read → edit → verify). Returns a tree of nodes carrying **names and counts only, never values**:
no path data, no keyframe arrays, no gradient stops. Use it to locate an element by `id` and see
which channels are animated, instead of ever downloading the project JSON.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id |
| `item` | string | no | Scope the map to the subtree under this element id (e.g. `#g-abc123`). Omit for the full project, which then also returns `definitions` |
| `depth` | integer ≥ 0 | no | Element-tree depth to descend; `0` = infinite (default) |
| `fields` | string | no | Comma-separated extra node fields to include, e.g. `title,locked` |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `meta` | object | Project-level metadata: `animation` summary (duration, iterations, direction), `viewBox`, canvas `size`, `bgColor` |
| `tree` | node | The element tree (see node shape below) |
| `definitions` | node[] | Reusable `<defs>` subtrees, referenced by `use` nodes via `ref`. Present only for a full (unscoped) skeleton |

**Node shape**

| Field | Meaning |
| --- | --- |
| `id`, `type` | Element identity (`null` when the stored document left a node un-ided) |
| `hidden` | Present **only** when the element is truly hidden |
| `ref` | On a `use` node: the target definition id (look it up in `definitions`) |
| `properties` | Property channel **names** only — read a value with `read_part` |
| `animators` | Animated channels, **nested** group → channel → keyframe count, e.g. `{"transform":{"translate":3,"scale":5}}` means `translate` has 3 keyframes (valid indices 0..2). A disabled channel is `{"keys":<count>,"disabled":true}` instead of a bare count |
| `children` | Child nodes |

---

## `read_part`

Reads the exact value or element at an `item` path — step 2 of the editing loop, after
`get_skeleton` has located the element. The path can target an **element** (`#<id>`), a
**property/animator sub-object** (`#<id>/properties/fill`), or a **leaf value**
(`#<id>/properties/fill/paint`).

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id |
| `item` | string | yes | Id-anchored path to read |
| `depth` | integer ≥ 0 | no | Child levels to include; only meaningful for element targets (`0` = infinite) |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `resolvedItem` | string | The canonical id-anchored path the item resolved to — copy it **verbatim** as the target of a subsequent edit |
| `itemAnimated` | boolean | **Important:** `true` means this static value is shadowed by an animation, so editing it has no visible effect — edit the keyframe (`…/animators/<group>/<channel>/keys/<i>/value`) instead |
| `value` | any | The value at `item`: a leaf scalar, a paint/transform object, a sub-object, or an element subtree — depends on `item` and `depth` |
| `projectAnimated` | boolean | Whether the project has any animation at all |

---

## `edit_part`

Applies **one validated edit** at `item` and persists it — step 3 of the editing loop. Invalid
edits are rejected with a descriptive validation error and nothing is persisted. See
[Editing projects](editing.md) for the workflow, path grammar, and authoring rules.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id |
| `item` | string | yes | Id-anchored target: `#<id>` (whole element), a property/leaf path, an animator channel, or a keyframe `#<id>/animators/<group>/<channel>/keys/<i>`. For `move`, the element to relocate |
| `action` | enum | no | `update` (default) \| `before` \| `after` \| `append` \| `prepend` \| `delete` \| `move` |
| `value` | any | for update/inserts | The new value (`update`) or the new element subtree (inserts). Omit for `delete` and `move` |
| `target` | string | for move | Destination element id |
| `position` | enum | for move | `before` \| `after` \| `append` \| `prepend` relative to `target` |
| `preview` | enum | no | `none` (default, cheap) \| `image` (static SVG string for visual verification) \| `module` (SVG + animation data; opt-in, can be ~400 KB) |

**Actions**

- **`update`** — overwrites the value at `item`. A bare `#<id>` replaces the whole element but
  keeps its id; a node that omits `children` keeps its existing subtree, while `animators` are
  taken verbatim (an omitted animator is **removed**). A path inside the element replaces just
  that value.
- **`before` / `after`** — insert a sibling element; **`append` / `prepend`** — add a last/first
  child (the only way to populate an empty container). Send the new element subtree as `value`.
  Element targets only.
- **`move`** — relocates the `item` element (preserving its id and animators) to
  `target` + `position`.
- **`delete`** — removes an element (and its subtree), a whole animator channel
  (`…/animators/fill/paint`), or a single keyframe (`…/keys/<i>`).

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `result` | string | `"success"` when the edit was validated and persisted |
| `element` | object \| null | The affected element **as finally stored** (server-assigned ids, first-keyframe-synced baseline, preserved children), bounded to the posted depth + 1 — verify the result without a follow-up read. `null` on an element delete |
| `insertedItem` | string | Inserts: the canonical path of the new element — address it by **this**, never by the id you sent |
| `idMap` | object | Inserts: map of sent id → server-assigned id, across the whole subtree |
| `deletedItem` / `movedItem` | string | The path of the removed / relocated target |
| `itemAnimated` | boolean | `true` flags that the edited static value is shadowed by an animation (the edit is a visual no-op — edit the keyframe instead) |
| `projectAnimated` | boolean | Whether the project has any animation after the edit |
| `preview` | string | `preview=image\|module`: the rendered static SVG string |
| `animations`, `options`, `root`, `playerId`, `version` | various | `preview=module` only: the split-out animation payload and its decode keys |

---

## `create_project`

Creates a **new** project from a full project JSON, owned by the authenticated user. This is a
coarse write (a whole document, often hundreds of KB) — use it for a document you have assembled
or for cloning/templating; for incremental tweaks prefer [`edit_part`](#edit_part). The whole
project is validated as one unit: a single wrong field fails the entire create (and nothing is
persisted), so consult [`describe_shape`](#describe_shape) for each element type and concept
before authoring.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `title` | string | no | Project title (a DB field, not part of the document). Defaults to `"Untitled"` |
| `document` | object | yes | The document tree — the root `svg` element. A minimal blank project needs only the root svg's `properties.shape.position {x,y}` and `size {width,height}` |
| `definitions` | object[] | no | Reusable `<defs>` element subtrees, referenced by `use` elements via id |
| `options` | object | no | Export/config. **Caution:** a stored project nests every format under `options.export.settings.<format>` — this is *not* the flattened single-format shape `export_project` takes; do not copy export parameters here |
| `folder` | string \| null | no | Id of an existing folder (`fd_…`) to file the project under. `null` moves it out of any folder; omit to keep the current one (create defaults to no folder) |

**Output:** the saved project metadata (same shape as [`get_project`](#get_project), including
`editor_url`) — not the full project back.

---

## `replace_project`

Replaces an existing project's **entire** JSON with the supplied full project. Requires edit
rights (demo/template projects are read-only). Last write wins — there is no concurrency check —
but validation runs before any mutation, so a rejected replace leaves the stored project
untouched. When `title` is omitted the current title is kept.

**Inputs:** `project_id` (required) plus the same `title` / `document` / `definitions` /
`options` / `folder` as [`create_project`](#create_project).

**Output:** the saved project metadata, including `editor_url`.

---

## `duplicate_project`

Makes a **copy** of an existing project, owned by the authenticated user. Server-side copy — it
preserves the full document and its image references without downloading the project. The copy
keeps the source title unless you pass `title`, and is filed in the **same folder** as the source
unless you pass `folder_id`. Duplicating into the same folder is allowed.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id (`pi_…`) to duplicate |
| `folder_id` | string \| null | no | Folder (`fd_…`, must already exist) to file the copy under. `null` = no folder; omit to reuse the source project's folder |
| `title` | string | no | Title for the copy. Omit to reuse the source project's title |

**Output:** the **new** project's metadata (its own id, title, folder, timestamps, `editor_url`),
same shape as [`get_project`](#get_project).

---

## `move_project`

Moves an existing project to a different folder and/or renames it (requires edit rights). Omit
`folder_id` to keep the current folder; omit `title` to keep the current title. Moving into the
folder the project is already in **without** a rename is rejected (nothing to do) — but renaming in
place (same/omitted folder + a new title) is allowed. Lightweight — it does not rewrite the
document.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id (`pi_…`) to move |
| `folder_id` | string \| null | no | Destination folder (`fd_…`, must already exist). `null` removes it from any folder; omit to keep the current folder |
| `title` | string | no | New title. Omit to keep the current title |

**Output:** the updated project metadata, including `editor_url`.

---

## `list_assets`

Lists the authenticated user's **reusable assets** — static or animated SVG items you can inspect
or drop into a project — newest first. Returns lean rows (no document) plus a total count for
pagination.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `limit` | integer 1–1000 | no | Max assets to return (default 100) |
| `offset` | integer ≥ 0 | no | Pagination offset |
| `search` | string | no | Filter by title substring |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `assets` | array | Rows of `{ id: "as_…", title, preview, created, updated }` |
| `total` | number | Total matching assets (for pagination) |
| `offset` | number | The offset this page starts at |

---

## `get_asset`

A single asset including its `data` — the full reusable SVG item. Takes an asset id (`as_…`) from
[`list_assets`](#list_assets).

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `asset_id` | string | yes | Asset id, e.g. `as_abc123` |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `id`, `title` | string | Asset metadata |
| `preview` | string \| null | CDN preview URL |
| `created`, `updated` | string | Timestamps (ATOM/UTC) |
| `data` | object | The asset payload: `{ document, definitions }` |

---

## `create_asset`

Creates a **new** reusable asset from an asset document, owned by the authenticated user. The
whole document is validated as one unit — a single wrong field rejects the create and nothing is
persisted — so consult [`describe_shape`](#describe_shape) for each element type before authoring.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `title` | string | no | Asset title (a DB field, not part of the document). Defaults to `"Untitled"` |
| `document` | object | yes | The document tree — the root `svg` element. **Must** carry a numeric canvas size at `properties.shape.size {width,height}` (it seeds the asset dimensions). Can be static or animated |
| `definitions` | object[] | no | Reusable `<defs>` element subtrees, referenced by `use` elements via id |

**Output:** the saved asset (same shape as [`get_asset`](#get_asset), including its `data`).

---

## `update_asset`

Replaces an existing asset's **entire** document with the supplied one (coarse write). Validation
runs before any mutation, so a rejected update leaves the stored asset untouched. When `title` is
omitted the current title is kept.

**Inputs:** `asset_id` (required) plus the same `title` / `document` / `definitions` as
[`create_asset`](#create_asset).

**Output:** the saved asset, including its `data`.

---

## `list_folders`

Lists the authenticated user's **folders** — the labels projects are filed under — newest first.
Returns lean rows plus a total count for pagination. Use a folder id (`fd_…`) as the `folder`
argument of [`create_project`](#create_project) / [`replace_project`](#replace_project) to file a
project under it.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `limit` | integer 1–1000 | no | Max folders to return (default 100) |
| `offset` | integer ≥ 0 | no | Pagination offset |
| `search` | string | no | Filter by title substring |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `folders` | array | Rows of `{ id: "fd_…", title, created, updated }` |
| `total` | number | Total matching folders (for pagination) |
| `offset` | number | The offset this page starts at |

---

## `get_folder`

A single folder's summary, by its id (`fd_…`) from [`list_folders`](#list_folders).

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `folder_id` | string | yes | Folder id, e.g. `fd_abc123` |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `id`, `title` | string | Folder metadata |
| `created`, `updated` | string | Timestamps (ATOM/UTC) |

---

## `create_folder`

Creates a **new** folder (a label to organize projects) owned by the authenticated user. Only a
`title` is needed.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `title` | string | yes | Folder name (non-empty) |

**Output:** the saved folder (`{ id: "fd_…", title, created, updated }`). Use that id as the
`folder` argument of [`create_project`](#create_project) / [`replace_project`](#replace_project).

---

## `update_folder`

Renames an existing folder. Only the `title` can be changed.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `folder_id` | string | yes | Folder id to rename, e.g. `fd_abc123` |
| `title` | string | yes | The new folder name (non-empty) |

**Output:** the saved folder.

---

## `describe_shape`

The structural reference for **one building block** — the exact JSON shape to copy when authoring
elements for `edit_part` inserts or `create_project` / `replace_project`, so field names are
never guessed. Pure reference data; requires no authentication.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `topic` | enum | yes | A concept or element type (below) |

Concepts: `root`, `animation`, `element`, `keyframe`, `easing`, `paint`, `morph`, `transform`,
`fill`, `stroke`, `compositing`, `filter`.
Element types: `svg`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`, `path`, `text`,
`tspan`, `image`, `use`, `a`, `g`, `mask`, `clipPath`, `defs`, `symbol`, `pattern`, `marker`.

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `topic` | string | The topic described |
| `summary` | string | One-line description |
| `shape` | any | A copy-ready example of the exact JSON shape (or a map of variants) |
| `groups` | string[] | Element types: the property groups **this** element carries — not universal (e.g. `image`/`g`/`a` have no fill/stroke; `svg`/`defs` have none) |
| `properties` | object | Channels annotated `"<value hint> · animatable\|static · default <x>"` |
| `animators` | object | Element types: the animatable channels as a **nested** map mirroring the required `animators.<group>.<channel>` JSON (group = `transform` \| `fill` \| `stroke` \| `compositing` \| `filter` \| `shape`) |
| `notes` | string[] | Gotchas for this topic |
| `see` | string[] | Related topics to fetch next |

The full catalog behind this tool is also published as a single MCP resource,
`svgator://reference/element-shapes` — see [Skill & resources](skill-and-resources.md).

---

## `export_project`

Exports a project to a deliverable format. The result always has a `kind` field that tells you
how the file is delivered — see [Exporting projects](exporting.md) for the workflow, all
override parameters, and their format gating.

**Core inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `project_id` | string | yes | Project id |
| `format` | enum | yes | `svg` \| `lottie` \| `react-native` \| `flutter` \| `video` \| `gif` |
| `inline` | boolean | no | Sync formats only. Default `false`: the result is a temporary `content_url` link (token-light). `true`: the file contents come back inline. Ignored for `video`/`gif` |

Plus the optional override parameters documented in
[Exporting projects → Export options](exporting.md#export-options) (animation playback,
interactivity triggers, raster quality/resolution, background colour, …). Every override is
optional — omit it to keep the project's saved setting. An override that does not apply to the
chosen format returns a validation error naming the exact constraint.

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `format` | enum | The requested format |
| `kind` | enum | `"link"` — temporary URL delivery (default for sync formats); `"content"` — file inline (`inline=true`); `"render"` — async raster job to poll |
| `content_url` | string | `kind="link"`: temporary URL to the exported file, valid ~1–2 days |
| `content` | string | `kind="content"`: the exported file as a string (SVG markup, Lottie JSON, or framework-wrapped source) |
| `exportId` | string | Stored export reference for a content/link export |
| `render_id` | string | `kind="render"`: the job id to poll with [`get_render`](#get_render) |
| `status`, `jobStatus`, `percent`, `extension`, `download`, `downloadExpires`, `preview` | various | `kind="render"`: the initial job state (same fields as `get_render`) |
| `note` | string | Delivery guidance for the assistant |

---

## `get_render`

Checks the status of a raster (video/gif) render started by `export_project`.

**Inputs**

| Parameter | Type | Required | Meaning |
| --- | --- | --- | --- |
| `render_id` | string | yes | Render job id, e.g. `ri_abc123` |

**Output**

| Field | Type | Meaning |
| --- | --- | --- |
| `render_id`, `project_id` | string | Job identity |
| `status` | string | `pending` \| `starting` \| `running` \| `done` \| `failed` |
| `jobStatus` | number | The raw status code behind `status` |
| `percent` | number \| null | Render progress |
| `extension` | string \| null | Output container (mp4, gif, …) |
| `download` | string \| null | **Signed, short-lived (~5 min)** download URL; present once `status` is `done` |
| `downloadExpires` | string \| null | When the download URL expires |
| `preview` | string \| null | Preview URL |
| `note` | string | Present while the job is still running |
