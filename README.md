# URL Shortener

![Tests](https://img.shields.io/badge/tests-32%20passing-brightgreen)
![Coverage](https://img.shields.io/badge/coverage-98.27%25-brightgreen)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-blue)
![CI](https://github.com/adcsm2/url-shortener/actions/workflows/ci.yml/badge.svg)

A URL shortener API (like bit.ly) built with Express, TypeScript, and PostgreSQL.

Shorten URLs, redirect via short codes, and track click statistics — all through a RESTful API with rate limiting and full test coverage.

## Setup

```bash
npm install
docker-compose up -d          # Start PostgreSQL
cp .env.example .env
npm run migration:run          # Run database migrations
```

## Usage

### Shorten a URL
```bash
curl -X POST http://localhost:3000/api/shorten \
  -H "Content-Type: application/json" \
  -d '{"url": "https://www.google.com"}'
```

### Redirect via short code
```bash
curl -I http://localhost:3000/abc1234
# HTTP 302 → Location: https://www.google.com
```

### Get URL statistics
```bash
curl http://localhost:3000/api/stats/abc1234
```

### Health check
```bash
curl http://localhost:3000/health
```

## Development

```bash
npm run dev            # Express server with hot reload
npm run build          # Compile TypeScript to dist/
npm start              # Run compiled output
npm test               # Run tests
npm run test:watch     # Run tests in watch mode
npm run test:coverage  # Run tests with coverage report
npm run lint           # Run ESLint
```

## Tech Stack

- **Node.js + TypeScript** (strict mode)
- **Express** — HTTP server
- **PostgreSQL + TypeORM** — Database and ORM
- **nanoid** — Short code generation
- **express-rate-limit** — Two-layer rate limiting
- **Jest + Supertest** — Testing
- **GitHub Actions** — CI pipeline
- **Docker Compose** — PostgreSQL container

## Architecture

```
Express HTTP → Routes → Services → TypeORM Entities → PostgreSQL
```

- `src/index.ts` — Express setup, middleware, route mounting
- `src/routes/` — HTTP layer (parse request, call service, return response)
- `src/services/` — Business logic, database queries via TypeORM
- `src/entities/` — TypeORM entity definitions
- `src/middleware/` — URL validation, rate limiting
- `src/utils/` — ID generation with nanoid

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/shorten` | Shorten a URL |
| `GET` | `/:shortCode` | Redirect to original URL (302) |
| `GET` | `/api/stats/:shortCode` | Get URL statistics |
| `GET` | `/health` | Health check |

## What I Learned

This project was built to demonstrate key technical competencies:

- **RESTful API Design**: Resource-based routing, proper HTTP status codes (201, 302, 404, 429), JSON responses
- **Database Design**: PostgreSQL with TypeORM, unique indexes, atomic operations (`clicks = clicks + 1`), auto-generated migrations
- **Rate Limiting**: Two-layer strategy (global + per-route) calibrated by operation cost, with standard `RateLimit-*` headers
- **ID Generation**: Cryptographically secure short codes with nanoid, custom alphabet excluding ambiguous characters, collision retry logic
- **Comprehensive Testing**: 32 tests with 98.27% coverage, service mocking with Jest, HTTP testing with Supertest
- **CI/CD**: GitHub Actions pipeline running lint, build, and tests on every PR
- **TypeScript Strict Mode**: No implicit `any`, explicit return types, decorator metadata for TypeORM

## Architecture Decisions

Key technical decisions are documented in `docs/dev/`:

- **[ID Generation (nanoid)](docs/dev/architecture/001-id-generation.md)** — Why nanoid over hashing or auto-increment
- **[Database Choice (PostgreSQL)](docs/dev/architecture/002-database-choice.md)** — Why PostgreSQL over MongoDB or Redis
- **[Rate Limiting (Two Layers)](docs/dev/architecture/003-rate-limiting.md)** — Why global + per-route rate limiting
- **[TypeORM Strict Mode](docs/dev/learnings/001-typeorm-strict-mode.md)** — Handling strict TypeScript with TypeORM entities
- **[Docker on Windows](docs/dev/learnings/002-docker-windows-setup.md)** — WSL2 and Linux containers setup
