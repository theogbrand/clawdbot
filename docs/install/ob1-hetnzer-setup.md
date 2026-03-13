---
summary: "Custom scratch setup flow for running OpenClaw on a Hetzner-style VPS with Docker and an `oc` shell helper"
title: "OB1 Hetnzer Setup"
---

# OB1 Hetnzer Setup

This is a scratch setup flow for running OpenClaw in Docker on a Hetzner-style
VPS, then managing it from the host with an `oc` shell helper.

## Before the first startup

Before the initial `docker compose build`, check whether the VPS has swap. On
smaller hosts, the Docker image build can get OOM-killed without it.

If swap is missing, follow [Swap File to Prevent OOM During Docker Builds](/swap-file-to-prevent-oom)
first, then come back here.

## Build and launch

Run from the repo root:

```bash
docker compose build
docker compose up -d openclaw-gateway
```

## Add a host shortcut

Add this once so `oc ...` runs the OpenClaw CLI inside the running Docker
container:

```bash
cat >> ~/.bashrc <<'EOF'
oc() {
  (
    cd /path/to/openclaw || exit 1
    docker compose exec openclaw-gateway openclaw "$@"
  )
}
EOF

source ~/.bashrc
```

Replace `/path/to/openclaw` with your repo path on the VPS.

## Verify the gateway

```bash
docker compose logs -f openclaw-gateway
```

You want to see:

```text
[gateway] listening on ws://0.0.0.0:18789
```

## Open the Control UI from your laptop

Create an SSH tunnel from your laptop to the VPS:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Then open:

```text
http://127.0.0.1:18789/
```

Paste your gateway token.

## Pair the browser device

This requires manually running 2 commands to pair:

```bash
oc devices list
oc devices approve <requestId>
```

The underlying OpenClaw commands are:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

## Common host-side commands

```bash
oc --help
oc devices list
oc channels status
oc dashboard --no-open
```
