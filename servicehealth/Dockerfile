# Copyright (C) 2026 Sam Dornan
# This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

# ==========================================
# STAGE 1: TESTING
# ==========================================
# We use Debian 13 here to maintain ecosystem consistency with the Distroless target
FROM debian:13@sha256:9c6e381421d0f48d1b8133e573f33c185ee16cbf505eda9c4a8a4791b25ea8a6 AS tester

RUN apt-get update && \
    apt-get install -y bats && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY monitor.sh .
COPY monitor.bats .
RUN chmod +x monitor.sh

# Execute the test suite
RUN bats monitor.bats

# ==========================================
# STAGE 2: PRODUCTION
# ==========================================
# We strictly pin to the mathematically verified Minimal base built in the pi-prod-base repo.
# Hardware binaries (vcgencmd, curl) and SSL certs are pre-harvested in this layer.
FROM ghcr.io/s-dornan/pi-prod-base@sha256:c7ff4f767b32baa1a6a6d89a5027ac9d9e821b25d0ea794e0fba2f8b5dde26a2

# Add the BusyBox tools (like sleep, grep, awk) for the monitor script
ENV PATH="/usr/bin:/bin:/busybox"

# Copy the AGPL-licensed script into the container and make it executable
COPY --chmod=755 monitor.sh /bin/monitor.sh

# We use ENTRYPOINT to lock the container into running this specific script via the Distroless shell.
# There is no default CMD daemon; when the script stops, the container dies.
ENTRYPOINT ["/busybox/sh", "/bin/monitor.sh"]
