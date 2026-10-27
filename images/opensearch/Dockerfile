# OpenSearch with the analysis-icu plugin, as upstream infra/local/opensearch.
# The channels index mapping uses an `icu_folding` filter from analysis-icu,
# which the stock image does not ship. Installing it at build time keeps the
# container start offline. Version 3.5 matches upstream's AWS domains.
FROM opensearchproject/opensearch:3.5.0@sha256:dbb01641baadae5104e18acd888bf05e8fdd9af3567fd30624a76ba3e5a31dec
RUN /usr/share/opensearch/bin/opensearch-plugin install --batch analysis-icu
