## 5. Infraestructura

### 5.1 Contenedores: Docker

**Seleccionado:** Docker + Docker Compose

**Justificación:**

- Entorno reproducible para desarrollo
- Mismo artefacto para dev/staging/prod
- Facilita onboarding de contributors
- Compatible con cualquier cloud provider

**docker-compose.yml (desarrollo):**

```yaml
services:
  api:
    build: ./api
    ports:
      - '3000:3000'
    environment:
      - DATABASE_MAIN_URL=postgresql://...
    depends_on:
      - postgres
      - minio
      - redis

  web:
    build: ./web
    ports:
      - '5173:5173'

  postgres:
    image: postgres:18-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=dev_password

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    ports:
      - '9000:9000'
      - '9001:9001'
    volumes:
      - minio_data:/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
  minio_data:
  redis_data: {}
```

### 5.2 Object Storage: MinIO / S3

**Decisión heredada de ADR-011**

| Entorno    | Solución               | Justificación                  |
| ---------- | ---------------------- | ------------------------------ |
| Desarrollo | MinIO                  | S3-compatible, local, gratuito |
| Producción | AWS S3 / Cloudflare R2 | Escalable, económico           |

**SDK:** `@aws-sdk/client-s3` (compatible con ambos)

### 5.3 Hosting Producción (Recomendado)

| Componente     | Servicio Recomendado      | Alternativa |
| -------------- | ------------------------- | ----------- |
| Backend        | Railway / Render          | Fly.io      |
| Frontend       | Vercel / Cloudflare Pages | Netlify     |
| PostgreSQL     | Railway / Supabase        | Neon        |
| Object Storage | Cloudflare R2             | AWS S3      |

**Justificación:**

- Servicios con tier gratuito o muy económico
- Despliegue automático desde GitHub
- Escalado automático si necesario
- SSL incluido
