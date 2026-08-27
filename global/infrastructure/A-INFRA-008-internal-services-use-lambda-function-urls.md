# INFRA-008: Internal Services Use Lambda Function URLs, Not the Shared API Gateway

**Date:** 2026-08-26\
**Status:** Accepted — fully implemented 2026-08-27 (`ServicesApi` and `ServicesInfrastructure` stacks deleted: RestApi, custom domain, ACM cert, hosted zone, and the cross-account DNS delegation from `amateurradio.engineer` all removed)\
**Deciders:** Jean-Sébastien Dominique

## Context

INFRA-002 put every internal service behind one shared, path-routed API Gateway (`ServicesApi`) to avoid per-service custom-domain costs, with INFRA-003 giving each service ownership of its own route on that gateway. INFRA-001/INFRA-007 layered account-level IAM restrictions on top so any in-account Lambda could call any route without per-service role configuration.

In practice this added a layer of indirection every internal call paid for without buying much: one path-routed REST API in front of Lambdas that were all in the same AWS account already, all reachable directly. It also meant every new service needed its own route registered on the shared gateway (an `AWS::ApiGateway::Resource`/`Method`/`Permission` set, redeployed via a `CfnDeployment` trigger on the shared stage) before it could be called at all, and a service's route lived or died with the shared gateway's own health.

AWS Lambda Function URLs (GA since April 2022) give each Lambda its own HTTPS endpoint with `AWS_IAM` auth, no API Gateway resource in between. `lambda_http`, the Rust crate every service's handler already used, auto-detects the Function URL payload shape (`payload-format-2.0`) the same way it detects an API Gateway REST proxy event -- migrating a service's front door doesn't touch its handler code at all.

## Decision

Every internal service gets its own Lambda Function URL (`AWS_IAM` auth) instead of a route on the shared `ServicesApi` gateway. Callers sign requests with the `lambda` SigV4 service scope (`lambda:InvokeFunctionUrl` + `lambda:InvokeFunction`, both required since AWS's October 2025 policy change) instead of `execute-api`. The identity-based grant is account-wide on the *caller's* role (`grantInternalFunctionUrlInvoke` -- one grant covers every internal Function URL, no per-callee list to maintain), carrying forward INFRA-007's "account-level, zero per-service config" principle into the new mechanism. The callee's resource-based permission only ever needs `lambda:InvokeFunctionUrl`.

`ServicesApi` itself is retired -- no longer instantiated in the CDK app -- once every caller has migrated. The underlying hosted zone/custom-domain infrastructure (`ServicesInfrastructureStack`) is left standing rather than torn down in the same pass, since it owns real DNS delegation outside the scope of a caller migration; a future ADR can cover its removal if/when nothing references it.

This decision covers **internal service-to-service traffic only**. The public-facing `PublicApi` gateway (external webhook/API routes) is unaffected -- Regional endpoints (INFRA-006) still apply wherever an API Gateway is actually in use.

## Consequences

### Positive

- One fewer moving part per call: no shared gateway, no path routing, no per-service route registration/redeploy step
- A service is callable the moment its own stack deploys -- not gated on a route existing on a separate shared stack
- Eliminates the `ServicesApi` custom domain + Public Hosted Zone cost (~$1/month) and its DNS delegation chain
- No shared-gateway blast radius: one service's Function URL having an issue doesn't touch anyone else's
- `lambda_http` needed zero code changes -- Function URL payload-format-2.0 auto-detected alongside the existing API Gateway proxy event shape

### Negative

- Each service's endpoint is a distinct Function URL (`https://<id>.lambda-url.<region>.on.aws/`) rather than one predictable path under a shared custom domain -- callers read it from CDK stack outputs (`Fn::GetStackOutput`) instead of constructing a path
- `Fn::GetStackOutput` (used instead of `Fn.importValue`/CFN exports specifically to avoid the "export in use" redeploy deadlock) is invisible to CDK's automatic stack-dependency inference -- every cross-stack reference needs an explicit `addStackDependency`, or a first deploy of a new producer/consumer pair can attempt them out of order and fail (hit once, for a first-time deploy of the Guides service; fixed by wiring dependencies explicitly rather than relying on the target stack already existing)

### Neutral

- Service discovery shifts again, from path-based (INFRA-002) to Function-URL-output-based
- The Docker test harness's `http-logger` proxy (HTTP-001) keeps its path-based routing regardless -- it demuxes to the right backend container either way, and now also selects which RIE invocation event shape (`rie` vs `riefunctionurl`) to wrap a request in per service, mirroring whichever front door that service actually uses in prod

## Alternatives Considered

- **Keep the shared gateway, just fix route lifecycle issues:** Rejected -- doesn't remove the indirection or the per-service route-registration step that motivated the move in the first place
- **Private API Gateway per service:** Same VPC/NAT cost problem INFRA-001 already rejected, now for N services instead of one shared one
- **Direct Lambda `Invoke` (not HTTP):** Rejected for the same reason INFRA-001 originally rejected it -- doesn't fit the Smithy-generated client/server model already in use; Function URLs keep the HTTP interface Smithy expects

## Implementation Notes

- `addInternalFunctionUrl`/`grantInternalFunctionUrlInvoke` (`cdk/lib/lambda.ts`) are the shared helpers: the former adds the Function URL + minimal resource policy to a callee, the latter grants a caller both required SigV4 actions account-wide
- Migration was staged: add every service's Function URL first (dual-fronted alongside its existing gateway route), migrate callers one at a time, then remove each service's now-dead gateway route, then remove `ServicesApiStack` once nothing referenced it
- `jd_smithy_sigv4::SigV4Plugin` exposes a `.lambda()` builder to sign for the `lambda` service scope instead of the default `execute-api` one -- this is the only client-side code change any caller needed
- See the `riplomacy_bot` repo's own memory of this migration for the deploy-ordering gotcha and its fix (explicit `addStackDependency` for every `Fn::GetStackOutput` reference)
