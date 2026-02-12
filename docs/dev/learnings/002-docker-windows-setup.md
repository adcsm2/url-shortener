# Learning 002: Docker Desktop on Windows - Initial Setup

## Problem

When running `docker-compose up -d`, we encountered several consecutive errors:

1. `docker daemon is not running` — Docker Desktop was not open
2. `open \\.\pipe\docker_engine: The system cannot find the file specified` — WSL2 kernel missing
3. `open \\.\pipe\docker_engine_windows` — Docker was in Windows containers mode

## Solutions Applied

### 1. WSL2 kernel not installed

```bash
wsl --update
```

Docker Desktop on Windows requires WSL2 as its backend. If the kernel is not installed, Docker cannot start.

### 2. Windows containers vs Linux containers

Docker Desktop can run either Windows or Linux containers, but not both simultaneously. PostgreSQL (`postgres:15-alpine`) is a Linux image.

**Fix**: Right-click Docker Desktop (tray icon) → "Switch to Linux containers..."

## Checklist for Windows Setup

1. Install Docker Desktop
2. Run `wsl --update` as administrator
3. Verify Docker uses Linux containers (not Windows)
4. Open Docker Desktop and wait until it says "Engine running"
5. Run `docker ps` to verify connection
