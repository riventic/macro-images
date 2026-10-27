# syntax=docker/dockerfile:1
#
# The Macro web app as static files, served under /app/ (upstream's vite `base`).
# Built by the `web-build` job with VITE_LOCAL_BACKEND_ORIGIN=same-origin: the
# bundle calls /auth, /dss, /email, ... on the origin it is served from, so one
# image works on any hostname. The reverse proxy in front routes those paths to
# the services.
FROM nginxinc/nginx-unprivileged:1.29-alpine@sha256:0c79d56aee561a1d81c63f00eee5fb5fe29279560cdc55e91425133104c7fbe6

ARG MACRO_VERSION
ARG MACRO_REVISION
LABEL org.opencontainers.image.title="Macro web (unofficial)" \
      org.opencontainers.image.description="Macro web app bundle, built from macro-inc/macro ${MACRO_VERSION}" \
      org.opencontainers.image.source="https://github.com/riventic/macro-images" \
      org.opencontainers.image.url="https://github.com/macro-inc/macro/tree/${MACRO_VERSION}" \
      org.opencontainers.image.version="${MACRO_VERSION}" \
      org.opencontainers.image.revision="${MACRO_REVISION}" \
      org.opencontainers.image.licenses="AGPL-3.0-only"

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY dist/ /usr/share/nginx/html/app/

EXPOSE 8080
