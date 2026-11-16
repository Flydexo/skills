# Operations and troubleshooting

Use for status, logs, failures, restart/redeploy requests, and recovery. See [dokploy-api.md](dokploy-api.md#status-logs-and-acceptance-checks) for verified Compose status/log endpoints; discover Application-specific operations from the current schema rather than substituting a Compose ID.

## Observe before acting

Status and logs requests stay read-only. Resolve the project/environment/resource/server, then collect relevant deployment, container, and routing evidence. Use finite log tails or time ranges, redact sensitive lines, and avoid unbounded streams or unrelated services' logs.

Distinguish build/deployment logs from runtime logs. Record deployment IDs, timestamps, status, and returned container identities as needed. A healthy old container can coexist with a failed new build; a successful deployment record can coexist with an unhealthy new container.

## Follow the submitted deployment

- Capture its deployment ID when returned. Otherwise correlate history with the resource, submission time, and available source metadata. Report ambiguity if concurrent deployments cannot be distinguished.
- A queued acknowledgement is not completion. Use bounded status checks and a finite observation window appropriate to the build and execution budget. If still running, report the state and ID rather than claiming failure or success.
- The instance notes observed `running`, `done`, and `error`; inspect current schema/responses for other states. Unknown states are unverified, not successful.
- If submission times out, read status/history before repeating the write. A transport error does not prove the deploy failed or was never accepted.
- After `done`, check container health and public HTTPS separately. Exercise a safe principal workflow where practical; a successful build does not verify runtime behavior or delivered source assets.

## Find the failing layer

| Evidence | Inspect next |
| --- | --- |
| Authorization or target error | Instance, credential resolution/header, resource IDs, and server scope; stop writes while access is unresolved. |
| Build failure | This deployment's build logs, source revision, build context, lockfile, and build-time variables. |
| Exited or unhealthy container | Runtime logs, start command, listener/bind address, runtime variables, dependencies, and health check. |
| Healthy container, failing public URL | Domain service/port, routing labels, DNS, and edge/origin behavior. An initial Traefik 404 can clear after health becomes ready. |
| Redirect loop or insecure cookies | This instance's edge-TLS settings and the app's public-HTTPS/cookie configuration, not global proxy changes. |
| Wrong version or broken static assets | Source revision, image/build context, and served JS/CSS bytes; consult raw-source fidelity notes for embedded source. |

For a requested fix, correct the evidenced in-scope cause and verify again. If the same cause recurs without a new correction, stop retries and report the blocker. Do not reset volumes, prune the host, or change unrelated apps to force success.

## Restart, redeploy, and rollback

These actions have different effects. Inspect the available Dokploy operation's current schema and semantics before choosing one; do not promise that restarting builds new code or that redeploying restores prior configuration.

For rollback, identify the exact known-good image/revision and associated configuration changes. Check persistent-data and migration compatibility: restoring an image does not restore a database. Obtain confirmation for data-loss risks and verify the recovered deployment. If the installed version lacks a suitable rollback operation, report the limitation and propose explicit source/image redeployment rather than inventing an endpoint.

## Acceptance and handoff

Report deployment state, runtime health, and public reachability separately. Clean up disposable smoke-test sessions; do not change existing user data to prove success. State exactly which checks could not be performed.

Railway inspiration: [deployment verification](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/references/deploy.md) and [operations](https://github.com/railwayapp/railway-skills/blob/main/plugins/railway/skills/use-railway/references/operate.md). Railway status names and lifecycle commands are not Dokploy contracts.
