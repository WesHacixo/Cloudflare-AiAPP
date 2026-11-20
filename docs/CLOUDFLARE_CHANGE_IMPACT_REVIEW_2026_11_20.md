# Cloudflare platform change-impact review — 2026-11-20

**Disposition:** research-only review for this VibeSDK-derived repository. No functional code, binding, model, credentials, infrastructure, or deployment changes. Validate actual package versions and account eligibility before implementing any proposal.

## Repository context

The existing application uses Workers, a Durable Object-based code-generation agent, D1, WebSockets, and an external runner/sandbox path. This review identifies recent Cloudflare changes that *may* affect its implementation. It does not assert that the present repository uses every new feature.

## Candidate impact map

| Recent change | Existing surface to inspect | Safety / migration considerations |
| --- | --- | --- |
| Durable Objects automatic retries (Nov 5) | Agent RPC and fetch calls, state transitions, session creation | Avoid duplicate tool executions or generation steps; do not label non-idempotent methods `retryable()`; test ambiguous acknowledgement and restart |
| Durable Object placement/routing/reliability (Nov 9/11/19) | Cold starts, connection latency, database operations | Performance improvement is not deterministic SLA; measure workload-specific latency and fault recovery |
| Browser Run /script (Nov 20), /snapshot expansion | Optional preview/evaluation instrumentation | Separate untrusted generated-code/browser execution from authenticated sessions; restrict egress and secrets |
| TCP/gRPC + Containers (Nov 19 beta) | External runner interface | Could offer typed native transport; requires explicit network admission, authz, Spectrum eligibility, flow control and cancellation |
| Workers VPC mTLS (Nov 18) | Private backend calls | Endpoint mutual authentication does not authorize application tool invocations |
| AI Gateway model aliases and provider-key activity | Model client and provider-key handling | Record actual resolved model, route, input/output policy, cost; last-used telemetry is not a complete credential audit |
| Workers AI planned model retirements (Nov 20) | Any pinned Workers AI model names | Audit exact model IDs before changes; retired models may return HTTP 410/error 5028 |
| Workers/Pages version retention (Nov 19) | Preview links and deployments | Do not rely on hosted previews for permanent build provenance; archive necessary source and receipts separately |
| Cloudflare CLI 1.0 release candidate | Tooling and CI | Preserve existing `wrangler.jsonc` + Wrangler; no CLI migration justified by changelog alone |

## Qualification cases (proposed, not executed)

1. **DO retry:** simulate disconnected client, request timeout, duplicate call, DO recycle; verify a single *effective* generation transition and complete attempt log. If effects cross services, enforce independent idempotency.
2. **Model route:** pin test model and alias; record gateway-resolved model, provider, cost and policy. Assert provenance is retained even after changing alias targets.
3. **Browser execution:** run a public synthetic HTML fixture in an isolated session; verify no cookies or secrets leak and no access to forbidden origins. Close sessions deliberately.
4. **Runner interface:** pure gRPC echo/hash service only; test invalid credentials, oversized frame, cancellation and failure recovery before contemplating code execution.
5. **Release reproducibility:** reconstruct old build from archived commit and lockfile rather than preview URL, with versioned runtime configuration.

## Security and adoption gates

- Repo security notes flag OAuth/session hardening and tests as unfinished; this review does not resolve them.
- Require explicit feature-level scope, owner approval, account-plan eligibility, test evidence, and rollback plan before implementation.
- Do not turn on remote observability/telemetry, introduce SaaS SDKs, or expose existing secrets as a side-effect of research.
- **No new `cf` CLI configuration**: the project already has `wrangler.jsonc`; follow current project tooling unless a separately scoped migration is approved.

## Official documentation

- https://developers.cloudflare.com/changelog/post/2026-11-05-durable-objects-automatic-retries/
- https://developers.cloudflare.com/changelog/post/2026-11-19-durable-objects-runtime-update-reliability/
- https://developers.cloudflare.com/changelog/post/2026-11-20-script-endpoint/
- https://developers.cloudflare.com/changelog/post/2026-11-19-inbound-tcp/
- https://developers.cloudflare.com/changelog/post/2026-11-18-custom-ca-mtls/
- https://developers.cloudflare.com/changelog/post/2026-11-04-model-aliases/
- https://developers.cloudflare.com/changelog/post/2026-11-06-provider-key-security/
- https://developers.cloudflare.com/changelog/product/workers/
- https://developers.cloudflare.com/changelog/post/2026-11-05-planned-model-deprecations/
