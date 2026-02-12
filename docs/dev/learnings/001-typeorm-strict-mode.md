# Learning 001: TypeORM Entities with TypeScript Strict Mode

## Problem

When generating the migration with TypeORM, TypeScript threw `TS2564` errors on all entity properties:

```
Property 'id' has no initializer and is not definitely assigned in the constructor.
```

## Cause

With `strict: true` in tsconfig.json, TypeScript enables `strictPropertyInitialization`, which requires all class properties to be initialized in the constructor or at declaration.

TypeORM initializes properties via decorators and reflection metadata, not in the constructor, so TypeScript cannot verify it statically.

## Solution

Use the `!` operator (definite assignment assertion) on each entity property:

```typescript
@PrimaryGeneratedColumn()
id!: number;  // <- the ! tells TS "trust me, this will be initialized"

@Column({ unique: true, length: 10 })
shortCode!: string;
```

## Discarded Alternative

Disabling `strictPropertyInitialization` in tsconfig.json. We didn't do this because we want to keep full strict mode for the rest of the code.

## Reference

- [TypeORM FAQ - Strict mode](https://typeorm.io/faq#how-to-use-typeorm-with-ts-strict-mode)
