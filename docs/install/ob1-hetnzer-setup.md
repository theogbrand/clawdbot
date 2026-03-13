---
summary: "Custom scratch setup flow for running OpenClaw on a Hetzner-style VPS with Docker and an `oc` shell helper"
title: "OB1 Hetnzer Setup"
---

# OB1 Hetnzer Setup

This is a scratch setup flow for running OpenClaw in Docker on a Hetzner-style
VPS, then managing it from the host with an `oc` shell helper.

This flow assumes you access the Control UI through an SSH tunnel, so keep the
gateway bind on loopback. If you later switch to a non-loopback bind, you must
also configure `gateway.controlUi.allowedOrigins`.

## Before the first startup

Before the initial `docker compose build`, check whether the VPS has swap. On
smaller hosts, the Docker image build can get OOM-killed without it.

If swap is missing, follow [Swap File to Prevent OOM During Docker Builds](/swap-file-to-prevent-oom)
first, then come back here.

## Build and launch

Before launch, make sure `.env` contains:

```bash
OPENCLAW_GATEWAY_BIND=loopback
```

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
    cd /root/openclaw || exit 1
    docker compose exec openclaw-gateway openclaw "$@"
  )
}
EOF

source ~/.bashrc
```

If your clone lives somewhere else, change `/root/openclaw` to match.

## Verify the gateway

```bash
docker compose logs -f openclaw-gateway
```

You want to see:

```text
[gateway] listening on ws://127.0.0.1:18789
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
oc setup # adds config files like openclaw.json
oc configure
oc devices list
oc channels status
oc dashboard --no-open
```

docker compose build --no-cache
docker compose up -d --force-recreate

Sample Baseline Config File:
{
  "meta": {
    "lastTouchedVersion": "2026.3.13",
    "lastTouchedAt": "2026-03-13T11:51:16.117Z"
  },
  "wizard": {
    "lastRunAt": "2026-03-13T11:51:16.109Z",
    "lastRunVersion": "2026.3.13",
    "lastRunCommand": "configure",
    "lastRunMode": "local"
  },
  "auth": {
    "profiles": {
      "openai-codex:default": {
        "provider": "openai-codex",
        "mode": "oauth"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "openai-codex/gpt-5.4"
      },
      "models": {
        "openai-codex/gpt-5.4": {}
      },
      "workspace": "/home/node/.openclaw/workspace",
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  },
  "messages": {
    "ackReactionScope": "group-mentions"
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw"
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": ""
    },
    "controlUi": {
      "allowedOrigins": [
        "http://127.0.0.1:18789",
        "http://localhost:18789"
      ]
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  }
}
