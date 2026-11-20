<img src="assets/icon.svg" width="64" height="64" alt="">

# skills

Created in [T3 Code](https://t3.codes).

## Install the skills and MCP servers

Install the `dokploy-deploy` and `hetzner-deploy` skills in the coding agent's
skill directory so the agent can load their instructions when handling
deployment and infrastructure tasks. For Codex, that directory is
`~/.codex/skills/`; install each skill as its own folder containing its
`SKILL.md` and any referenced files. Keep the skill folder names
`dokploy-deploy` and `hetzner-deploy` so their names and descriptions can be
discovered by the agent. Restart or reload the agent if it does not discover a
newly installed skill.

The skill source directories must include the complete skill contents,
including references. Do not copy only the main `SKILL.md` when a skill links
to supporting files. During setup, ask the user for their Hetzner and
Cloudflare API keys if they have not already configured them. Add each key to
the selected coding agent's global secret store (Codex, Claude Code, or the
equivalent for the chosen agent), using the key names and access method
required by its CLI or MCP configuration. Do not ask the user to paste keys
into ordinary chat, and do not put credentials in this repository, skill
files, MCP configuration committed to source control, shell arguments, or
logs. If the agent provides a secure secret-entry flow, use it; otherwise
explain how the user can enter the value directly in the agent's global secret
store without disclosing it to the conversation.

Configure a Cloudflare MCP server in the coding agent's MCP/server settings
using the official or otherwise trusted Cloudflare MCP package and its current
setup instructions. Do not guess a package name, command, or authentication
method. Ask the user to choose or provide the Cloudflare account/zone context
when a task needs it, and use only the permissions required for that task.
Never commit MCP credentials or tokens.

### PostHog skills and MCP

Use PostHog's [AI plugins and skills](https://posthog.com/docs/ai/skills) for
PostHog-specific workflows. Install the PostHog plugin using the current
first-party instructions for the agent in use; the plugin supplies PostHog
skills and keeps them current. For a standalone skill install, use the
PostHog-maintained skill source and copy the complete skill folder (including
referenced files) into the agent's skill directory. Confirm the skill's
frontmatter name and description remain intact, then reload the agent if
needed.

Install the PostHog MCP using the [official MCP setup guide](https://posthog.com/docs/model-context-protocol).
The PostHog wizard can configure supported agents with:

```sh
npx @posthog/wizard mcp add
```

For manual setup, use PostHog's hosted endpoint `https://mcp.posthog.com/mcp`
and follow the instructions for the specific agent. Authenticate with the
PostHog flow in that guide so it can route to the correct region. Never place
PostHog tokens in this repository or committed MCP configuration.

### Stripe skills and MCP

Prefer Stripe's [official agent setup](https://docs.stripe.com/mcp), which
installs Stripe's agent skills and configures supported agents' MCP settings.
Install the Stripe CLI using the current instructions in that guide, then run:

```sh
stripe agent setup
```

For Codex-only manual MCP setup, Stripe documents:

```sh
codex mcp add stripe --url https://mcp.stripe.com
```

Then follow Stripe's authentication flow. Team and Enterprise accounts may
require an owner to enable the connector. If installing skills separately,
use Stripe's official skill source and copy full skill folders into the
agent's skill directory; keep their metadata and supporting files intact.
Never commit Stripe credentials or tokens. Refresh the first-party setup
instructions before installation because agent support and commands can
change.

### Tavily skills and MCP

Install the Tavily agent skills from the Tavily-maintained repository with
the [Skills CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add tavily-ai/skills
```

Install the full skill folders, including any referenced files. Choose the
skills that match the work: `tavily-search` for web research and page
extraction, `tavily-research` for in-depth research, `tavily-monitor` for
recurring monitoring, `tavily-list` or `tavily-entity` for finding or
enriching records, and `tavily-sdk` for building Tavily integrations. Reload
the agent if it does not discover the installed skills.

For Codex, Tavily documents this hosted MCP setup and OAuth login:

```sh
codex mcp add tavily --url https://mcp.tavily.com/mcp/
codex mcp login tavily
```

The MCP endpoint is `https://mcp.tavily.com/mcp`. Follow Tavily's
[MCP setup guide](https://github.com/tavily-ai/tavily-mcp) for other clients.
Use OAuth where supported; otherwise provide the API key through the client's
protected secret or header mechanism. Never place API keys in this repository,
committed configuration, shell arguments, or logs. Restart the client after
adding the server if it does not appear.

### Runtime details for agents

- When a task involves Dokploy, first ask the user for the URL of their
  Dokploy server (including its scheme, such as `https://`). Use that URL as
  the target for that task; do not assume `dokploy.flydexo.com` or silently
  reuse a URL from another context. Then follow the `dokploy-deploy` skill and
  its referenced guidance.
- When a task involves Hetzner Cloud, follow the `hetzner-deploy` skill. Use
  the user's configured `hcloud` context or `HCLOUD_TOKEN`; never expose a
  token in chat, repository files, shell arguments, or logs. If no credential
  is configured, collect it through a secure flow and store it in the agent's
  global secrets.
- For Cloudflare tasks, use the configured Cloudflare MCP after checking its
  available tools and schemas. Do not assume it can perform every Cloudflare
  operation. Ask for missing account or zone selection if multiple targets
  match, and verify the result through available read operations. If the MCP
  needs an API key and one is not configured, collect it securely and store it
  in the agent's global secrets.
- For PostHog tasks, load the relevant PostHog skill first and use the
  configured PostHog MCP after checking its available tools and schemas. Ask
  which project to use when the target is ambiguous. Use the PostHog OAuth
  flow; never expose or commit credentials.
- For Stripe tasks, load the relevant Stripe skill first and use the
  configured Stripe MCP after checking its available tools and schemas. Use
  Stripe's documented authentication flow and request only the access needed
  for the task. Never expose or commit credentials.
- For Tavily web research tasks, use the installed Tavily skill that matches
  the task and prefer an already configured Tavily MCP or CLI. Use the CLI
  only when needed by the task or requested. If authentication is required,
  use Tavily's guided sign-in or a protected secret channel; never expose or
  commit API keys.
