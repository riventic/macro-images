# macro-images

**Unofficial** container images of [Macro](https://github.com/macro-inc/macro) (`macro-inc/macro`), built from upstream release tags on GitHub-hosted runners and published to `ghcr.io/riventic/macro-*`.

Not affiliated with or endorsed by Macro. Upstream does not publish server images yet ([macro-inc/macro#6447](https://github.com/macro-inc/macro/issues/6447)). Delete this repo once official images exist.

## What this repo holds

- Build workflows and Dockerfiles only. No Macro source is copied here — each build checks out `macro-inc/macro` at an exact release tag.
- Every image carries the label `org.opencontainers.image.source` + `org.opencontainers.image.revision` naming the upstream tag and commit it was built from (AGPL-3.0 §6: corresponding source).

## Images

All tagged with the upstream release tag (`v2026.10.26.1`, …). Pin by digest.

| Image | Content | Built from |
|---|---|---|
| `macro-services` | every Rust service binary in `/app/out/`; run one per container with `command: ["/app/out/<binary>"]` | `cargo build --release` on the runner + `images/services/Dockerfile` |
| `macro-web` | web app bundle, served by nginx under `/app/` on `:8080` | `bun run build` + `images/web/Dockerfile` |
| `macro-websocket-service` | websocket service | upstream `docker/websocket-service.Dockerfile`, unchanged |
| `macro-sync-service` | sync service (Rust → wasm, run by wrangler/workerd) | upstream `docker/sync-service.Dockerfile`, unchanged |
| `macro-lexical-service` | lexical document service | upstream `docker/lexical-service.Dockerfile`, unchanged |
| `macro-init` | one-shot setup: `migrate`, `opensearch`, `kafka-topics`, `version` (see `macro-init help`) | `sqlx-cli` 0.8.6 + upstream files at the tag + `images/init/Dockerfile` |
| `macro-opensearch` | OpenSearch 3.5.0 with the `analysis-icu` plugin (upstream `infra/local/opensearch`) | `images/opensearch/Dockerfile` |

`macro-init` contents: `/macro/migrations` (`crates/macro_db_client/migrations`), `/macro/opensearch-helpers` (`infra/stacks/opensearch/helpers`), `/macro/kafka-topics.json` (`.github/kafka-cluster-topics.json`), `/macro/fusionauth/templates` (`infra/stacks/fusionauth-instance/templates`). It does not create Kafka topics, S3 buckets, SQS queues, DynamoDB tables, KMS keys or the FusionAuth kickstart: those depend on the backends a deployment chooses. Upstream's list of AWS resources: `tooling/xtask/crates/xtask_local/src/local/resources.rs` + `localstack.rs`.

Binaries in `macro-services`: `agent_harness_service`, `authentication_service`, `calendar_service`, `connection_gateway_service`, `contacts_service`, `document_storage_service`, `document_upload_finalizer_local_worker`, `email_service`, `image_proxy_service`, `notification_service`, `pubsub_workers`, `search_processing_service`, `slack_import_worker`, `static_file_service`, `unfurl_service`. Production features only — upstream's local-only `return_passwordless_code` / `no_rate_limit` are not enabled.

## How a build runs

`.github/workflows/build.yml` — manual (`workflow_dispatch`, optional `tag`, `force`) or daily (latest upstream release). Images are built per group — `services`, `web`, `node` (the three upstream Dockerfiles), `init`, `opensearch` — and a group is built only when one of its images is missing for the tag. `force` names groups to rebuild anyway (`init,web`, or `all`).

| Job | Token | Does |
|---|---|---|
| `resolve` | `packages: read` | tag → commit SHA; lists the groups to build |
| `services-build` | `contents: read` | compiles the binaries, uploads an artifact |
| `services-image` | `packages: write` | packages the artifact, checks every binary's shared libraries, pushes |
| `node-images` | `packages: write` | builds upstream Dockerfiles in BuildKit, pushes |
| `web-build` | `contents: read` | builds the bundle, uploads an artifact |
| `web-image` | `packages: write` | packages the artifact, pushes |
| `init-build` | `contents: read` | builds `sqlx-cli`, stages upstream files, uploads an artifact |
| `init-image` | `packages: write` | packages the artifact, runs the migrations twice against an empty Postgres, pushes |
| `opensearch-image` | `packages: write` | builds stock OpenSearch + `analysis-icu`, checks the plugin is listed, pushes (runs no upstream Macro code) |

Jobs that run upstream code (`build.rs`, install scripts) never hold a write token. Write jobs run upstream code only inside BuildKit containers, which cannot read the runner's registry login.

## Not built yet

| Component | Why |
|---|---|
| `ai-editing-worker`, `analytics-proxy` | upstream Dockerfiles are dev-only: they bind-mount the source tree at runtime. Need a self-contained Dockerfile. |
| `convert_service` | needs LibreOffice bindings; not in upstream's local stack |
| `document_cognition_service`, `search_upload_worker`, `scheduled_action_service`, agent trigger | not in upstream's local stack inventory; check how production runs them |

## Open points

- `macro-web` uses upstream's static-build env (`MODE=development`, `VITE_LOCAL_BACKEND_ORIGIN=same-origin`), which keeps `import.meta.env.DEV` true in the bundle. Check which dev-only paths that enables before production use.
- `agent_harness_service` sandboxes: upstream's local stack runs them on the host Docker daemon (docker socket). This image has no docker CLI on purpose; use a remote sandbox provider.

## Rules

| Rule | Why |
|---|---|
| GitHub-hosted runners only — never self-hosted | public repo: its workflows must never run on Riventic infrastructure |
| No repository secrets; push with `GITHUB_TOKEN` (`packages: write`) | nothing to leak |
| No `pull_request_target` | fork PRs must never get write access |
| Third-party actions pinned to a full commit SHA | tags are mutable |
| Build only upstream release tags, never `main` | reproducible, traceable to a release |
| Consumers pin images by digest | a tag can be re-pushed |

## License

AGPL-3.0, same as upstream Macro.
