# workspace-base — source for the team's gold image.
#
# Structure mirrors the block model (see gold-server/glossary.md):
#   block00 region = Microsoft base + decomk bootstrap tooling
#   block0  region = decomk runs the workspace-config Makefile
#
# When we cut block00 as its own image, the first region moves upstream
# and this Dockerfile shrinks to ~2 lines.

# ---- block00 region --------------------------------------------------------
FROM mcr.microsoft.com/devcontainers/base:ubuntu@sha256:4bcb1b466771b1ba1ea110e2a27daea2f6093f9527fb75ee59703ec89b5561cb

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      golang-go=2:1.22~2build1 \
      git=1:2.43.0-1ubuntu7.3 \
      make=4.3-4.1build2 \
      ca-certificates=20240203 \
 && rm -rf /var/lib/apt/lists/*

# Alpha: @latest intentionally unpinned until a stable tag is cut.
# See TODO 026.2. Will pin to a specific tag/commit when available.
RUN go install github.com/stevegt/decomk/cmd/decomk@latest \
 && mv /root/go/bin/decomk /usr/local/bin/decomk

# Create dev user (uid 1000) with passwordless sudo, matching decomk convention.
# The Microsoft base image creates a vscode user at uid 1000; replace it.
RUN userdel -r vscode 2>/dev/null || true \
 && useradd --create-home --shell /bin/bash --uid 1000 dev \
 && echo 'dev ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/dev \
 && chmod 0440 /etc/sudoers.d/dev

# ---- block0 region ---------------------------------------------------------
RUN mkdir -p /var/decomk /var/log/decomk \
 && git clone https://github.com/ciwg/workspace-config /var/decomk/conf \
 && decomk run

RUN chown -R dev:dev /var/decomk /var/log/decomk
