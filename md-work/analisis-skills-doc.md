# Analisis de Skills para Gestion de Documentacion

**Proyecto:** Associated - ERP Ligero para Colectividades Espanolas
**Fecha:** Febrero 2026
**Contexto:** Evaluacion del planteamiento para crear 2 skills (`doc-spec-generator` y `doc-spec-manager`)

---

## 1. Lo que esta bien y por que

### 1.1 Separacion en dos skills (generador vs gestor)

**Valoracion: Excelente.**

La decision de separar la responsabilidad en dos skills es acertada porque:

- **Principio de responsabilidad unica**: El generador transforma; el gestor navega. Mezclarlos crearia un skill demasiado grande y con dos modos de operacion muy diferentes.
- **Alineamiento con skill-creator**: El skill-creator recomienda que los skills sean "modulares y autocontenidos". Cada skill tiene un trigger claro y diferenciado.
- **Frecuencia de uso distinta**: El generador se invoca puntualmente (tras cambios en `spec/`); el gestor se consulta continuamente durante el desarrollo.

### 1.2 Flujo spec/ --> generador --> gestor

**Valoracion: Excelente.**

El flujo unidireccional `spec/` (fuente humana) --> `doc-spec-generator` (transformacion) --> `doc-spec-manager` (consumo agente) es correcto porque:

- **Single source of truth**: La documentacion en `spec/` es la unica fuente. Las references son derivadas.
- **Separacion de audiencias**: `spec/` para humanos (documentos densos, narrativos); `references/` para agentes (fragmentos atomicos, navegables).
- **Regenerabilidad**: Si se pierde `references/`, se puede reconstruir desde `spec/`.

### 1.3 Scripts de fragmentacion determinista

**Valoracion: Excelente.**

Usar scripts (bash/python) para fragmentar documentos es la decision correcta porque:

- **Determinismo total**: La misma entrada siempre produce la misma salida.
- **Alineamiento con skill-creator**: Los scripts van en `scripts/` y se ejecutan sin necesidad de cargarlos en el contexto.
- **Los documentos son altamente estructurados**: Todos siguen patrones regulares de markdown con separadores `---`, headers `###` y codigos identificables (`N2RF01`, `RNF-001`, `ADR-001`, `US-001`, `UC-001`, `BC-*`). Esto los hace ideales para parsing con regex.

### 1.4 Estructura de references/ por tipo de entidad

**Valoracion: Bien planteada.**

Organizar por tipo (`rf/`, `rnf/`, `bc/`, `adr/`, `us/`, `uc/`, `rnft/`) refleja la codificacion real de la documentacion:

| Carpeta | Fuente | Patron de codigo | Cantidad |
|---------|--------|-------------------|----------|
| `rf/` | 003 | NxRFyy | 221 |
| `rnf/` | 004 | RNF-xxx | 66 |
| `bc/` | 005 | BC-Name | 6 |
| `adr/` | 006 | ADR-xxx | 12 |
| `stack/` | 007 | (por seccion) | ~8 secciones |
| `rnft/` | 008 | RNFT-xxx | ~61 |
| `us/` | 009 | US-xxx | 202 |
| `uc/` | 010 | UC-xxx | 76 |

Esto permite a Claude cargar exactamente lo que necesita: si trabaja en BC-Treasury, carga `bc/bc-treasury.md` y las US/UC relacionadas, sin cargar los 32.000+ lineas completas.

---

## 2. Lo que esta mal y como corregirlo

### 2.1 CRITICO: Nombres de archivo con mayusculas

**Problema:** El planteamiento propone archivos como `N2RF01.md`, `BC-Identity.md`, `ADR-001.md`, `US-001.md`, `UC-001.md`.

**Esto viola directamente CLAUDE.md:**

> **CRITICAL**: Never use uppercase letters, spaces, or special characters.

**Correccion:** Todos los nombres deben ser lowercase kebab-case:

| Propuesto | Corregido |
|-----------|-----------|
| `N2RF01.md` | `n2rf01.md` |
| `BC-Identity.md` | `bc-identity.md` |
| `ADR-001.md` | `adr-001.md` |
| `RNF-001.md` | `rnf-001.md` |
| `RNFT-001.md` | `rnft-001.md` |
| `US-001.md` | `us-001.md` |
| `UC-001.md` | `uc-001.md` |

**Nota importante:** Dentro del contenido de los archivos, los codigos si mantienen su formato original (`N2RF01`, `BC-Identity`, etc.) porque son codigos de negocio, no nombres de archivo. La restriccion es solo para el sistema de archivos.

