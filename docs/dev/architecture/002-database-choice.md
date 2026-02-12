# ADR 002: PostgreSQL vs Other Databases

## Status

Accepted

## Context

The URL shortener needs a database to store shortened URLs, their original URLs, and click statistics. The choice of database affects data integrity, scalability, and development complexity.

## Options Considered

### 1. PostgreSQL (CHOSEN)
- Relational database with full ACID guarantees
- Unique indexes on `shortCode` column prevent duplicates at the database level
- Rich query capabilities for future analytics (clicks over time, top URLs, etc.)
- Mature ecosystem with TypeORM support
- Free tier available on most cloud providers

### 2. MongoDB
- Document-oriented NoSQL database
- Flexible schema, no migrations needed
- Problem: no native ACID transactions across documents (before v4.0)
- Unique indexes exist but relational queries (joins) are limited
- Overkill flexibility for a well-defined schema like ours

### 3. Redis
- In-memory key-value store, extremely fast reads/writes
- Problem: data is volatile by default (lost on restart unless persistence is configured)
- No complex queries — only key-based lookups
- No relational capabilities for future features (user accounts, URL groups, etc.)
- Better suited as a cache layer, not primary storage

## Decision

**PostgreSQL 15** running in Docker, accessed via TypeORM.

Key reasons:
- **ACID guarantees**: every URL creation is atomic — the short code is either fully saved or not at all
- **Unique index on `shortCode`**: the database enforces uniqueness even under concurrent writes, preventing duplicate codes
- **Relational model**: supports future expansion (user accounts, URL groups, tags) with proper foreign keys
- **`increment()` operation**: atomic `UPDATE SET clicks = clicks + 1` prevents race conditions when counting clicks

## Trade-offs

| Aspect | PostgreSQL | MongoDB | Redis |
|--------|-----------|---------|-------|
| Data integrity | ACID, strong | Eventual (pre-v4) | Volatile |
| Schema flexibility | Rigid (migrations) | Flexible | Key-value only |
| Query power | Full SQL | Document queries | Key lookup only |
| Setup complexity | Higher (Docker) | Medium | Low |
| Future scalability | Relational growth | Document growth | Cache layer |
| Read performance | Fast (with indexes) | Fast | Fastest |

## When to Consider Redis

Redis would complement PostgreSQL as a **cache layer** for high-traffic scenarios:
- Cache frequently accessed short codes to avoid database lookups on every redirect
- Store rate limiting counters (replacing the current in-memory store) for multi-instance deployments
- This is an optimization, not a replacement — PostgreSQL remains the source of truth
