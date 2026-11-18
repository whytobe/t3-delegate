# Install T3 Delegate

Use this guide when the user asks you to install `t3-delegate` or enable
routing to T3. Work on the current bot or the agent profile the user selected.
Use the host's supported settings, tool schemas, and approval flow.

## 1. Get the published skill

Download the files for your host from this pinned snapshot of the public
`whytobe/t3-delegate` repository. The files are readable without a GitHub
account.

- [SKILL.md](https://raw.githubusercontent.com/whytobe/t3-delegate/1d6c34528ceb642d137d3f2e367517f23771340d/SKILL.md)
- [Installation and routing guide](https://raw.githubusercontent.com/whytobe/t3-delegate/1d6c34528ceb642d137d3f2e367517f23771340d/references/install.md)
- [Repository task brief](https://raw.githubusercontent.com/whytobe/t3-delegate/1d6c34528ceb642d137d3f2e367517f23771340d/references/brief.md)
- [Self-contained skill for Rakazo and other single-body hosts](https://raw.githubusercontent.com/whytobe/t3-delegate/1d6c34528ceb642d137d3f2e367517f23771340d/dist/rakazo.md)

Read the installation and routing guide before changing settings. For a
directory install, save the first three files in a temporary `t3-delegate/`
directory as `SKILL.md`, `references/install.md`, and `references/brief.md`.
For a single-body install, read the full self-contained file and submit it in
the format the host expects. If the host stores the title and description
separately, map the frontmatter to those fields and keep the complete body
including both appendices. Verify the installed content by reading it back.

## 2. Connect T3

Reuse an existing authorized T3 MCP connection when its tools answer. Otherwise
use the host's custom remote MCP setup with **Streamable HTTP over HTTPS and
OAuth**. Get the endpoint from T3's **Use T3 Code in another app (MCP)** flow or
from an existing connection. A hosted bot needs a reachable URL, such as the T3
Connect address. Ask the user to approve any new connection in the supported
sign-in flow. Keep credentials and private endpoints out of public files.

From the target agent, verify the connection with project listing and inspect
the available tools. A read-only connection can list projects. Starting a
thread requires the user's separate authorization to operate it.

[Official T3 guide for outside agents](https://github.com/pingdotgg/t3code/blob/main/docs/user/outside-agents.md)

## 3. Install for your host

### OpenClaw

Discover the current profile, target agent ID, and workspace. Check the running
CLI's help and install the staged directory using the verified profile or its
wrapper. For the default profile, omit `--profile PROFILE`.

```sh
openclaw --profile PROFILE skills install /absolute/path/to/t3-delegate --agent AGENT_ID
openclaw --profile PROFILE skills info t3-delegate --agent AGENT_ID --json
```

Merge the routing block from the installation guide into that agent's
always-loaded workspace `AGENTS.md`. Preserve existing rules and local edits.
Use the gateway's supported MCP setup when a connection needs to be added.

### Rakazo

Use the available skill and bot-instruction tools for the selected bot. Inspect
their schemas and read existing entries before creating or updating anything.
Use the self-contained skill body when companion files cannot be written.
Attach or enable the skill for the selected bot if the host requires it. Merge
the routing block into its persistent instructions and read back both changes.
Keep an existing working T3 connector.

### Other agents

Use the same skill with any host that exposes both an authorized T3 MCP
connection and a supported way to install instructions. If the host can use
T3 MCP and only supports instructions in chat, read the skill and brief for the
current task when the user asks to route it to T3. Report that this is a
session-only setup until fresh-session routing is verified.

- **Grok Bot** documents adding a custom connector with a Remote HTTPS URL.
  Verify that the selected account can authorize this T3 endpoint and expose
  its tools. [Grok Bot connector docs](https://docs.x.ai/grok-bot/computer-and-apps)
- **ChatGPT Dot** documents access to supported plugins and local skills on a
  connected computer. Verify which plugins and actions the dot can actually
  use before relying on a custom T3 MCP connection.
  [OpenAI Dot guide](https://help.openai.com/en/articles/20001530-getting-started-with-your-dot)
- **Meta Muse** documents a connector integration and review process. Verify
  what the account exposes. **Muse Code** separately documents Streamable
  HTTP MCP and OAuth in its settings.
  [Muse connector guidelines](https://platform.muse.ai/docs),
  [Muse Code MCP docs](https://dev.meta.ai/docs/muse-code/extending#-mcp-servers)

MCP gives an authorized client access to T3 tools. The host controls skill
installation, persistent instructions, tool permissions, and completion
notifications. If the host cannot connect to this endpoint through its
supported flow, report the exact missing capability.

## 4. Verify and report

After enabling automatic routing, use a new session on the target agent.
Give it a small, read-only task in an existing authorized repository without
mentioning T3 in the task itself. Tell it to make no edits, install nothing,
and send no external messages. Verify that it routes the work to T3 before
inspecting repository implementation locally. Read the completed result and
check one concrete claim against the repository. A working connector alone
only verifies connectivity.

Report the installed agent or profile, the source revision, whether the
connection was reused, and the actual T3 thread link. If the host cannot start
a fresh session itself, give the user the exact test prompt to run there and
label automatic routing unverified. Use the full installation guide for
questions, takeover, cross-turn resumption, and any other lifecycle checks
the user requests.