### 2.2 CRITICO: Los `head-*.md` NO deben generarse con LLM

**Problema:** El planteamiento propone usar prompts + templates + LLM para generar los archivos `head-*.md`. Esto introduce **no-determinismo** inevitable: dos invocaciones del LLM con la misma entrada produciran resultados con variaciones de redaccion, orden de frases, nivel de detalle, etc.

**Por que es un problema real:**

- Si el generador se invoca sin cambios en `spec/`, el `git diff` mostrara cambios espurios.
- No se puede verificar si un cambio en `head-*.md` refleja un cambio real en la documentacion o simplemente una variacion del LLM.
- Rompe el objetivo declarado: "si se invoca sin modificar la documentacion, el resultado debe ser muy similar".

**Correccion:** Los `head-*.md` deben generarse **100% con scripts deterministas** que extraigan:

1. **Metadatos del header**: Titulo, version, fecha, estado, inputs, totales.
2. **Indice de secciones**: Copiado textual del indice del documento.
3. **Tabla de resumen**: Codigos, cantidades, categorias.
4. **Mapa de trazabilidad**: Referencias cruzadas extraidas del propio documento.
5. **Lista de archivos fragmentados**: Links a los archivos individuales en su subcarpeta.

Esto es perfectamente viable porque **todos los documentos tienen estructura regular**: header con metadatos, indice, secciones con patron predecible, y seccion de trazabilidad al final.

**Ejemplo de `head-requisitos-funcionales.md` generado deterministicamente:**

```markdown
# Requisitos Funcionales (RF)

## Metadatos
- **Fuente:** spec/003_requisitos-funcionales.md
- **Version:** 1.0
- **Estado:** Validado
- **Total RFs:** 221

## Indice por seccion
| Seccion | Ambito | RFs | Rango |
|---------|--------|-----|-------|
| N2 | Arquitectura y Acceso | 8 | n2rf01 - n2rf08 |
| N3 | Gestion de Socios | 34 | n3rf01 - n3rf34 |
| ... | ... | ... | ... |

## Trazabilidad
- **Downstream:** RNF (004), BC (005), US (009)
- **Upstream:** KB-002

## Archivos individuales
Cada RF se encuentra en `rf/n{x}rf{yy}.md`.
Para consultar un RF especifico: `references/rf/n2rf01.md`
Para buscar por seccion: `grep -l "Seccion: N4" references/rf/`
```

### 2.3 CRITICO: El `SKILL.md` de doc-spec-manager NO debe generarse con LLM

**Problema:** El planteamiento propone generar `SKILL.md` con LLM a partir de los `references/`. Esto tiene los mismos problemas de no-determinismo, pero ademas:

- `SKILL.md` contiene **instrucciones procedimentales** (como navegar, cuando cargar que archivo, patrones de busqueda). Esto no cambia cuando cambia el contenido de la documentacion.
- Lo que cambia son los `references/`, no las instrucciones de navegacion.

**Correccion:** `SKILL.md` debe ser un archivo **redactado manualmente una unica vez** y actualizado solo cuando:

- Se anade un nuevo tipo de documento (ej: un nuevo `spec/011_*.md`).
- Cambia la estructura de `references/`.

El `SKILL.md` debe ser **estable** y solo referenciar los `head-*.md` como punto de entrada a la documentacion. Los `head-*.md` si se regeneran con cada invocacion del generador, pero son deterministas (ver 2.2).

### 2.4 IMPORTANTE: Organizacion de `stack/`

**Problema declarado por el usuario:** Duda sobre como organizar la carpeta `stack/`.

**Analisis:** El documento `007_stack.md` tiene 815 lineas y esta organizado en secciones tematicas:

```
1. Resumen Ejecutivo (tabla resumen + principios)
2. Backend (TypeScript, NestJS)
3. Frontend (React, Mantine, Vite)
4. Base de Datos (PostgreSQL, Prisma)
5. Infraestructura (Docker, MinIO/S3)
6. Testing (Vitest, Playwright)
7. DevOps y CI/CD (GitHub Actions)
8. Herramientas de Desarrollo
9. Servicios Externos
10. Matriz de Decisiones (tabla trazabilidad)
```

**Recomendacion:** Fragmentar por seccion tematica **es correcto**, pero:

- El Resumen Ejecutivo (seccion 1) y la Matriz de Decisiones (seccion 10) son **transversales** y deben ir en `head-stack.md`, no en archivos individuales.
- Las secciones 2-9 son los fragmentos individuales.

