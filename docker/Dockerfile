# =============================================================================
# nagoya-bus-sumo — minimal headless SUMO 1.19.0 runtime
# =============================================================================
# Copyright (c) 2026 Toyota Motor Corporation
# SPDX-License-Identifier: MIT
#
# Licensing: see LICENSE.md in the nagoya-bus-sumo repository. The MIT License
# covers this build file only, not the Eclipse SUMO runtime that the image
# installs (EPL 2.0) or the other dependencies pulled in with it.
#
# This image contains ONLY the SUMO 1.19.0 runtime. The scenario bundle (the
# repository's scenario/ directory) is NOT baked in — mount it at /scenario.
#
# The scenario is pinned to SUMO 1.19.0; the eclipse-sumo wheel provides exactly
# that version. The wheel is architecture-specific but Python-version agnostic,
# and pip picks the right one automatically, so this image builds on both
# linux/amd64 and linux/arm64. The base image is pinned by digest so that
# rebuilds stay reproducible.
#
# Build (from the repository root; the build context is docker/):
#   docker build -f docker/Dockerfile -t nagoya-bus-sumo:1.19.0 docker/
#
# Run the full 24-hour scenario (end=86400 from nagoya_bus.sumocfg):
#   docker run --rm -v "$PWD/scenario:/scenario:ro" nagoya-bus-sumo:1.19.0
#
# Append SUMO options after the image name to override or add outputs, e.g.
#   docker run --rm --user "$(id -u):$(id -g)" \
#       -v "$PWD/scenario:/scenario:ro" -v "$PWD/out:/out" \
#       nagoya-bus-sumo:1.19.0 --end 3600 --tripinfo-output /out/tripinfo.xml
#
# Output must go to an absolute /out/... path on a second, writable mount:
# /scenario is meant to be mounted read-only, and /out is deliberately NOT
# created in the image so a forgotten mount fails loudly. Use --user to have
# the generated files owned by you instead of root.
FROM python:3.11-slim@sha256:9c900dea9e8fb7e16277c179b555cc72d29a352dbc33cff48ad5a0412fd5bfc7

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libx11-6 libxext6 libxrender1 libgl1 \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --no-cache-dir eclipse-sumo==1.19.0

# The `sumo` console script sets SUMO_HOME itself; set it here too so that SUMO's
# own tool scripts (e.g. tools/xml/xml2csv.py, which imports sumolib through it)
# and TraCI clients also work when invoked directly in this image.
ENV SUMO_HOME=/usr/local/lib/python3.11/site-packages/sumo

WORKDIR /scenario

ENTRYPOINT ["sumo", "-c", "nagoya_bus.sumocfg"]
