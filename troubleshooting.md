# Troubleshooting

## Connecting & authentication

**Stuck on "connecting" / an auth loop (claude.ai / Claude Desktop).**
Remove the connector and re-add it (Settings → Connectors) — this re-runs the OAuth client
registration — then authorize again.

**Asked to reconnect after a while.**
Access tokens are short-lived (about an hour) and refresh automatically; if a session is lost you
may be asked to sign in to SVGator again. Your projects are unaffected.

**`invalid_client` when reconnecting from a custom MCP client.**
The client's dynamic registration is no longer known to the server. Re-register (for Claude Code:
`claude mcp remove svgator` then `claude mcp add …` again).

**"Not authenticated" from a tool.**
The session's token expired mid-conversation. Re-authenticate the connector; the tools work again
immediately.

**Is the server even reachable?**
The `ping` tool needs no account and returns the server name, version, and time — ask the
assistant to "ping the SVGator server".

## Exporting

**"Animate it first" on a video/gif export.**
Raster export renders the animation, so the project must have one. Add animation (in the editor,
or by asking the assistant to animate it) and retry.

**"Invalid export parameters — …" validation error.**
The message names the exact constraint — e.g. a GIF-only option (`dither`, `matte`,
`colorPaletteSize`) used on a video export, or `fps` above the format's cap (100 web/mobile/
lottie, 60 video, 30 gif). Adjust the offending parameter and retry. The full applicability
matrix is in [Exporting projects → Export options](exporting.md#export-options).

**The download link stopped working.**
Two different lifetimes are at play:
- a raster render's `download` URL is a signed link valid ~**5 minutes** — poll `get_render`
  again to get a fresh one;
- a sync export's `content_url` lives ~**1–2 days** — re-run the export if it has lapsed.

**A render seems stuck.**
`get_render` reports `status` (`pending` → `starting` → `running` → `done` / `failed`) and
`percent`. Keep polling while it is running; a `failed` status means the render itself errored —
re-export, and if it persists, check the project renders correctly in the SVGator editor.

## Editing

**An edit "succeeded" but nothing changed visually.**
Check `itemAnimated` in the response: `true` means the static value you wrote is shadowed by an
animation. Edit the keyframe instead — `#<id>/animators/<group>/<channel>/keys/<i>/value`. See
[Editing projects](editing.md#the-one-rule-that-prevents-invisible-edits-itemanimated).

**A `422` validation error on `edit_part` / `create_project` / `replace_project`.**
The server validates before persisting anything, and the message says exactly what's wrong
(unknown field, wrong enum value, malformed keyframe, …). The fix is usually to call
`describe_shape` for the element/concept in question and copy the exact field shape. For the
coarse writes, remember the whole document validates as one unit — one wrong field fails the
entire call.

**"Read-only project" on `replace_project` / `edit_part`.**
Demo and template projects can't be overwritten — they're shared, and writable by nobody. Call
`duplicate_project` with the id to get your own copy, then edit that. This is the normal first step
after [`list_templates`](tools.md#list_templates), not a workaround. Reading a template
(`get_project`, `get_skeleton`, `read_part`) and exporting one as it is both work without
duplicating.

**`invalid_structure` when inserting or building a clip/mask.**
A `clipPath` or `mask` belongs **inside** the group that references it, and a container may hold
**at most one** — two as direct siblings is rejected. Nest two groups when one element needs both a
clip and a mask. See
[Clips and masks](editing.md#clips-and-masks-one-per-container).

**"Invalid folder ID specified" on `list_projects`.**
`folder_id` must be one of your own folders — call `list_folders` to get valid `fd_…` ids. The
filter is refused rather than ignored on purpose, so an empty result never hides a dropped filter.

**An inserted element can't be found afterwards.**
The server assigns/remaps ids on insert. Address the new node via the `insertedItem` path (and
`idMap` for its subtree) returned by the insert — never via the id that was sent.

## General

**Responses feel token-heavy / slow.**
The server is designed for token-light flows: `get_skeleton` instead of full project JSON,
`kind:"link"` delivery instead of inline files, `preview:"none"` by default on edits. If an
assistant is inlining large exports, ask it to hand you the `content_url` link instead.

**Removing access.**
claude.ai / Claude Desktop: **Settings → Connectors → SVGator → Remove**. Claude Code:
`claude mcp remove svgator`. This revokes the client's access; your SVGator account and projects
are untouched.