**Estructura propuesta:**

```
stack/
  backend.md          (seccion 2: TypeScript + NestJS)
  frontend.md         (seccion 3: React + Mantine + Vite)
  base-de-datos.md    (seccion 4: PostgreSQL + Prisma)
  infraestructura.md  (seccion 5: Docker + MinIO/S3)
  testing.md          (seccion 6: Vitest + Playwright)
  devops-ci-cd.md     (seccion 7: GitHub Actions)
  herramientas.md     (seccion 8: ESLint, Prettier, etc.)
  servicios.md        (seccion 9: Sentry, SMTP, etc.)
```

**Razonamiento:** A diferencia de otros documentos con codificacion individual (RF-xxx, RNF-xxx), el stack no tiene codigos individuales. La organizacion tematica es la unica logica posible y coincide con la estructura del documento fuente.

### 2.5 IMPORTANTE: El doc-spec-generator deberia ser un workflow, no un skill clasico

**Problema:** El planteamiento trata `doc-spec-generator` como un skill al mismo nivel que `doc-spec-manager`. Sin embargo, sus naturalezas son fundamentalmente diferentes:

| Aspecto | doc-spec-generator | doc-spec-manager |
|---------|-------------------|------------------|
| Frecuencia | Esporadica | Continua |
| Interaccion | Ejecutar y esperar | Consulta interactiva |
| Complejidad interna | Alta (scripts, validacion) | Media (navegacion) |
| Output | Archivos en disco | Informacion en contexto |

**Recomendacion:** `doc-spec-generator` **si debe ser un skill**, pero su SKILL.md debe disearse como un **workflow secuencial** (ver `references/workflows.md` del skill-creator):

```
Generacion/actualizacion de references:
1. Validar que spec/ existe y contiene documentos esperados
2. Ejecutar scripts de fragmentacion (extract-*.sh/py)
3. Ejecutar scripts de generacion de head-*.md
4. Validar integridad (conteos, links rotos)
5. Reportar cambios al usuario
```

### 2.6 MENOR: Falta manejar secciones transversales de RF

**Problema:** Los RF de las secciones N8-N13 son transversales (no pertenecen a un unico BC). El planteamiento no distingue esto.

**Recomendacion:** No es necesario cambiar la estructura de `rf/` (todos los RF van ahi independientemente de si son transversales), pero los `head-*.md` y el SKILL.md deben documentar que:

- N2-N7: Mapeados 1:1 a BCs (Identity, Membership, Treasury, Events, Communication, Documents)
- N8-N13: Transversales (Import/Export, Reporting, Portal Socio, Cumplimiento, Especificas, Aragonesas)

---

## 3. Planificacion detallada de implementacion

### Fase 0: Preparacion (prerequisito)

**Objetivo:** Establecer la infraestructura base.

| Paso | Accion | Detalle |
|------|--------|---------|
| 0.1 | Crear estructura de carpetas | `.agents/skills/doc-spec-generator/` y `.agents/skills/doc-spec-manager/` |
| 0.2 | Inicializar con `init_skill.py` | Usar el script del skill-creator para ambos skills |
| 0.3 | Actualizar CLAUDE.md | Anadir reglas especificas para la gestion documental |
| 0.4 | Actualizar .gitignore | Los skills generados SI deben estar en el repositorio (son artefactos del proyecto) |

### Fase 1: Scripts de fragmentacion (doc-spec-generator)

**Objetivo:** Crear los scripts que dividen cada documento de `spec/` en archivos individuales.

#### 1.1 Script base: `lib-parser.sh` o `lib_parser.py`

Libreria compartida con funciones comunes:

- Extraer metadatos del header (titulo, version, fecha, estado, totales)
- Parsear indice de secciones
- Dividir por patron de header markdown (`### CODIGO:` o `### CODIGO -`)
- Generar frontmatter YAML para cada fragmento
- Sanitizar nombres de archivo (lowercase, kebab-case)

**Recomendacion: Python sobre Bash.** Los documentos tienen codificacion UTF-8 con caracteres especiales (acentos, enes). Python maneja esto de forma nativa y robusta; bash + sed/awk es fragil con multibyte.

#### 1.2 Scripts de extraccion por documento

