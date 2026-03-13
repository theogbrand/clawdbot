# Swap File to Prevent OOM During Docker Builds

The Docker image build (`pnpm build:docker`) runs `tsdown` which is memory-hungry. On low-RAM VPS hosts (8GB or less) with no swap, the Linux OOM killer will SIGKILL the bundler process, causing the build to fail with:

```
ERR_PNPM_RECURSIVE_EXEC_FIRST_FAIL  Command was killed with SIGKILL (Forced termination): tsdown ...
```

## Symptoms

- Docker build fails at the `pnpm build:docker` step after ~150s
- Exit code 137 or `SIGKILL` in the error message
- `dmesg | grep -i oom` shows the kernel killed a node process
- `free -h` shows 0B swap

## Fix: Add Swap

Create a 4GB swap file (adjust size for your host — 4GB is fine for 8GB RAM):

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Persist across reboots:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Verify:

```bash
free -h
# Should show Swap: 4.0Gi
```

## Dockerfile: Node Heap Cap

The `Dockerfile` also caps Node heap for the build step to prevent runaway memory usage:

```dockerfile
RUN NODE_OPTIONS=--max-old-space-size=3072 pnpm build:docker
```

The `pnpm install` step already has a similar cap (`--max-old-space-size=2048`). Both are needed — install and build are separate OOM risk points.

## Quick Checklist for New VPS Setup

1. `free -h` — check if swap exists
2. If swap is 0B, create the swap file (commands above)
3. Confirm the `Dockerfile` has `NODE_OPTIONS` on the build step
4. `docker compose up -d openclaw-gateway` — build should complete without OOM
