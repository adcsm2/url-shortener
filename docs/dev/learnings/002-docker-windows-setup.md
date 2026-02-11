# Learning 002: Docker Desktop en Windows - Setup Inicial

## Problema

Al intentar `docker-compose up -d`, obtuvimos varios errores consecutivos:

1. `docker daemon is not running` — Docker Desktop no estaba abierto
2. `open \\.\pipe\docker_engine: El sistema no puede encontrar el archivo especificado` — WSL2 kernel faltante
3. `open \\.\pipe\docker_engine_windows` — Docker estaba en modo Windows containers

## Soluciones aplicadas

### 1. WSL2 kernel no instalado

```bash
wsl --update
```

Docker Desktop en Windows requiere WSL2 como backend. Si el kernel no está instalado, Docker no puede arrancar.

### 2. Windows containers vs Linux containers

Docker Desktop puede ejecutar contenedores Windows o Linux, pero no ambos simultáneamente. PostgreSQL (imagen `postgres:15-alpine`) es una imagen Linux.

**Fix**: Clic derecho en Docker Desktop (tray icon) → "Switch to Linux containers..."

## Checklist para setup en Windows

1. Instalar Docker Desktop
2. Ejecutar `wsl --update` como administrador
3. Verificar que Docker usa Linux containers (no Windows)
4. Abrir Docker Desktop y esperar a que diga "Engine running"
5. Ejecutar `docker ps` para verificar conexión