| Script | Fuente | Output | Patron de corte |
|--------|--------|--------|-----------------|
| `extract_rf.py` | 003 | `rf/n{x}rf{yy}.md` | `### N{x}RF{yy}:` |
| `extract_rnf.py` | 004 | `rnf/rnf-{xxx}.md` | `### RNF-{xxx}:` |
| `extract_bc.py` | 005 | `bc/bc-{name}.md` | `## {N}. BC-{Name}:` (secciones H2 principales) |
| `extract_adr.py` | 006 | `adr/adr-{xxx}.md` | `## ADR-{xxx}:` |
| `extract_stack.py` | 007 | `stack/{seccion}.md` | `## {N}. {Titulo}` (secciones H2, excluyendo 1 y 10) |
| `extract_rnft.py` | 008 | `rnft/rnft-{xxx}.md` | `### {N}.{M} RNFT-{xxx}:` |
| `extract_us.py` | 009 | `us/us-{xxx}.md` | `#### US-{xxx}:` |
| `extract_uc.py` | 010 | `uc/uc-{xxx}.md` | `### UC-{xxx}:` |

#### 1.3 Scripts de generacion de `head-*.md`

| Script | Fuente | Output | Contenido |
|--------|--------|--------|-----------|
| `gen_head_rf.py` | 003 | `head-requisitos-funcionales.md` | Metadatos + indice secciones + tabla RF por seccion + trazabilidad |
| `gen_head_rnf.py` | 004 | `head-requisitos-no-funcionales.md` | Metadatos + categorias + tabla RNF por categoria |
| `gen_head_bc.py` | 005 | `head-modelo-dominio.md` | Metadatos + tabla BCs + subdominios + context map simplificado |
| `gen_head_adr.py` | 006 | `head-adrs.md` | Metadatos + tabla ADR + estado de cada uno |
| `gen_head_stack.py` | 007 | `head-stack.md` | Resumen ejecutivo + tabla seleccion + principios |
| `gen_head_rnft.py` | 008 | `head-requisitos-no-funcionales-tech.md` | Metadatos + tabla RNFT por categoria + stack referencia |
| `gen_head_us.py` | 009 | `head-user-stories.md` | Metadatos + distribucion MoSCoW + tabla US por BC |
| `gen_head_uc.py` | 010 | `head-use-cases.md` | Metadatos + distribucion por BC + tabla UC resumen |

**Nota:** Los scripts `gen_head_*.py` pueden reutilizar `lib_parser.py` y su logica es puramente extractiva: copian textualmente secciones especificas del documento fuente (metadatos, indice, tablas resumen, trazabilidad).

#### 1.4 Script orquestador: `generate_all.py`

Script principal que:

1. Verifica que `spec/` contiene los archivos esperados.
2. Limpia el directorio `references/` de destino (para evitar archivos huerfanos).
3. Ejecuta cada script de extraccion.
4. Ejecuta cada script de generacion de head.
5. Ejecuta validacion de integridad:
   - Conteo de archivos generados vs esperados.
   - Verificacion de que todos los codigos estan cubiertos.
   - Links internos validos.
6. Genera reporte de ejecucion.

#### 1.5 Script de validacion: `validate_references.py`

Validaciones post-generacion:

- Todos los codigos de `spec/` estan representados en `references/`.
- Cada `head-*.md` referencia correctamente sus archivos.
- No hay archivos huerfanos (generados anteriormente pero sin correspondencia actual).
- Los conteos en `head-*.md` coinciden con los archivos reales.

### Fase 2: SKILL.md de doc-spec-manager (manual)

**Objetivo:** Redactar el SKILL.md que define como navegar la documentacion fragmentada.

**Contenido propuesto para SKILL.md:**

```yaml
---
name: doc-spec-manager
description: >
  Navegacion y consulta de la especificacion del proyecto Associated.
  Usar cuando se necesite: (1) consultar requisitos funcionales (RF),
  (2) verificar requisitos no funcionales (RNF/RNFT),
  (3) consultar el modelo de dominio (BCs, Aggregates),
  (4) revisar decisiones arquitectonicas (ADR),
  (5) consultar el stack tecnologico,
  (6) buscar user stories (US) o casos de uso (UC),
  (7) verificar trazabilidad entre entidades documentales.
---
```

**Cuerpo del SKILL.md (esquema):**

1. **Estructura de la documentacion**: Breve explicacion de los 8 tipos de entidad.
2. **Como navegar**:
   - Empezar por el `head-*.md` correspondiente al tipo de entidad.
   - Usar grep para buscar codigos especificos.
   - Patrones de busqueda comunes.
