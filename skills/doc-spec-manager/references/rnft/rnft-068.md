> **Categoría:** 2. Seguridad

### 2.9 RNFT-068: Blacklist de Access Tokens en Redis

**RNF Base:** RNF-068 (Invalidación Inmediata de Access Tokens Post-Logout)

**Contexto:** Cuando un usuario cierra sesión (UC-002 FA-4), el access token sigue siendo criptográficamente válido hasta su expiración natural (15 min). La blacklist en Redis permite revocar ese token de forma inmediata, garantizando que cualquier petición posterior con ese token devuelva 401.

#### Guard de verificación (BlacklistCheck)

```typescript
// blacklist.guard.ts
@Injectable()
export class BlacklistGuard implements CanActivate {
  constructor(
    @InjectRedis() private readonly redis: Redis,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const payload: JwtPayload = request.user; // ya validado por JwtAuthGuard

    try {
      const revoked = await this.redis.get(`blacklist:${payload.jti}`);
      if (revoked !== null) {
        throw new UnauthorizedException('Token revocado');
      }
      return true;
    } catch (err) {
      if (err instanceof UnauthorizedException) throw err;
      // Redis no disponible → fail-closed
      throw new ServiceUnavailableException('Servicio de autenticación no disponible');
    }
  }
}
```

**Comportamiento ante indisponibilidad de Redis:**

| Estado Redis  | Comportamiento      | HTTP | Justificación              |
| ------------- | ------------------- | ---- | -------------------------- |
| Disponible    | Comprueba blacklist | —    | Flujo normal               |
| Key presente  | Token revocado      | 401  | Logout ejecutado           |
| Key ausente   | Token válido        | —    | Flujo normal               |
| No disponible | Fail-closed         | 503  | Seguridad > disponibilidad |

#### Servicio de logout (operación SET)

```typescript
// auth.service.ts
async revokeAccessToken(accessToken: string): Promise<void> {
  const payload = this.jwtService.decode(accessToken) as JwtPayload;
  const ttl = payload.exp - Math.floor(Date.now() / 1000); // segundos restantes

  if (ttl > 0) {
    try {
      await this.redis.set(`blacklist:${payload.jti}`, '1', 'EX', ttl);
    } catch (err) {
      // Fail-open en escritura: loguear y continuar con revocación del refresh token
      this.logger.warn(`Redis no disponible al revocar token ${payload.jti}: ${err.message}`);
    }
  }
  // Continuar siempre con revocación del refresh token en DB
}
```

**Política de fallo en escritura (logout):** best-effort. Si Redis no está disponible durante el logout, se registra warning en el log pero la operación continúa y el refresh token se revoca en DB. El access token expirará de forma natural en máximo 15 minutos.

#### Configuración

| Parámetro             | Valor             | Variable de entorno    |
| --------------------- | ----------------- | ---------------------- |
| Prefijo de clave      | `blacklist:`      | —                      |
| TTL máximo (AT)       | 900 s (15 min)    | `JWT_ACCESS_EXPIRES`   |
| Modo fallo (lectura)  | closed → 503      | — (hardcoded, ADR-014) |
| Modo fallo (escritura)| open → log + skip | —                      |

#### Estructura de clave Redis

```
blacklist:{jti}  →  "1"  (EX {ttl_segundos_restantes})
```

- La clave expira automáticamente cuando el token habría expirado de todas formas: no se acumula basura en Redis.
- El valor `"1"` es un marcador booleano; el contenido no se evalúa, solo la existencia de la clave.
- El campo `jti` (JWT ID) debe estar presente en todos los access tokens generados; es un UUID v4.

#### Verificación

- Test unitario: `BlacklistGuard` lanza 401 cuando `redis.get` devuelve `"1"`
- Test unitario: `BlacklistGuard` lanza 503 cuando Redis lanza error de conexión
- Test unitario: `revokeAccessToken` llama a `redis.set` con TTL correcto
- Test unitario: `revokeAccessToken` continúa si Redis falla (solo warning en log)
- Test E2E: POST /auth/logout → token en blacklist → GET /members → 401 (UC-002 FA-4)
- Test E2E: POST /auth/logout con Redis caído → refresh token revocado en DB → 204 (UC-002 FE-4)
- Test E2E: Redis no disponible durante request autenticado → 503 (FE-5)
- Test E2E: token expirado naturalmente es rechazado por JwtAuthGuard antes de BlacklistCheck

**Trazabilidad:** RNF-068, ADR-014, UC-002 (FA-4, FE-4, FE-5)
