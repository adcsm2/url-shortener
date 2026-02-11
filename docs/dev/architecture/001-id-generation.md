# ADR 001: Algoritmo de Generación de IDs

## Estado

Aceptado

## Contexto

Necesitamos generar IDs cortos, únicos y URL-safe para las URLs acortadas. El ID aparece directamente en la URL (`http://localhost:3000/abc1234`), por lo que debe ser:
- Corto (7 caracteres)
- Único (sin colisiones)
- URL-safe (sin caracteres que requieran encoding)
- Legible (sin caracteres ambiguos)

## Opciones evaluadas

### 1. nanoid con custom alphabet (ELEGIDA)
- Generación aleatoria criptográficamente segura
- Alphabet personalizado sin caracteres ambiguos (0, O, I, l)
- 7 chars con 57 caracteres de alphabet = 57^7 ≈ 1.95 mil millones de combinaciones
- Probabilidad de colisión muy baja

### 2. Hash (MD5/SHA) truncado
- Hashear la URL completa y tomar los primeros 7 caracteres
- Determinístico: misma URL → mismo hash
- Problema: alta probabilidad de colisión al truncar (birthday paradox)
- Se pierde la ventaja del hashing al necesitar verificar colisiones igualmente

### 3. Auto-increment + Base62
- ID numérico autoincremental convertido a base62
- IDs secuenciales → predecible (se pueden enumerar todas las URLs)
- Requiere lógica adicional de conversión de base
- Problema de seguridad: revela cuántas URLs hay en el sistema

## Decisión

**nanoid v3** con custom alphabet de 57 caracteres y longitud 7.

```typescript
const alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";
const nanoid = customAlphabet(alphabet, 7);
```

## Manejo de colisiones

Aunque la probabilidad es extremadamente baja (~1 en 1.95 mil millones), implementamos un retry loop con máximo 5 intentos:

```typescript
do {
  shortCode = generateShortCode();
  const collision = await repository.findOne({ where: { shortCode } });
  if (!collision) break;
  attempts++;
} while (attempts < MAX_ATTEMPTS);
```

## Trade-offs

| Aspecto | nanoid | Hash truncado | Auto-increment |
|---------|--------|---------------|----------------|
| Seguridad | Alta (aleatorio) | Media | Baja (predecible) |
| Unicidad | Muy alta | Media (colisiones) | Garantizada |
| Performance | Rápido | Medio (hash + truncate) | Rápido |
| Complejidad | Baja | Media | Media |

## Nota sobre nanoid v3 vs v5

Usamos nanoid v3 porque es compatible con CommonJS (`require()`). nanoid v5+ es ESM-only, lo que causa problemas con TypeORM y ts-node en modo CommonJS.
