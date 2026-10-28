# Getting started

The SVGator MCP server runs at **`https://mcp.svgator.com/mcp`**. You connect an MCP client to
that URL and sign in with your SVGator account; after that, the SVGator tools are available in
your conversations.

**Jump to your client:** [Claude](#claude) · [Claude Code](#claude-code) ·
[ChatGPT](#chatgpt) · [Gemini](#gemini) · [Grok](#grok) · [Cursor](#cursor) ·
[VS Code](#vs-code) · [Any other MCP client](#any-other-mcp-client)

> **Signing in is always the same.** Whatever the client, you enter only the server URL —
> `https://mcp.svgator.com/mcp` — and the client opens a browser window for SVGator's normal
> sign-in. **Never type an SVGator password, API key or bearer token into a client's
> configuration:** the server doesn't accept them, and anything asking for one isn't this server.

Once connected, ask the assistant to *"list my SVGator projects"* to confirm it works.

Client plan and availability details below were checked on **6 October 2026** — these products
move quickly, so follow each client's own documentation if its UI has changed.

## Claude

Custom connectors are available on Claude **Pro, Max, Team, and Enterprise** plans.

SVGator is listed in Claude's connector directory, which is the shortest route:

1. Open **Settings → Connectors**.
2. Type `SVGator` in the search box — SVGator's community MCP server appears in the results.
3. Click the result.

<img src="images/claude-step1.jpeg" width="600" alt="Searching for SVGator in Claude's connector settings">

4. On the SVGator community page, click **Connect to Claude**.

<img src="images/claude-step2.jpeg" width="600" alt="The SVGator connector page with the Connect to Claude button">

5. Authorize the access on SVGator's authorization page.

**Or add it manually** — same result, useful if the directory listing isn't available to you:

1. **Settings → Connectors → Add custom connector**.
2. **Name:** `SVGator` · **URL:** `https://mcp.svgator.com/mcp`.
3. Click **Add / Connect**. Claude registers itself with the server and opens a browser window to
   sign in to SVGator and approve access.
4. The connector shows **Connected** and the SVGator tools become available in your chats.

## Claude Code

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

## ChatGPT

Requires developer mode / custom MCP apps — see
[OpenAI's documentation](https://help.openai.com/en/articles/12584461-developer-mode-and-mcp-apps-in-chatgpt).

1. Select **Plugins**, click **Add**, then **Add custom MCP**.

<img src="images/chatgpt-step1.jpeg" width="600" alt="Adding a custom MCP in ChatGPT">

2. Fill in the connection details and click **Create as a plugin**:

   - **Name:** `SVGator`
   - **Connection:** `https://mcp.svgator.com/mcp`
   - **Authentication:** OAuth

<img src="images/chatgpt-step2.jpeg" width="600" alt="The ChatGPT custom MCP form filled in for SVGator">

3. Click **Connect SVGator** and complete the authorization on SVGator's side.

<img src="images/chatgpt-step3.jpeg" width="600" alt="Connecting and authorizing the SVGator plugin">

4. In a new chat, select **SVGator** from the tools menu.

## Gemini

See [Gemini custom Connected Apps](https://support.google.com/gemini/answer/17209137) and
[availability](https://support.google.com/gemini/answer/17094507).

1. At [gemini.google.com](https://gemini.google.com), go to **Customise → Connectors** and click
   **Custom**.

<img src="images/gemini-step1.jpeg" width="600" alt="The Custom connector option in Gemini">

2. Under **Connect to an MCP server**, enter `https://mcp.svgator.com/mcp` and click **Next**.

<img src="images/gemini-step2.jpeg" width="600" alt="Entering the SVGator MCP server URL in Gemini">

3. Accept the security & privacy notice and click **Connect**.

<img src="images/gemini-step3.jpeg" width="600" alt="Gemini's security and privacy confirmation">

4. Agree and complete the connection.

<img src="images/gemini-step4a.jpeg" width="420" alt="Gemini connection confirmation, first step">
<img src="images/gemini-step4b.jpeg" width="420" alt="Gemini connection confirmation, second step">

## Grok

See [Grok Connectors](https://x.ai/news/grok-connectors) and
[connector management](https://docs.x.ai/grok/connector-management).

1. On Grok web, go to **Plugins** and select **New Connector** (top right).

<img src="images/grok-step1.jpeg" width="600" alt="Creating a new connector in Grok">

2. Choose the **Custom** option.

<img src="images/grok-step2.jpeg" width="600" alt="Selecting the Custom connector type in Grok">

3. Fill in the name and server URL:

   - **Name:** `SVGator`
   - **Server URL:** `https://mcp.svgator.com/mcp`

<img src="images/grok-step3.jpeg" width="600" alt="The Grok custom connector form filled in for SVGator">

## Cursor

See the [Cursor MCP guide](https://docs.cursor.com/context/model-context-protocol).

1. Click **Customize**, open the **MCPs** tab, then click **New MCP Server**.
2. Add the configuration and save (`Cmd/Ctrl + S`):

```json
{
  "mcpServers": {
    "svgator": {
      "url": "https://mcp.svgator.com/mcp"
    }
  }
}
```

3. SVGator appears in the MCP list — click **Authenticate**, then authorize Cursor's access to
   your SVGator account.

<img src="images/cursor-step1.jpeg" width="600" alt="The SVGator entry in Cursor's MCP list with the Authenticate button">

## VS Code

See the [VS Code MCP guide](https://code.visualstudio.com/docs/agent-customization/mcp-servers)
and the [Copilot requirements](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/customize-copilot/extend-copilot-with-tools-and-context/extend-copilot-chat-with-mcp?tool=vscode).

1. Run **MCP: Add Server** from the Command Palette.

<img src="images/vscode-step1.jpeg" width="600" alt="Running MCP: Add Server from the VS Code command palette">

2. Choose **HTTP** and enter `https://mcp.svgator.com/mcp`.

<img src="images/vscode-step2.jpeg" width="320" alt="Choosing the HTTP server type">
<img src="images/vscode-step3.jpeg" width="320" alt="Entering the SVGator endpoint">
<img src="images/vscode-step4.jpeg" width="320" alt="Confirming the server name">

3. The first time the MCP is used you'll be asked to authenticate — or authenticate through
   Copilot.

## Any other MCP client

Use the endpoint in any client that supports **remote Streamable HTTP** MCP servers and **OAuth
2.1** with Dynamic Client Registration and PKCE. The client must support MCP **tools**; resources
and prompts are optional (without them, the motion-design craft skill is still reachable through
the [`get_motion_skill`](tools.md#get_motion_skill) tool).

Two configuration shapes cover most clients:

```json
{
  "mcpServers": {
    "svgator": { "url": "https://mcp.svgator.com/mcp" }
  }
}
```

```json
{
  "servers": {
    "svgator": {
      "type": "http",
      "url": "https://mcp.svgator.com/mcp"
    }
  }
}
```

**What can't connect:** clients limited to `stdio`, legacy SSE, or static API-key headers, and
any client without browser-based OAuth. Client plans and organization policies may also block
remote or write-capable MCP servers.

## Protocol details (integrators)

The server speaks the **Streamable HTTP** transport at `POST /mcp` and advertises the `tools`,
`resources`, and `prompts` capabilities.

Authentication follows the standard MCP authorization flow:

- **OAuth 2.1** with **Dynamic Client Registration** and **PKCE**. The server publishes standard
  OAuth discovery metadata (`/.well-known/…`), so any spec-compliant MCP client can register,
  send the user through the SVGator sign-in, and obtain a bearer token without manual setup.
- Every MCP request (except the OAuth endpoints themselves) requires the resulting
  **`Authorization: Bearer <token>`** header. Access tokens are short-lived (about an hour) and
  are refreshed automatically via the refresh-token grant.
- Three tools work without any account context once a session exists: `ping` (connectivity
  diagnostic), `describe_shape` (static authoring reference) and `list_templates` (SVGator's
  public template catalogue).

On `initialize`, the server also returns an `instructions` string — a compact motion-design
router that primes the model to apply animation craft when authoring (see
[Skill & resources](skill-and-resources.md)). Clients that surface server instructions to the
model get better animation output for free; clients that don't can reach the same material
through the [`get_motion_skill`](tools.md#get_motion_skill) tool.

See the [MCP authorization specification](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)
for the full flow.

## First things to try

- *"List my SVGator projects"* — returns your projects with direct editor links.
- *"Find me a SVGator template for a loading spinner and put a copy in my account"*
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