3. **Cadena de trazabilidad**: RF --> RNF --> RNFT --> ADR --> BC --> US --> UC.
4. **Indice de references**: Links a todos los `head-*.md`.

**Tamano estimado:** ~150-200 lineas (dentro del limite de 500 recomendado por skill-creator).

### Fase 3: SKILL.md de doc-spec-generator (manual)

**Contenido propuesto para SKILL.md:**

```yaml
---
name: doc-spec-generator
description: >
  Generacion y actualizacion de las referencias fragmentadas del skill
  doc-spec-manager a partir de la documentacion fuente en spec/.
  Usar cuando: (1) se hayan modificado documentos en spec/,
  (2) se necesite regenerar las referencias del doc-spec-manager,
  (3) se quiera validar la integridad de las referencias actuales.
---
```

**Cuerpo: workflow secuencial** con los pasos de `generate_all.py`.

### Fase 4: Testing y validacion

| Test | Descripcion | Criterio de exito |
|------|-------------|-------------------|
| Idempotencia | Ejecutar generador 2 veces sin cambios en spec/ | `diff` vacio entre ambas ejecuciones |
| Completitud | Contar archivos generados | 221 RF + 66 RNF + 6 BC + 12 ADR + 8 stack + ~61 RNFT + 202 US + 76 UC = ~652 archivos + 8 heads |
| Integridad | Verificar links en heads | Todos los links apuntan a archivos existentes |
| Encoding | Verificar caracteres especiales | Acentos, enes, simbolos correctos en todos los archivos |
| Navegabilidad | Test manual con Claude | Claude puede encontrar informacion sobre "remesas SEPA" partiendo de doc-spec-manager |

### Fase 5: Iteracion

Segun el paso 6 del skill-creator:

1. Usar `doc-spec-manager` en tareas reales de desarrollo.
2. Identificar problemas de navegacion o informacion faltante.
3. Ajustar scripts y SKILL.md segun hallazgos.

---

## 4. Estructura final propuesta

```
.agents/skills/
  doc-spec-generator/
    SKILL.md
    scripts/
      lib_parser.py          (funciones comunes de parsing)
      extract_rf.py          (003 --> rf/)
      extract_rnf.py         (004 --> rnf/)
      extract_bc.py          (005 --> bc/)
      extract_adr.py         (006 --> adr/)
      extract_stack.py       (007 --> stack/)
      extract_rnft.py        (008 --> rnft/)
      extract_us.py          (009 --> us/)
      extract_uc.py          (010 --> uc/)
      gen_head_rf.py         (003 --> head-requisitos-funcionales.md)
      gen_head_rnf.py        (004 --> head-requisitos-no-funcionales.md)
      gen_head_bc.py         (005 --> head-modelo-dominio.md)
      gen_head_adr.py        (006 --> head-adrs.md)
      gen_head_stack.py      (007 --> head-stack.md)
      gen_head_rnft.py       (008 --> head-requisitos-no-funcionales-tech.md)
      gen_head_us.py         (009 --> head-user-stories.md)
      gen_head_uc.py         (010 --> head-use-cases.md)
      generate_all.py        (orquestador principal)
      validate_references.py (validacion post-generacion)
  doc-spec-manager/
    SKILL.md
    references/
      head-requisitos-funcionales.md
      head-requisitos-no-funcionales.md
      head-modelo-dominio.md
      head-adrs.md
      head-stack.md
      head-requisitos-no-funcionales-tech.md
      head-user-stories.md
      head-use-cases.md
      rf/
        n2rf01.md ... n13rf04.md     (221 archivos)
      rnf/
        rnf-001.md ... rnf-066.md    (66 archivos)
      bc/
        bc-identity.md
        bc-membership.md
        bc-treasury.md
        bc-events.md
        bc-communication.md
        bc-documents.md
        bc-transversal.md            (seccion 7bis: cumplimiento normativo)
      adr/
        adr-001.md ... adr-012.md    (12 archivos)
      stack/
        backend.md
        frontend.md
        base-de-datos.md
        infraestructura.md
        testing.md
        devops-ci-cd.md
        herramientas.md
        servicios.md
      rnft/
        rnft-001.md ... rnft-061.md  (~61 archivos)
      us/
        us-001.md ... us-202.md      (202 archivos)
      uc/
        uc-001.md ... uc-076.md      (76 archivos)
```

**Total estimado: ~652 archivos fragmentados + 8 heads.**

---

## 5. Modificaciones necesarias en CLAUDE.md

### 5.1 Anadir seccion de documentacion

