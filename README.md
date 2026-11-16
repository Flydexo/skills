<img src="assets/icon.svg" width="64" height="64" alt="">

# skills

Created in [T3 Code](https://t3.codes).

## Install the deployment skills and Cloudflare MCP

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
to supporting files. Keep credentials out of this repository and out of skill
files; configure them through the relevant local secret store or environment.

Configure a Cloudflare MCP server in the coding agent's MCP/server settings
using the official or otherwise trusted Cloudflare MCP package and its current
setup instructions. Do not guess a package name, command, or authentication
method. Ask the user to choose or provide the Cloudflare account/zone context
when a task needs it, and use only the permissions required for that task.
Never commit MCP credentials or tokens.

### Runtime details for agents

- When a task involves Dokploy, first ask the user for the URL of their
  Dokploy server (including its scheme, such as `https://`). Use that URL as
  the target for that task; do not assume `dokploy.flydexo.com` or silently
  reuse a URL from another context. Then follow the `dokploy-deploy` skill and
  its referenced guidance.
- When a task involves Hetzner Cloud, follow the `hetzner-deploy` skill. Use
  the user's configured `hcloud` context or `HCLOUD_TOKEN`; never request or
  expose a token in chat, repository files, shell arguments, or logs.
- For Cloudflare tasks, use the configured Cloudflare MCP after checking its
  available tools and schemas. Do not assume it can perform every Cloudflare
  operation. Ask for missing account or zone selection if multiple targets
  match, and verify the result through available read operations.
