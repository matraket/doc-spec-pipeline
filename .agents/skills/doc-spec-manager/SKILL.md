---
name: doc-spec-manager
description: >
  Navegación y consulta de la especificación del proyecto Associated.
  Usar cuando se necesite: (1) consultar requisitos funcionales (RF),
  (2) verificar requisitos no funcionales (RNF/RNFT),
  (3) consultar el modelo de dominio (BCs, Aggregates),
  (4) revisar decisiones arquitectónicas (ADR),
  (5) consultar el stack tecnológico,
  (6) buscar user stories (US) o casos de uso (UC),
  (7) verificar trazabilidad entre entidades documentales.
---

# Doc Spec Manager

Navegación eficiente de la especificación fragmentada del proyecto Associated.
La documentación fuente (`spec/`) está fragmentada en ~627 archivos atómicos
optimizados para consulta por agentes.

## Estructura de la Documentación

### Tipos de Entidad

| Tipo | Código | Carpeta | Head file | Cantidad |
|------|--------|---------|-----------|----------|
| Requisitos Funcionales | N{x}RF{yy} | `rf/` | `head-requisitos-funcionales.md` | 221 |
| Requisitos No Funcionales | RNF-{xxx} | `rnf/` | `head-requisitos-no-funcionales.md` | 66 |
| Bounded Contexts | BC-{Name} | `bc/` | `head-modelo-dominio.md` | 7 |
| Decisiones Arquitectónicas | ADR-{xxx} | `adr/` | `head-adrs.md` | 12 |
| Stack Tecnológico | (por sección) | `stack/` | `head-stack.md` | 8 |
| RNF Técnicos | RNFT-{xxx} | `rnft/` | `head-requisitos-no-funcionales-tech.md` | 27 |
| User Stories | US-{xxx} | `us/` | `head-user-stories.md` | 202 |
| Casos de Uso | UC-{xxx} | `uc/` | `head-use-cases.md` | 76 |

### Jerarquía de Archivos

```
references/
├── head-*.md              ← Punto de entrada por tipo (metadatos + índice)
├── rf/n{x}rf{yy}.md      ← Fragmento individual de RF
├── rnf/rnf-{xxx}.md       ← Fragmento individual de RNF
├── bc/bc-{name}.md        ← Fragmento individual de BC
├── adr/adr-{xxx}.md       ← Fragmento individual de ADR
├── stack/{seccion}.md     ← Fragmento individual de Stack
├── rnft/rnft-{xxx}.md     ← Fragmento individual de RNFT
├── us/us-{xxx}.md         ← Fragmento individual de US
└── uc/uc-{xxx}.md         ← Fragmento individual de UC
```

## Cómo Navegar

### 1. Consulta por tipo de entidad

**Siempre empezar por el head file** correspondiente. Los head files contienen
metadatos, índice de secciones, tablas resumen y trazabilidad.

```
# Para entender los requisitos funcionales:
Read references/head-requisitos-funcionales.md

# Para ver el modelo de dominio completo (incluye Context Map y trazabilidad):
Read references/head-modelo-dominio.md
```

### 2. Consulta de un item específico

Si ya conoces el código, accede directamente al fragmento:

```
# RF específico
Read references/rf/n2rf01.md

# RNF específico
Read references/rnf/rnf-001.md

# BC específico
Read references/bc/bc-identity.md

# ADR específico
Read references/adr/adr-001.md

# Stack por sección
Read references/stack/backend.md

# RNFT específico
Read references/rnft/rnft-001.md

# US específica
Read references/us/us-001.md

# UC específico
Read references/uc/uc-001.md
```

### 3. Búsqueda por contenido

```
# Buscar en qué RFs se menciona "SEPA"
Grep "SEPA" references/rf/

# Buscar todas las US de prioridad Must
Grep "Must" references/us/

# Buscar qué UCs afectan a un BC
Grep "BC-Treasury" references/uc/

# Buscar RNFs de una categoría
Grep "Categoría.*Seguridad" references/rnf/
```

### 4. Secciones del Stack

Las secciones de `stack/` no tienen código numérico. Mapeo:

| Archivo | Contenido |
|---------|-----------|
| `backend.md` | TypeScript + NestJS |
| `frontend.md` | React + Mantine + Vite |
| `base-de-datos.md` | PostgreSQL + Prisma |
| `infraestructura.md` | Docker + MinIO/S3 |
| `testing.md` | Vitest + Playwright |
| `devops-ci-cd.md` | GitHub Actions |
| `herramientas.md` | ESLint, Prettier, etc. |
| `servicios.md` | Sentry, SMTP, etc. |

## Cadena de Trazabilidad

La documentación sigue una cadena de refinamiento progresivo:

```
RF (qué necesita el negocio)
 └→ RNF (restricciones agnósticas de tecnología)
     └→ RNFT (concreción técnica con el stack elegido)
         └→ ADR (decisiones arquitectónicas que justifican el stack)
 └→ BC (modelo de dominio: Aggregates, Entities, Value Objects)
     └→ US (historias de usuario agrupadas por BC)
         └→ UC (casos de uso: flujos, servicios, eventos)
```

### Ejemplos de trazabilidad

**De RF a implementación:**
1. Leer `rf/n2rf01.md` → menciona "Aislamiento Multi-Tenant"
2. Buscar RNF relacionado: `Grep "N2RF01" references/rnf/` → `rnf-004.md`
3. Buscar RNFT: `Grep "RNF-004" references/rnft/` → `rnft-004.md`
4. Ver ADR correspondiente: referenciado en RNFT → `adr/adr-002.md`

**De UC a requisitos:**
1. Leer `uc/uc-001.md` → referencia US-001
2. Leer `us/us-001.md` → referencia N2RF01
3. Leer `rf/n2rf01.md` → requisito de negocio original

## Convenciones

- **Nombres de archivo**: siempre lowercase kebab-case (`rnf-001.md`, `bc-identity.md`).
- **Códigos de negocio**: dentro del contenido mantienen formato original (`RNF-001`, `BC-Identity`).
- **Contexto en fragmentos**: los RF, RNF, RNFT, US y UC incluyen una línea de contexto
  al inicio (blockquote `>`) indicando su sección/categoría/BC padre.

## Regeneración

Si la documentación en `spec/` cambia, las references se regeneran con el skill
`doc-spec-generator`. **Nunca editar references/ directamente.**
