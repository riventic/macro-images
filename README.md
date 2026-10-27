# macro-images

**Unofficial** container images of [Macro](https://github.com/macro-inc/macro) (`macro-inc/macro`), built from upstream release tags on GitHub-hosted runners and published to `ghcr.io/riventic/macro-*`.

Not affiliated with or endorsed by Macro. Upstream does not publish server images yet ([macro-inc/macro#6447](https://github.com/macro-inc/macro/issues/6447)). Delete this repo once official images exist.

## What this repo holds

- Build workflows and Dockerfiles only. No Macro source is copied here — each build checks out `macro-inc/macro` at an exact release tag.
- Every image carries the label `org.opencontainers.image.source` + `org.opencontainers.image.revision` naming the upstream tag and commit it was built from (AGPL-3.0 §6: corresponding source).

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
