---
name: dokploy-deploy
description: Deploy and operate applications on the user's Dokploy server, including Compose deployments, configuration, domains, deployment status, logs, and troubleshooting. Use when Dokploy is named or the application is already hosted on this instance.
---

# Dokploy Deploy

Before a Dokploy task, ask the user for their Dokploy server URL, including the scheme (for example, `https://`). Use that URL for the task. Do not assume a default Dokploy host.

## Choose the task

Resolve the target first, then read only the references needed for the request. An inspection request is not permission to deploy or change configuration.

| Request | Guidance |
| --- | --- |
| Deploy a new app or update its source | Follow **Deploying an app** below and [API and deployment notes](references/dokploy-api.md). |
| Change variables, domains, source settings, or persistent storage | Read [Configuration changes](references/configure.md). |
| Check status, read logs, diagnose a failure, restart, or recover a release | Read [Operations and troubleshooting](references/operate.md). |
| Select API versus MCP, or diagnose tool access | Read [Tools and MCP](references/tools.md). |

For a compound request, inspect once, apply related changes together where supported, deploy only when needed, then verify the same target. Borrow Railway's workflow, not its CLI commands or resource semantics.

## Resolve the target

- Inspect local deployment files and relevant Dokploy inventory before creating resources. A folder name alone is not a remote project binding.
- Resolve the project, environment, resource type (`compose` or `application`), resource ID, and server when applicable. A Compose stack can contain several services; identify the requested service within it. Use returned IDs and the actual generated `appName`.
- Prefer an existing matching resource. Preserve its deployment method and environment rather than converting an existing Application to Compose merely because Compose is the default for new apps.
- If several resources plausibly match and a write could affect the wrong app or environment, ask one focused question before changing anything. A read-only request can report the candidates without choosing one.
- Carry the resolved IDs through configuration, deployment, and log calls. Do not silently fall back to a default environment or another server.

## Deployment defaults

- HTTPS is terminated by a reverse proxy in front of Dokploy. Do not configure TLS certificates or HTTPS termination in the app or Dokploy as part of a deployment; configure the app domain and its normal HTTP/internal listener and let the upstream proxy handle HTTPS.
- For this setup, configure Dokploy domains with `https: false` and `certificateType: "none"`, while reporting and verifying the public `https://` URL. Do not change global proxy/Cloudflare SSL settings to fix an individual deployment. Applications must still issue Secure cookies in production; the origin's forwarded protocol may be HTTP.
- Unless the user specifies another deployment method, deploy new apps from a Docker Compose file.
- Unless the user specifies a domain, use `<project-name>.flydexo.com`. Choose a concise, DNS-safe project name based on the app; check for an existing project/domain before creating a conflict.
- Respect any explicit deployment method, domain, project naming, or infrastructure choices in the request.

## Credentials and tools

- The Dokploy API key is stored in the Codex global config at `~/.codex/secrets.env`. Read the file locally only when an API operation needs the key. Identify the relevant variable name without printing the file or key; keep the value in a process environment or another protected input channel.
- The current variable is `DOKPLOY_API_TOKEN`. Load the trusted shell environment file rather than parsing its assignment as a literal string: it can resolve the token through command substitution. Send the resolved value in the `x-api-key` request header, never as a bearer-token assumption.
- Never put the API key in the repository, skill files, command arguments, logs, chat, or deployment configuration. Do not include it in a URL.
- Use Dokploy's API for routine management when available. Check the current API documentation or OpenAPI schema for the server/version before relying on endpoint names or payload formats; do not guess destructive or mutating calls.
- Read [API and deployment notes](references/dokploy-api.md) for the verified schema endpoint, project/Compose/domain payloads, command-prefix handling, and deployment/log inspection. These notes were checked against this server's v0.30.6 API; refresh the schema when the server changes.
- An already configured, suitable Dokploy MCP connection can be used when its tools support the operation and target this instance. Discover its actual schemas first; do not assume it mirrors Railway MCP or supports every Dokploy operation. See [Tools and MCP](references/tools.md). Updating this skill does not authorize installing packages or changing client/server configuration.

## Deploying an app

1. Inspect the project and identify its runtime, build process, ports, persistent data, required environment variables, and health endpoint. Reuse existing project conventions where present.
2. Prepare a production-ready Compose file for a new app, or make the smallest necessary update to the existing deployment method. Use clear service names, appropriate internal routing, persistent volumes only where needed, and health checks when supported. Never commit secrets; configure them through Dokploy's environment settings or another secret store.
   - Preserve Dokploy's generated command unless a change is necessary. Its `command` field omits the leading `docker`; use the actual returned `appName`, which can have a generated suffix.
   - For raw Compose with embedded source, see the reference's interpolation and payload-size guidance. Verify delivered JavaScript/CSS against local source; a successful image build does not prove source bytes survived export.
3. Configure the requested or default domain and make sure the service listens on the expected internal port. Do not assume the app's container port or framework; verify it from the project.
4. Deploy through Dokploy, then inspect deployment status and logs. Capture the returned deployment ID when available; otherwise correlate history to the resource, submission time, and available source metadata. Do not mistake an older success or concurrent deployment for this one. If submission times out, inspect history before resubmitting.
5. Do not claim success until Dokploy reports a successful deployment and the app is reachable when a suitable health check can be performed.
   - A queued response is not success, and `done` does not replace container health or public HTTPS checks. Smoke-test the app's main workflow with a disposable session and clean up test resources. An initial routing 404 can clear once the container becomes healthy.

## Changes and troubleshooting

For updates, inspect the current deployment and make the smallest change that accomplishes the request. Preserve existing volumes and data. Before a change that could delete a project, application, volume, or persistent data, explain the concrete impact and get the user's confirmation. For ordinary deploys and reversible configuration changes requested by the user, proceed and report the result.

Use [Configuration changes](references/configure.md) for variables, routing, and storage; use [Operations and troubleshooting](references/operate.md) for bounded log queries, failure classification, and recovery. Do not turn a status or logs request into an unsolicited fix. Stop repeated retries when no new corrective action is available.

If API access is unavailable, report the specific blocker and continue with a safe alternative only when it can perform the requested operation without exposing credentials or making an ambiguous change.

## Report the outcome

State the affected project/environment/service, what changed (or that nothing changed), the observed deployment and health state, and the public HTTPS URL when relevant. Distinguish queued, building, deployed, healthy, and unknown outcomes. Summarize useful evidence without secrets, and name any blocker or remaining user action. Do not report success solely because an API/MCP call returned successfully.

Source links for the Railway-inspired routing, context resolution, and verification patterns are in [Tools and MCP](references/tools.md).
