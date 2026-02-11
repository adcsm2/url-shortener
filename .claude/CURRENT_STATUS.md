# URL Shortener - Estado Actual

**Última actualización**: 2025-02-11  
**Proyecto**: URL Shortener API  
**Objetivo**: Aprender Express, PostgreSQL, generación de IDs únicos

## ✅ Completado

Nada aún - proyecto por empezar.

## 🚧 En progreso

Ninguno.

## 📋 Fase 1: Setup inicial (SIGUIENTE)

### Tareas:
1. Crear estructura de carpetas
2. Inicializar npm y git
3. Instalar dependencias
4. Configurar TypeScript (strict mode)
5. Configurar Jest + Supertest
6. Configurar ESLint
7. Crear docker-compose.yml para PostgreSQL
8. Crear .env.example
9. Configurar TypeORM
10. Crear .gitignore
11. Crear README.md

### Archivos a crear:
- `package.json`
- `tsconfig.json`
- `jest.config.js`
- `eslint.config.js`
- `docker-compose.yml`
- `.env.example`
- `src/data-source.ts` (TypeORM config)
- `.gitignore`
- `README.md`

## 📋 Fase 2: Entidad y migración

### Tareas:
1. Crear entity `Url`
2. Generar migración inicial
3. Configurar scripts para migrations
4. Probar conexión a DB

### Archivos a crear:
- `src/entities/Url.ts`
- `src/migrations/XXXXX-CreateUrlTable.ts` (auto-generado)

## 📋 Fase 3: Endpoint POST /api/shorten

### Tareas:
1. Crear `src/utils/idGenerator.ts`
2. Crear `src/services/urlService.ts`
3. Crear `src/routes/shorten.ts`
4. Crear middleware de validación
5. Configurar Express en `src/index.ts`
6. Crear tests unitarios para idGenerator
7. Crear tests unitarios para urlService
8. Crear tests de integración para POST /api/shorten

### Archivos a crear:
- `src/utils/idGenerator.ts`
- `src/services/urlService.ts`
- `src/routes/shorten.ts`
- `src/middleware/validateUrl.ts`
- `src/index.ts`
- `tests/utils/idGenerator.test.ts`
- `tests/services/urlService.test.ts`
- `tests/routes/shorten.test.ts`

## 📋 Fase 4: Endpoint GET /:shortCode (redirect)

### Tareas:
1. Crear `src/routes/redirect.ts`
2. Implementar lógica de redirect
3. Incrementar contador de clicks
4. Crear tests de integración para GET /:shortCode

### Archivos a crear:
- `src/routes/redirect.ts`
- `tests/routes/redirect.test.ts`

## 📋 Fase 5: Endpoint GET /api/stats/:shortCode

### Tareas:
1. Crear `src/routes/stats.ts`
2. Implementar lógica de stats
3. Crear tests de integración para GET /api/stats

### Archivos a crear:
- `src/routes/stats.ts`
- `tests/routes/stats.test.ts`

## 📋 Fase 6: Documentación y polish

### Tareas:
1. Documentar decisión 001: ID generation (nanoid)
2. Documentar decisión 002: PostgreSQL choice
3. Documentar learnings sobre TypeORM
4. Actualizar README con badges y ejemplos
5. Añadir rate limiting (express-rate-limit)
6. Verificar cobertura >80%

### Archivos a crear:
- `docs/dev/architecture/001-id-generation.md`
- `docs/dev/architecture/002-database-choice.md`
- `docs/dev/learnings/001-typeorm-migrations.md`
- `src/middleware/rateLimiter.ts`

## 🎯 Comandos esperados (end-to-end)
````bash
# Levantar DB
docker-compose up -d

# Correr migrations
npm run migration:run

# Levantar servidor
npm run dev

# Acortar URL
curl -X POST http://localhost:3000/api/shorten \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/very/long/url"}'

# Respuesta:
# {
#   "shortUrl": "http://localhost:3000/abc123",
#   "shortCode": "abc123",
#   "originalUrl": "https://example.com/very/long/url"
# }

# Probar redirect
curl -I http://localhost:3000/abc123

# Respuesta:
# HTTP/1.1 302 Found
# Location: https://example.com/very/long/url

# Ver stats
curl http://localhost:3000/api/stats/abc123

