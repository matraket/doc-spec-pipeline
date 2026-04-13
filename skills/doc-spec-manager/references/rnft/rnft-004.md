> **Categoría:** 2. Seguridad

### 2.4 RNFT-004: Multi-tenant con Prisma

**RNF Base:** RNF-004 (Aislamiento Multi-Tenant por Base de Datos)

**Implementación con Prisma dinámico:**

```typescript
// prisma-tenant.service.ts
@Injectable()
export class PrismaTenantService {
  private clients = new Map<string, PrismaClient>();

  getClient(tenantId: string): PrismaClient {
    if (!this.clients.has(tenantId)) {
      const client = new PrismaClient({
        datasources: {
          db: { url: this.getTenantDatabaseUrl(tenantId) },
        },
      });
      this.clients.set(tenantId, client);
    }
    return this.clients.get(tenantId);
  }

  private getTenantDatabaseUrl(tenantId: string): string {
    // Cada tenant tiene su propia DB y usuario
    return `postgresql://tenant_${tenantId}:${password}@host:5432/associated_${tenantId}`;
  }
}
```

**Middleware de tenant:**

El `tenantId` se extrae EXCLUSIVAMENTE del claim JWT ya validado por `JwtAuthGuard` (amendment ADR-006 Abr 2026). El header `X-Tenant-Id` NO se lee para resolver el tenant activo; si está presente, puede registrarse como dato de observabilidad (detección de drift cache-cliente vs. JWT) pero NUNCA debe influir en la lógica de scope ni en la conexión Prisma per-tenant.

```typescript
// tenant.middleware.ts
@Injectable()
export class TenantMiddleware implements NestMiddleware {
  private readonly logger = new Logger(TenantMiddleware.name);

  use(req: Request, res: Response, next: NextFunction) {
    // JwtAuthGuard ya validó el token y pobló req.user con los claims
    const user = req['user'] as { userId: string; tenantId: string } | undefined;

    if (!user?.tenantId) {
      // No hay JWT válido: los endpoints públicos no deberían pasar por aquí;
      // los protegidos fallarán antes en JwtAuthGuard.
      return next();
    }

    req['tenantId'] = user.tenantId;

    // Observabilidad: si el cliente envió X-Tenant-Id y no coincide con el JWT,
    // lo logueamos pero NO influye en la decisión. Útil para detectar caches
    // client-side desactualizadas tras un switch-tenant.
    const headerTenant = req.headers['x-tenant-id'];
    if (headerTenant && headerTenant !== user.tenantId) {
      this.logger.warn(
        `X-Tenant-Id drift: header=${headerTenant} jwt=${user.tenantId} userId=${user.userId}`,
      );
    }

    next();
  }
}
```

**Nota (amendment Abr 2026):** La versión previa de este middleware leía `X-Tenant-Id` del header y lanzaba `BadRequestException` si faltaba. Esa implementación queda DEPRECADA por la amendment de ADR-006 Abr 2026: el tenant activo es exclusivamente el claim `tenantId` del JWT.

**Configuración PostgreSQL por tenant:**

```sql
-- Crear usuario específico por tenant
CREATE USER tenant_abc123 WITH PASSWORD 'secure_password';
CREATE DATABASE associated_abc123 OWNER tenant_abc123;

-- El usuario SOLO puede acceder a su DB
REVOKE ALL ON DATABASE associated_main FROM tenant_abc123;
GRANT CONNECT ON DATABASE associated_abc123 TO tenant_abc123;
```

**Métricas:**

- Conexiones por tenant monitorizadas en Sentry
- Pool máximo por tenant: 10 conexiones