Se propone anadir la siguiente seccion a `CLAUDE.md`:

```markdown
## Documentation Management

* Two skills manage the project specification:
  * `doc-spec-manager`: Navigates and queries the fragmented spec.
    Read its SKILL.md first for navigation instructions.
  * `doc-spec-generator`: Regenerates doc-spec-manager references
    from spec/ source files. Invoke after modifying spec/ documents.

* Documentation hierarchy:
  * `spec/` is the human-maintained source of truth.
  * `.agents/skills/doc-spec-manager/references/` contains
    agent-optimized fragments derived from spec/.
  * NEVER edit references/ directly. Always edit spec/ and regenerate.

* When implementing features:
  1. Use doc-spec-manager to find relevant RFs, USs, UCs.
  2. Follow traceability chain: RF -> RNF -> RNFT -> ADR -> BC -> US -> UC.

* File naming in references/:
  * All filenames are lowercase kebab-case (rnf-001.md, bc-identity.md).
  * Business codes inside files keep original format (RNF-001, BC-Identity).
```

### 5.2 Seccion de nombres de archivo: aclaracion

Ampliar la regla existente de caracteres permitidos:

```markdown
  * Allowed characters:
    * a-z
    * 0-9  ← (correccion: actualmente dice 0-1, deberia ser 0-9)
    * Hyphen `-`
    * Underscore `_`
    * Period `.` (only for file extensions)
```

**Nota:** El CLAUDE.md actual dice `0-1` en lugar de `0-9`. Esto es probablemente un error tipografico que deberia corregirse.

---

## 6. Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigacion |
|--------|-------------|---------|------------|
| Scripts no parsean correctamente todos los patrones | Alta | Alto | Tests unitarios por documento; validacion de conteos |
| Documentos futuros cambian su estructura | Media | Alto | lib_parser.py con configuracion flexible por documento |
| references/ se desincroniza con spec/ | Media | Medio | Validacion en CI; timestamp de generacion en heads |
| Contexto de Claude se llena con demasiados fragments | Baja | Medio | heads como punto de entrada; carga selectiva |

---

## 7. Orden de implementacion recomendado

```
Semana 1: Preparacion
  [0.1] Crear estructura de carpetas
  [0.2] Inicializar skills con init_skill.py
  [0.3] Actualizar CLAUDE.md y .gitignore

Semana 2: Scripts de fragmentacion (1/2)
  [1.1] lib_parser.py (funciones comunes)
  [1.2a] extract_rf.py + tests (003, mas simple para validar patron)
  [1.2b] extract_rnf.py + tests (004)
  [1.2c] extract_adr.py + tests (006)
  [1.2d] extract_stack.py + tests (007)

Semana 3: Scripts de fragmentacion (2/2)
  [1.2e] extract_bc.py + tests (005, mas complejo por subsecciones)
  [1.2f] extract_rnft.py + tests (008)
  [1.2g] extract_us.py + tests (009, volumen alto)
  [1.2h] extract_uc.py + tests (010, mas grande y complejo)

Semana 4: Generacion de heads + orquestacion
  [1.3] Scripts gen_head_*.py (8 scripts)
  [1.4] generate_all.py (orquestador)
  [1.5] validate_references.py

Semana 5: SKILL.md + testing
  [2.0] Redactar SKILL.md de doc-spec-manager
  [3.0] Redactar SKILL.md de doc-spec-generator
  [4.0] Tests de idempotencia, completitud, integridad
  [4.1] Test de navegabilidad con Claude

Semana 6: Iteracion
  [5.0] Uso real en tareas de desarrollo
  [5.1] Ajustes basados en hallazgos
```

---

## 8. Resumen de decisiones

| Decision | Planteamiento original | Recomendacion | Razon |
|----------|----------------------|---------------|-------|
| Dos skills | Correcto | Mantener | Responsabilidad unica |
| Scripts fragmentacion | Correcto | Mantener (Python) | Determinismo |
| head-*.md con LLM | **Incorrecto** | Scripts deterministas | No-determinismo inaceptable |
| SKILL.md con LLM | **Incorrecto** | Redaccion manual | Contenido procedimental estable |
| Nombres archivo | **Incorrecto** (mayusculas) | Lowercase kebab-case | Regla critica CLAUDE.md |
| stack/ organizacion | Duda declarada | Por seccion tematica | Sin codificacion individual |
| Lenguaje scripts | No especificado | Python | UTF-8 robusto, parsing complejo |
