# syntax=docker/dockerfile:1
#
# Every Macro Rust service binary in one image. A deployment runs one service
# per container with `command: ["/app/out/<binary>"]` — the same layout as
# upstream's local stack (docker/Dockerfile.dev `services_bundle`).
#
# The binaries are compiled by the `services-build` job and staged into the
# build context as `out/` + `pdfium-lib/`. Nothing is compiled here.

# deno: only agent_harness_service runs it (code-mode subprocesses).
FROM denoland/deno:2.9.6@sha256:2014dc167ece617ef7e7ba40631ac2234c59e75ce693e7cc2dc2602b3c87859d AS deno

FROM debian:trixie-slim@sha256:62bd423aeee912639ca8ee681c2b762cb3e25137e0a92b20be455898a9f034ad

ARG MACRO_VERSION
ARG MACRO_REVISION
LABEL org.opencontainers.image.title="Macro services (unofficial)" \
      org.opencontainers.image.description="Macro Rust service binaries, built from macro-inc/macro ${MACRO_VERSION}" \
      org.opencontainers.image.source="https://github.com/riventic/macro-images" \
      org.opencontainers.image.url="https://github.com/macro-inc/macro/tree/${MACRO_VERSION}" \
      org.opencontainers.image.version="${MACRO_VERSION}" \
      org.opencontainers.image.revision="${MACRO_REVISION}" \
      org.opencontainers.image.licenses="AGPL-3.0-only"

# curl: upstream's compose healthchecks call it.
RUN apt-get update \
  && apt-get install -y --no-install-recommends ca-certificates curl dumb-init \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

COPY --from=deno /usr/bin/deno /usr/local/bin/deno
ENV DENO_BINARY=/usr/local/bin/deno

RUN groupadd --system --gid 10001 macro \
  && useradd --system --uid 10001 --gid macro --home-dir /app --shell /usr/sbin/nologin macro

WORKDIR /app
# Root-owned and world-executable: the service user runs them, cannot rewrite them.
COPY --chmod=0755 out/ /app/out/
# search_processing_service loads pdfium from ./pdfium-lib/linux (relative to /app).
COPY pdfium-lib/ /app/pdfium-lib/

ENV PORT=8080
EXPOSE 8080
USER macro

# No CMD: each deployment service sets `command: ["/app/out/<binary>"]`.
ENTRYPOINT ["dumb-init", "--"]
