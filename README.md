# SVGator MCP Server — Documentation

The SVGator MCP server connects AI assistants to [SVGator](https://www.svgator.com), the online
SVG animation tool. Once connected, an assistant can **browse, inspect, edit, create, and export**
your SVGator projects on your behalf — from "list my projects" to "build me an animated logo
reveal and export it as a Lottie file".

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
