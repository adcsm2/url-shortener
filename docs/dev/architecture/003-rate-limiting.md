# ADR 003: Two-Layer Rate Limiting

## Status

Accepted

## Context

Without rate limiting, any client can make unlimited requests to the API. This exposes the service to:

- **Denial of service (DoS) attacks**: overwhelming the server with thousands of requests
- **Resource abuse**: excessive queries to PostgreSQL, CPU/memory consumption
- **Mass scraping**: automated data extraction
- **Unfairness**: a single user hogging all server resources

## Options Considered

### 1. Global rate limit only
- A single limit applied equally to all routes (e.g., 200 req/15min)
- Simple to implement
- Problem: doesn't distinguish between cheap (GET) and expensive (POST) operations. A user could make 200 database writes in 15 minutes

### 2. Per-route rate limits only
- Specific limits per operation based on computational cost
- Granular and efficient
- Problem: an attacker can distribute requests across multiple endpoints to bypass individual limits

### 3. Two layers: global + per-route (CHOSEN)
- Global layer as a safety net against distributed abuse
- Per-route layer to protect expensive operations
- Minimal added complexity: just one extra `rateLimit()` instance

## Decision

**Two layers of rate limiting** using `express-rate-limit` v8:

### Layer 1: Global limit

| Parameter | Value |
|-----------|-------|
| Window | 15 minutes |
| Maximum | 200 requests |

### Layer 2: Per-route limits

| Route | Maximum | Rationale |
|-------|---------|-----------|
| `POST /api/shorten` | 10 req/15min | Database write + ID generation. Most expensive operation. |
| `GET /:shortCode` | 100 req/15min | Database read + redirect. Most frequent operation, needs generous limit. |
| `GET /api/stats/:shortCode` | 50 req/15min | Database read. Intermediate cost. |

### Execution order

```
Request → express.json() → globalLimiter → router → routeLimiter → handler
```

## Implementation

- **Library**: `express-rate-limit` v8
- **Storage**: in-memory (MemoryStore, default)
- **Client identification**: by IP (library default)
- **Response when exceeded**: HTTP 429 with JSON `{ error: "Too many requests..." }`
- **Headers**: `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` (RFC 6585 standard)

## Trade-offs

| Aspect | Global only | Per-route only | Two layers |
|--------|-------------|----------------|------------|
| DoS protection | High | Medium | High |
| Granularity | Low | High | High |
| Bypass via distribution | Protected | Vulnerable | Protected |
| Complexity | Low | Medium | Medium |

## Scalability

The current MemoryStore works for a single instance. For multi-instance production deployments, replace with a centralized store like Redis:

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
