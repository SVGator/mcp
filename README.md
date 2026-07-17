# SVGator MCP Server — Documentation

The MCP server connects your AI assistant straight to [SVGator](https://www.svgator.com), the online SVG animation tool. 
From there, your assistant can work inside your projects to **browse, inspect, edit, create, and export** them for you, 
handling everything from a simple “edit colors according to this brand kit” to a complete 
“build me an animated logo reveal and export it as a CSS-only file.”

- **Endpoint:** `https://mcp.svgator.com/mcp`
- **Protocol:** [Model Context Protocol](https://modelcontextprotocol.io) over Streamable HTTP
- **Authentication:** OAuth 2.1 (sign in with your SVGator account — see
  [Getting started](getting-started.md))

## What you can do

| Capability | Tools | Example prompt |
| --- | --- | --- |
| Browse your projects | `list_projects`, `get_project`, `get_profile` | "List my SVGator projects" |
| Inspect a project's structure | `get_skeleton`, `read_part` | "What elements are in project X, and which are animated?" |
| Edit a project incrementally | `edit_part` | "Make the circle red and loop its bounce forever" |
| Create or replace whole projects | `create_project`, `replace_project` | "Build me a new animated loading spinner" |
| Duplicate, move & rename projects | `duplicate_project`, `move_project` | "Copy project X into my 'Drafts' folder" |
| Manage reusable assets | `list_assets`, `get_asset`, `create_asset`, `update_asset` | "Save this icon as a reusable asset" |
| Organize projects into folders | `list_folders`, `get_folder`, `create_folder`, `update_folder` | "Make a 'Marketing' folder and file this project under it" |
| Look up authoring reference | `describe_shape` | (used by the assistant to get exact JSON shapes) |
| Export to deliverable formats | `export_project`, `get_render` | "Export project X as an MP4 video" |
| Get motion-design craft guidance | skill resources + prompt | "Art-direct a logo reveal, then build it" |

Every project the server returns includes an `editor_url` — a deep link that opens the project in
the SVGator editor, so you can always continue by hand where the assistant left off.

## Tools

The server exposes 22 tools. See the [Tools reference](tools.md) for full parameter and output
tables.

**Projects**

- **`list_projects`** — List the authenticated user's SVGator projects (newest first), with an `editor_url` deep link per row and a total count for pagination.
- **`get_project`** — Get a single project summary: title, preview, folder, timestamps, canvas size, background colour, animation timeline, and saved export settings.
- **`create_project`** — Create a new SVGator project from a full project JSON, owned by the authenticated user.
- **`replace_project`** — Replace an existing project's entire JSON with a supplied full project (validation runs before any mutation).
- **`duplicate_project`** — Make a copy of an existing project, optionally into a different folder or under a new title.
- **`move_project`** — Move a project to a different folder and/or rename it.

**Inspecting & editing a project**

- **`get_skeleton`** — Get a cheap, token-light structure map of a project (element tree with names and keyframe counts only, never values) — step 1 of the editing loop.
- **`read_part`** — Read the exact value or element at an id-anchored path — step 2 of the editing loop.
- **`edit_part`** — Apply one validated edit (update, insert, move, or delete) at a path and persist it — step 3 of the editing loop.
- **`describe_shape`** — Get the exact JSON shape of one building block (element or animator) before authoring, so field names are never guessed.

**Assets**

- **`list_assets`** — List the authenticated user's reusable assets (static or animated SVG items), newest first.
- **`get_asset`** — Get a single asset including its full `data` ({document, definitions}) payload.
- **`create_asset`** — Create a new reusable asset from an asset document.
- **`update_asset`** — Replace an existing asset's entire document.

**Folders**

- **`list_folders`** — List the authenticated user's folders (the labels projects are filed under).
- **`get_folder`** — Get a single folder (id, title, timestamps) by its id.
- **`create_folder`** — Create a new folder to organize projects.
- **`update_folder`** — Rename an existing folder.

**Export & account**

- **`export_project`** — Export a project to a deliverable format (SVG, Lottie, React Native, Flutter, video, or GIF), with optional per-export overrides.
- **`get_render`** — Poll the status of a raster (video/GIF) render started by `export_project` and get the signed download URL when done.
- **`get_profile`** — Get the authenticated customer profile: name, email, plan, and teams.

## Documentation map

| Document | For | Covers |
| --- | --- | --- |
| [Getting started](getting-started.md) | Everyone | Connecting from claude.ai, Claude Desktop, Claude Code, and other MCP clients; authentication; privacy |
| [Tools reference](tools.md) | Integrators / the curious | Every tool with full parameter and output tables |
| [Editing projects](editing.md) | Integrators / power users | The navigate → read → edit → verify loop, path addressing, authoring rules |
| [Exporting projects](exporting.md) | Everyone | Formats, delivery modes, export options, render polling |
| [Skill & resources](skill-and-resources.md) | Integrators | The motion-design craft skill, MCP resources, and the prompt |
| [Troubleshooting](troubleshooting.md) | Everyone | Connection issues, common errors, and what they mean |

## Requirements

- A **SVGator account** ([sign up](https://app.svgator.com)). Exports count against your plan's
  limits exactly as they do in the editor.
- To connect from **claude.ai / Claude Desktop**, custom connectors require a Claude **Pro, Max,
  Team, or Enterprise** plan. Claude Code and other MCP clients connect directly to the endpoint.

## How it works (one paragraph)

The server is a remote MCP server: your MCP client (Claude, or any client speaking Streamable
HTTP) connects to `https://mcp.svgator.com/mcp` and signs you in through SVGator's regular OAuth
login. Your SVGator credentials stay **server-side only** — the AI model never sees your access
token or secret key; it only receives the tool results (project lists, structure maps, exported
files). Tool inputs are validated against published schemas, and invalid requests return
descriptive errors the assistant can read and correct.


## ⚙️ Capabilities

The MCP server turns SVGator into something you can drive with plain language. Instead of
clicking through the editor, you describe what you want and your AI assistant does it: pulling up
a project, changing a color, animating a logo, or exporting a finished file.

It's built for designers who want an assistant to handle the repetitive parts of animating, for
marketing teams who need web-ready animations without waiting on a designer for every small
change, and for developers who want production-ready exports (Lottie, React Native, Flutter, or
SVG) that drop straight into a product. Every project comes back with a direct editor link, so you
can always open it and finish by hand, saving tokens.

Here's what your assistant can do once it's connected:

### 🗂️ Projects and Assets

- List projects: newest first, each with a direct editor link
- Search and page through large libraries
- Read project settings: canvas size, background, timeline, and export settings
- Create a project from a full definition
- Overwrite an existing project
- Duplicate a project: into the same or another folder
- Move or rename a project
- Organize projects into folders
- Manage reusable assets: list, read, create, and update

### 🔍 Structure and Inspection

- Map a project's structure: element tree with keyframe counts, no heavy data
- See what's animated: which channels carry keyframes on each element
- Read exact values: any property, keyframe, or element
- Look up authoring shapes: the exact JSON for any element or concept

### 🔷 Shapes and Elements

- Rectangles
- Circles and ellipses
- Lines, polylines, and polygons
- Custom paths
- Text and multi-line text
- Images
- Reused symbols and instances
- Groups
- Masks and clip paths

### 🎨 Fills, Strokes, and Color

- Solid color fills: static or animated
- Gradient fills
- Strokes: solid or gradient, static or animated
- Compositing: blend modes and opacity
- Filter effects such as blur

### 🎞️ Animation and Keyframes

- Add keyframes at specific times, with values
- Set static values where no motion is needed
- Read, inspect, and remove keyframes
- Set easing per keyframe: cubic bezier
- Animate transforms: position, rotation, scale, and skew
- Animate fill, stroke, compositing, and filter channels
- Morph one shape into another

### 📤 Export and Delivery

- Export animated SVG: driven by JavaScript or CSS
- Set a start trigger: load, hover, click, scroll, or programmatic
- Set click and hover behavior: freeze, restart, reverse, or reset
- Export to Lottie
- Export to React Native or Flutter source
- Render video: MP4, MOV, AVI, MKV, or WebM
- Render GIF, WebP, PNG sequence, or ZIP
- Control looping: iterations, direction, ping-pong, and speed
- Set frame rate, resolution, and quality
- Handle transparency and background color

## ✅ Best practices

**Install the SVGator motion design skill**, then call it by name: It is craft guidance that helps your assistant make deliberate choices about timing, easing, and personality instead of flat, generic motion. Download it from the [SVGator skills repository](https://github.com/SVGator/skills), then trigger it right in your prompt by starting with something like "use SVGator motion skills to animate this logo." You call it manually on purpose: that way the skill stays out of your token usage when you are only exporting or making small edits, and loads only when you actually want the extra art direction.

**Start from your own artwork, then animate:** Generating original shapes from scratch is the hardest thing to get right in any AI assistant. For the best results, build or bring in your SVG in SVGator’s editor first, then ask your assistant to animate it. That keeps your visual style consistent and lets the AI focus on what it does best: motion, timing, and sequencing.

**Name your elements before you prompt:** Your assistant reads element names to understand a project’s structure. Clear names like “left_arm”, “bg_circle”, or “headline” help it target the right element, avoid mistakes, and stay organized across complex scenes. Generic names make its job harder, and might end up costing you more tokens (depending on model .

**Break complex animations into steps:** Instead of “build me a full onboarding animation,” go one piece at a time: create the elements, add keyframes, then refine the timing. The server applies one validated change per edit, so smaller steps give you more control and make any mistake easy to catch and correct.

**Be specific about timing and feel:** “A smooth fade” is on the vague side. “A 400ms fade-in with ease-out” gives your assistant something concrete to work with. The more you describe the feel, whether snappy, bouncy, slow, or subtle, the closer the first result lands to what you had in mind.

**Match the export format to where it will live:** SVG is the right pick for the web, with your choice of JavaScript or CSS-only (for even better performance). Lottie, React Native, and Flutter hand off cleanly to app and mobile projects. Video and GIF are ready for social and email, where an animated SVG will not play. Choosing the format up front saves a round of re-exporting.

**Duplicate a project before big experiments:** If you want the AI to rebuild something from the ground up, ask it to duplicate the project first, then make the changes on the copy. The original stays exactly as it was, so a bold experiment never costs you the working version.

**Finish in the editor:** Every project comes back with a direct editor link. Once the assistant gets you most of the way there, open the project and handle the final polish by hand. The AI does the heavy lifting, and you keep control of the details. It also saves tokens, since small tweaks by hand are quicker and cheaper than having the assistant make each one.

