# URL Shortener

![Tests](https://img.shields.io/badge/tests-pending-yellow)
![Coverage](https://img.shields.io/badge/coverage-pending-yellow)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-blue)

A URL shortener API built with Express, TypeScript, and PostgreSQL.

## Setup

```bash
npm install
docker-compose up -d
cp .env.example .env
npm run migration:run
```

## Development

```bash
npm run dev
```

## Tech Stack

- Node.js + TypeScript
- Express
- PostgreSQL + TypeORM
- Jest + Supertest
- Docker Compose
