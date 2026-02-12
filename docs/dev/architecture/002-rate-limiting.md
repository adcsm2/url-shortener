# ADR 002: Rate Limiting de dos capas

## Estado

Aceptado

## Contexto

Sin rate limiting, cualquier cliente puede hacer peticiones ilimitadas a la API. Esto expone el servicio a:

- **Ataques de denegación de servicio (DoS)**: saturar el servidor con miles de requests
- **Abuso de recursos**: queries excesivas a PostgreSQL, consumo de CPU/memoria
- **Scraping masivo**: extracción automatizada de datos
- **Inequidad**: un solo usuario acaparando todos los recursos

## Opciones evaluadas

### 1. Rate limit global únicamente
- Un solo límite igual para todas las rutas (ej: 200 req/15min)
- Simple de implementar
- Problema: no distingue entre operaciones baratas (GET) y costosas (POST). Un usuario podría hacer 200 escrituras en BD en 15 minutos

### 2. Rate limits por ruta únicamente
- Límites específicos por operación según su coste
- Granular y eficiente
- Problema: un atacante puede distribuir requests entre múltiples endpoints para evadir los límites individuales

### 3. Dos capas: global + por ruta (ELEGIDA)
- Capa global como red de seguridad contra abuso distribuido
- Capa por ruta para proteger operaciones costosas
- Complejidad mínima: solo una instancia extra de `rateLimit()`

## Decisión

**Dos capas de rate limiting** usando `express-rate-limit` v8:

### Capa 1: Límite global

| Parámetro | Valor |
|-----------|-------|
| Ventana   | 15 minutos |
| Máximo    | 200 requests |

### Capa 2: Límites por ruta

| Ruta | Máximo | Justificación |
|------|--------|---------------|
| `POST /api/shorten` | 10 req/15min | Escritura en BD + generación de ID. Operación más costosa. |
| `GET /:shortCode` | 100 req/15min | Lectura de BD + redirect. Operación más frecuente, debe ser generosa. |
| `GET /api/stats/:shortCode` | 50 req/15min | Lectura de BD. Coste intermedio. |

### Orden de ejecución

```
Request → express.json() → globalLimiter → router → routeLimiter → handler
```

## Implementación

- **Librería**: `express-rate-limit` v8
- **Almacenamiento**: en memoria (MemoryStore, default)
- **Identificación de cliente**: por IP (default de la librería)
- **Respuesta al exceder**: HTTP 429 con JSON `{ error: "Too many requests..." }`
- **Headers**: `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` (estándar RFC 6585)

## Trade-offs

| Aspecto | Global solo | Por ruta solo | Dos capas |
|---------|-------------|---------------|-----------|
| Protección DoS | Alta | Media | Alta |
| Granularidad | Baja | Alta | Alta |
| Evasión por distribución | Protegido | Vulnerable | Protegido |
| Complejidad | Baja | Media | Media |

## Escalabilidad

El MemoryStore actual funciona para una sola instancia. En producción multi-instancia, reemplazar por un store centralizado como Redis:

```typescript
import RedisStore from "rate-limit-redis";
import { createClient } from "redis";

const client = createClient({ url: process.env.REDIS_URL });

const limiter = rateLimit({
  store: new RedisStore({ sendCommand: (...args) => client.sendCommand(args) }),
  windowMs: 15 * 60 * 1000,
  max: 200
});
```
