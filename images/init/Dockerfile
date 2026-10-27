# syntax=docker/dockerfile:1
#
# One-shot setup for a Macro deployment: database migrations, OpenSearch
# indices, and the upstream data a deployment's own provisioning needs
# (Kafka topic list, FusionAuth lambda/email templates).
#
# Built by the `init-build` job, which stages `macro/` (upstream files at the
# release tag) and `bin/sqlx` into the build context. Nothing is fetched here.
#
# Usage: see `macro-init help`.
FROM oven/bun:1.3.5-slim@sha256:6111acec4c5a703f2069d6e681967c047920ff2883e7e5a5e64f4ac95ddeb27f

ARG MACRO_VERSION
ARG MACRO_REVISION
LABEL org.opencontainers.image.title="Macro init (unofficial)" \
      org.opencontainers.image.description="Macro migrations and setup, from macro-inc/macro ${MACRO_VERSION}" \
      org.opencontainers.image.source="https://github.com/riventic/macro-images" \
      org.opencontainers.image.url="https://github.com/macro-inc/macro/tree/${MACRO_VERSION}" \
      org.opencontainers.image.version="${MACRO_VERSION}" \
      org.opencontainers.image.revision="${MACRO_REVISION}" \
      org.opencontainers.image.licenses="AGPL-3.0-only"

RUN apt-get update \
  && apt-get install -y --no-install-recommends ca-certificates \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

COPY --chmod=0755 bin/sqlx /usr/local/bin/sqlx
COPY --chmod=0755 macro-init /usr/local/bin/macro-init
COPY macro/ /macro/

ENV MACRO_VERSION=${MACRO_VERSION}
USER bun
WORKDIR /macro
ENTRYPOINT ["macro-init"]
CMD ["help"]
