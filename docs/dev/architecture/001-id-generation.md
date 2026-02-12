# ADR 001: ID Generation Algorithm

## Status

Accepted

## Context

We need to generate short, unique, URL-safe IDs for shortened URLs. The ID appears directly in the URL (`http://localhost:3000/abc1234`), so it must be:
- Short (7 characters)
- Unique (no collisions)
- URL-safe (no characters requiring encoding)
- Readable (no ambiguous characters)

## Options Considered

### 1. nanoid with custom alphabet (CHOSEN)
- Cryptographically secure random generation
- Custom alphabet excluding ambiguous characters (0, O, I, l)
- 7 chars with 57-character alphabet = 57^7 ~ 1.95 billion combinations
- Very low collision probability

### 2. Truncated hash (MD5/SHA)
- Hash the full URL and take the first 7 characters
- Deterministic: same URL → same hash
- Problem: high collision probability when truncating (birthday paradox)
- Loses the advantage of hashing since collision checks are still needed

### 3. Auto-increment + Base62
- Auto-incrementing numeric ID converted to base62
- Sequential IDs → predictable (all URLs can be enumerated)
- Requires additional base conversion logic
- Security problem: reveals how many URLs exist in the system

## Decision

**nanoid v3** with a custom 57-character alphabet and length 7.

```typescript
const alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";
const nanoid = customAlphabet(alphabet, 7);
```

## Collision Handling

Although the probability is extremely low (~1 in 1.95 billion), we implement a retry loop with a maximum of 5 attempts:

```typescript
do {
  shortCode = generateShortCode();
  const collision = await repository.findOne({ where: { shortCode } });
  if (!collision) break;
  attempts++;
} while (attempts < MAX_ATTEMPTS);
```

## Trade-offs

| Aspect | nanoid | Truncated hash | Auto-increment |
|--------|--------|----------------|----------------|
| Security | High (random) | Medium | Low (predictable) |
| Uniqueness | Very high | Medium (collisions) | Guaranteed |
| Performance | Fast | Medium (hash + truncate) | Fast |
| Complexity | Low | Medium | Medium |

## Note on nanoid v3 vs v5

We use nanoid v3 because it's compatible with CommonJS (`require()`). nanoid v5+ is ESM-only, which causes issues with TypeORM and ts-node in CommonJS mode.
