# Learning 001: TypeORM Entities con TypeScript Strict Mode

## Problema

Al generar la migración con TypeORM, TypeScript lanzó errores `TS2564` en todas las propiedades de la entity:

```
Property 'id' has no initializer and is not definitely assigned in the constructor.
```

## Causa

Con `strict: true` en tsconfig.json, TypeScript activa `strictPropertyInitialization`, que exige que todas las propiedades de una clase se inicialicen en el constructor o en la declaración.

TypeORM inicializa las propiedades via decoradores y metadata de reflexión, no en el constructor, por lo que TypeScript no puede verificarlo estáticamente.

## Solución

Usar el operador `!` (definite assignment assertion) en cada propiedad de la entity:

```typescript
@PrimaryGeneratedColumn()
id!: number;  // <- el ! le dice a TS "confía, esto se inicializará"

@Column({ unique: true, length: 10 })
shortCode!: string;
```

## Alternativa descartada

Desactivar `strictPropertyInitialization` en tsconfig.json. No lo hicimos porque queremos mantener strict mode completo para el resto del código.

## Referencia

- [TypeORM FAQ - Strict mode](https://typeorm.io/faq#how-to-use-typeorm-with-ts-strict-mode)
