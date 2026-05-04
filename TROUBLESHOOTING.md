# Docker 29.4.2 Ingress Network Fix

## Issue
Worker deployment fails with:
```
rpc error: code = InvalidArgument desc = Service cannot be explicitly attached to the ingress network "ingress"
```

## Root Cause
Docker 26+ strictly enforces that services cannot explicitly attach to the reserved `ingress` network. The code at line 703-705 was attaching to ingress unnecessarily - `PublishModeIngress` already handles routing mesh automatically.

Added in June 2023 (commit e319f0e9) likely to fix a networking issue on older Docker versions, but breaks on modern versions.

## Fix
Set environment variable: `SHUFFLE_ATTACH_INGRESS_NETWORK=false`

This disables explicit ingress attachment while keeping port publishing working via `PublishModeIngress`.

Default: `true` (for backwards compatibility with older Docker versions)

## Quick Start (On-Prem)
```bash
git pull origin main  # Get latest orborus with fixes
docker build -t ghcr.io/shuffle/shuffle-orborus:latest .
export SHUFFLE_ATTACH_INGRESS_NETWORK=false
docker restart shuffle-orborus
```

## Changes Made
- Line 703-705: Made ingress attachment conditional via env var (opt-out)
- Lines 2173-2182, 2515-2517: Fixed SensorMode field type mismatches (strings not bools)
