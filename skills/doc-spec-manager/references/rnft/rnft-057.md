> **Categoría:** 6. Mantenibilidad y Operaciones

### 6.1 RNFT-057: Documentación con OpenAPI

**RNF Base:** RNF-057 (Documentación Técnica)

**Swagger en NestJS:**

El tenant activo se deriva EXCLUSIVAMENTE del claim `tenantId` del JWT Bearer (amendment ADR-006 Abr 2026). El header `X-Tenant-Id` NO se registra como security scheme en OpenAPI: no es requerido para la autenticación ni la autorización. Documentar la API sin él evita confundir a consumidores (clientes generados, SDKs, integraciones externas).

```typescript
// main.ts
const config = new DocumentBuilder()
  .setTitle('Associated API')
  .setDescription(
    'API del ERP para colectividades. ' +
    'El tenant activo se resuelve desde el claim `tenantId` del JWT Bearer; ' +
    'no se requiere header X-Tenant-Id.',
  )
  .setVersion('1.0')
  .addBearerAuth()
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

**Decoradores en controllers:**

```typescript
@ApiTags('Members')
@ApiBearerAuth()
@Controller('members')
export class MembersController {

  @Get()
  @ApiOperation({
    summary: 'Listar members del tenant activo (derivado del JWT)',
  })
  @ApiResponse({ status: 200, type: [MemberDto] })
  @ApiQuery({ name: 'page', required: false, type: Number })
  findAll(@Query() query: ListMembersQuery) { ... }
}
```

**Nota (amendment Abr 2026):** Las versiones previas de este snippet registraban `X-Tenant-Id` como `apiKey` header en el `DocumentBuilder` y como `@ApiHeader({ name: 'X-Tenant-Id', required: true })` en los controllers. Esas declaraciones quedan DEPRECADAS por la amendment de ADR-006 Abr 2026 y deben eliminarse: el header ya no es mecanismo de resolución de tenant.

**Generación de tipos para frontend:**

```bash
# Generar tipos TypeScript desde OpenAPI
npx openapi-typescript http://localhost:3000/api-json -o ./src/api/types.ts
```

**Inventario de Endpoints:** El inventario formal de endpoints del sistema se define en `spec/013_inventario-de-endpoints.md` bajo el formato EP-NNN. Cada endpoint EP referencia el caso de uso (UC) que lo origina y las entidades (ENT) que expone. La especificación OpenAPI generada por `@nestjs/swagger` DEBE ser consistente con este inventario.
