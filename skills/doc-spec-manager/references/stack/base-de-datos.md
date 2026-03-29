## 4. Base de Datos

### 4.1 RDBMS: PostgreSQL 18.x

**Decisión heredada de ADR-005**

**Configuración multi-tenant (ADR-002):**

```
PostgreSQL Instance
├── associated_main          (BC-Identity: usuarios, tenants)
├── associated_tenant_001    (datos tenant 1)
├── associated_tenant_002    (datos tenant 2)
└── associated_tenant_XXX    (datos tenant N)
```

**Extensiones requeridas:**

- `uuid-ossp`: Generación de UUIDs
- `pg_trgm`: Búsqueda fuzzy (RNF-019)
- `pgcrypto`: Funciones criptográficas

### 4.2 ORM: Prisma 7.x

**Seleccionado:** Prisma

| Criterio      | Prisma         | TypeORM     | Drizzle      | MikroORM    |
| ------------- | -------------- | ----------- | ------------ | ----------- |
| Type safety   | ✅ Excelente   | ⚠️ Parcial  | ✅ Excelente | ✅ Bueno    |
| Migraciones   | ✅ Automáticas | ⚠️ Manuales | ⚠️ Manuales  | ✅ Buenas   |
| Multi-DB      | ✅ Soporte     | ✅ Soporte  | ✅ Soporte   | ✅ Soporte  |
| Query builder | ✅ Intuitivo   | ⚠️ Verbose  | ✅ SQL-like  | ⚠️ Complejo |
| Performance   | ✅ Bueno       | ⚠️ Variable | ✅ Excelente | ✅ Bueno    |
| DX            | ✅ Excelente   | ⚠️ Media    | ✅ Buena     | ⚠️ Media    |

**Justificación:**

- Generación automática de tipos TypeScript desde schema
- Migraciones versionadas y reproducibles (RNF-066)
- Prisma Studio para debugging
- Soporte nativo para múltiples datasources (multi-tenant)
- Excelente documentación

**Configuración multi-tenant:**

```typescript
// prisma/schema.prisma (main)
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_MAIN_URL")
}

// Tenant datasource dinámico en runtime
const tenantPrisma = new PrismaClient({
  datasources: {
    db: { url: getTenantConnectionUrl(tenantId) }
  }
});
```

### 4.3 Caché: Redis 7.x

**Seleccionado:** Redis 7.x (latest stable)

**Justificación:**

- Caching de sesiones server-side (RNFT-002)
- Bull Queue para procesamiento asíncrono y operaciones masivas (RNFT-018)
- BC-Communication: emails, notificaciones
- Locks distribuidos para operaciones críticas (remesas SEPA, generación de cargos masivos)

**Complemento en cliente:**

- React Query cache en frontend (stale-while-revalidate)
- HTTP caching headers para recursos estáticos
