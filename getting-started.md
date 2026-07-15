# Getting started

The SVGator MCP server runs at **`https://mcp.svgator.com/mcp`**. You connect an MCP client to
that URL and sign in with your SVGator account; after that, the SVGator tools are available in
your conversations.

## Connect from claude.ai or Claude Desktop

Custom connectors are available on Claude **Pro, Max, Team, and Enterprise** plans.

1. Open **Settings → Connectors**.
2. Click **Add custom connector**.
3. Fill in:
   - **Name:** `SVGator`
   - **URL:** `https://mcp.svgator.com/mcp`
4. Click **Add / Connect**. Claude registers itself with the server and opens a browser window to
   **sign in to SVGator** and approve access.
5. After you approve, the connector shows **Connected** and the SVGator tools become available in
   your chats.

## Connect from Claude Code

```bash
claude mcp add --transport http svgator https://mcp.svgator.com/mcp
```

Then run `/mcp` inside Claude Code and pick **svgator** to authenticate — a browser opens for the
SVGator sign-in and consent. Once connected, the tools (`list_projects`, `export_project`, …) are
available, the motion-design skill resources can be read, and the `/svgator-motion-design` prompt
is exposed as a slash command.

To disconnect:

```bash
claude mcp remove svgator
```

## Connect from other MCP clients (integrators)

The server speaks the **Streamable HTTP** transport at `POST /mcp` and advertises the `tools`,
`resources`, and `prompts` capabilities.

Authentication follows the standard MCP authorization flow:

- **OAuth 2.1** with **Dynamic Client Registration** and **PKCE**. The server publishes standard
  OAuth discovery metadata (`/.well-known/…`), so any spec-compliant MCP client can register,
  send the user through the SVGator sign-in, and obtain a bearer token without manual setup.
- Every MCP request (except the OAuth endpoints themselves) requires the resulting
  **`Authorization: Bearer <token>`** header. Access tokens are short-lived (about an hour) and
  are refreshed automatically via the refresh-token grant.
- Two tools work without any account context once a session exists: `ping` (connectivity
  diagnostic) and `describe_shape` (static authoring reference).

On `initialize`, the server also returns an `instructions` string — a compact motion-design
router that primes the model to apply animation craft when authoring (see
[Skill & resources](skill-and-resources.md)). Clients that surface server instructions to the
model get better animation output for free.

## First things to try

- *"List my SVGator projects"* — returns your projects with direct editor links.
- *"What's inside project X? Which parts are animated?"*
- *"Make the heading in project X fade in instead of sliding"*
- *"Create a new project: a bouncing ball with a soft shadow"*
- *"Export project X as an animated SVG that plays on hover"*
- *"Export project X as a GIF with a transparent background"*

## Privacy & security

- Sign-in uses SVGator's normal OAuth login. Your SVGator credentials (access token, secret key)
  are held **server-side only** and are **never exposed** to the AI model or the MCP client.
- The model receives only tool results: project lists, structure summaries, values you ask it to
  read, and exported files or links.
- Exports count against your SVGator plan's limits, exactly as in the editor.
- Removing the connector (or `claude mcp remove`) revokes the client's access. Your SVGator
  account and projects are untouched.

## Next steps

- [Tools reference](tools.md) — everything each tool accepts and returns.
- [Editing projects](editing.md) — how the assistant edits safely (and how to talk to it about edits).
- [Exporting projects](exporting.md) — formats, options, and delivery modes.