# Respuesta:
# {
#   "shortCode": "abc123",
#   "originalUrl": "https://example.com/very/long/url",
#   "clicks": 1,
#   "createdAt": "2025-02-11T10:00:00Z"
# }
````

## 📊 Métricas objetivo

- **Tests**: 25-30 tests pasando
- **Cobertura**: >80%
- **Endpoints**: 3 funcionando (shorten, redirect, stats)
- **Documentación**: 2-3 decisiones arquitectónicas

## 🔧 Stack confirmado

- Node.js v20 + TypeScript strict
- Express 4.x
- PostgreSQL 15 (via Docker)
- TypeORM 0.3.x
- Jest + Supertest
- nanoid para IDs
````

---

## Prompts por Fase

### **Prompt Fase 1: Setup inicial**
````
Vamos a crear un URL Shortener (acortador de URLs como bit.ly) para aprender sobre Express, PostgreSQL, y generación de IDs únicos.

Lee los archivos .claude/context.md y .claude/CURRENT_STATUS.md para contexto completo.

FASE 1: Setup inicial del proyecto

Ejecuta estos pasos en orden:

1. Crear estructura de carpetas:
````bash
mkdir -p src/{entities,routes,services,utils,middleware,migrations}
mkdir -p tests/{routes,services,utils}
mkdir -p docs/dev/{architecture,learnings}
mkdir .claude scripts
````

2. Inicializar proyecto:
````bash
npm init -y
git init
````

3. Instalar dependencias de producción:
````bash
npm install express typeorm reflect-metadata pg dotenv nanoid@3 validator express-rate-limit
npm install @types/express @types/node @types/validator
````

4. Instalar dependencias de desarrollo:
````bash
npm install -D typescript tsx ts-node-dev jest @types/jest ts-jest supertest @types/supertest eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
````

5. Crear tsconfig.json:
````json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
````

6. Crear jest.config.js:
````js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/tests'],
  testMatch: ['**/*.test.ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/migrations/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
````

7. Crear package.json scripts:
````json
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "typeorm": "typeorm-ts-node-commonjs",
    "migration:generate": "npm run typeorm migration:generate",
    "migration:run": "npm run typeorm migration:run",
    "migration:revert": "npm run typeorm migration:revert"
  }
}
````

8. Crear docker-compose.yml:
````yaml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    container_name: url-shortener-db
    environment:
      POSTGRES_USER: urlshortener
      POSTGRES_PASSWORD: password
      POSTGRES_DB: urlshortener
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
````

9. Crear .env.example:
````
# Server
PORT=3000
NODE_ENV=development

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=urlshortener
DB_PASSWORD=password
DB_DATABASE=urlshortener

# Application
BASE_URL=http://localhost:3000
SHORT_CODE_LENGTH=7
````

10. Crear .gitignore:
````
node_modules/
dist/
.env
coverage/
*.log
.DS_Store
data/
````

11. Crear README.md básico:
````markdown
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

## Usage
```bash
npm run dev
```

## Tech Stack

- Node.js + TypeScript
- Express
- PostgreSQL + TypeORM
- Jest + Supertest
````

12. Configurar TypeORM data source en src/data-source.ts:
````typescript
// Generated by Claude Code - 2025-02-11
import "reflect-metadata";
import { DataSource } from "typeorm";
import * as dotenv from "dotenv";

dotenv.config();

export const AppDataSource = new DataSource({
  type: "postgres",
  host: process.env.DB_HOST || "localhost",
  port: parseInt(process.env.DB_PORT || "5432"),
  username: process.env.DB_USERNAME || "urlshortener",
  password: process.env.DB_PASSWORD || "password",
  database: process.env.DB_DATABASE || "urlshortener",
  synchronize: false,
  logging: process.env.NODE_ENV === "development",
  entities: ["src/entities/**/*.ts"],
  migrations: ["src/migrations/**/*.ts"],
  subscribers: []
});
````

13. Configurar ESLint (eslint.config.js con Flat Config):
- No `any` permitido
- Return types obligatorios

Al terminar, ejecuta:
````bash
git add .
git commit -m "chore: initial project setup with TypeScript, Express, PostgreSQL

- Configured TypeScript strict mode
- Setup Jest with Supertest for API testing
- Added Docker Compose for PostgreSQL
- Configured TypeORM with migrations support
- Added ESLint with strict rules"
````

Marca el código generado con "// Generated by Claude Code"