> **Categoría:** 2. Seguridad

### 2.1 RNFT-001: Autenticación con JWT y Passport

**RNF Base:** RNF-001 (Autenticación de Usuarios)

**Implementación NestJS:**

```typescript
// auth.module.ts
@Module({
  imports: [
    JwtModule.registerAsync({
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: {
          expiresIn: '15m', // Access token corto
          algorithm: 'HS256',
        },
      }),
    }),
    PassportModule.register({ defaultStrategy: 'jwt' }),
  ],
})
export class AuthModule {}
```

**Configuración de tokens:**

| Token         | Duración   | Almacenamiento    | Renovación             |
| ------------- | ---------- | ----------------- | ---------------------- |
| Access Token  | 15 minutos | Memory (frontend) | Automática con refresh |
| Refresh Token | 7 días     | HttpOnly Cookie   | En login               |

**Excepción — Recordar sesión (Remember Me):**

Cuando el usuario activa la opción "Recordar sesión" en el login (UC-068 FA-3), el Refresh Token se genera con expiración de **30 días** en lugar de los 7 días por defecto. La cookie httpOnly mantiene el mismo mecanismo; solo varía el valor de `maxAge`. El Access Token no cambia (sigue siendo 15 minutos).

| Escenario               | Duración Refresh Token | Referencia |
| ----------------------- | ---------------------- | ---------- |
| Login estándar          | 7 días                 | RNFT-001   |
| Login con "Recordar Me" | 30 días                | UC-068 FA-3 |

**Política de contraseñas (class-validator):**

```typescript
// password.dto.ts
@Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/, {
  message: 'Password must have 8+ chars, uppercase, lowercase, number'
})
password: string;
```

**Bloqueo de cuenta:**

```typescript
// Configuración
MAX_FAILED_ATTEMPTS = 5;
LOCKOUT_DURATION_MINUTES = 15;
```

**Verificación:**

- Test E2E: Login con credenciales válidas/inválidas
- Test: Bloqueo tras 5 intentos fallidos
- Auditoría: Revisar tokens generados tienen claims correctos
