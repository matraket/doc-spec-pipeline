---
name: doc-spec-generator
description: >
  Generación y actualización de las referencias fragmentadas del skill
  doc-spec-manager a partir de la documentación fuente en spec/.
  Usar cuando: (1) se hayan modificado documentos en spec/,
  (2) se necesite regenerar las referencias del doc-spec-manager,
  (3) se quiera validar la integridad de las referencias actuales.
---

# Doc Spec Generator

Transforma los documentos de `spec/` en fragmentos atómicos optimizados para
agentes en `doc-spec-manager/references/`. Toda la generación es determinista
(scripts Python, sin LLM): la misma entrada siempre produce la misma salida.

## Cuándo Usar

- Después de modificar cualquier archivo en `spec/`.
- Si se sospecha que `references/` está desactualizado o corrupto.
- Para validar la integridad de las referencias actuales.

## Workflow: Regeneración Completa

Ejecutar el script orquestador:

```bash
python3 .agents/skills/doc-spec-generator/scripts/generate_all.py
```

Este script ejecuta los siguientes pasos automáticamente:

1. **Verificar spec/**: comprueba que los 8 archivos fuente existen.
2. **Limpiar references/**: elimina fragmentos y head files anteriores.
3. **Extraer fragmentos**: ejecuta los 8 scripts `extract_*.py`.
4. **Generar head files**: ejecuta los 8 scripts `gen_head_*.py`.
5. **Validar integridad**: ejecuta `validate_references.py`.

Salida esperada: `627 archivos` (619 fragmentos + 8 head files), ~6 segundos.

## Workflow: Solo Validación

Para verificar la integridad sin regenerar:

```bash
python3 .agents/skills/doc-spec-generator/scripts/validate_references.py
```

Comprueba: head files presentes, fragmentos completos, nombres lowercase,
contenido no vacío, sin huérfanos.

## Mapeo Fuente → Fragmentos

| Fuente en spec/ | Script | Output en references/ |
|---|---|---|
| `003_requisitos-funcionales.md` | `extract_rf.py` + `gen_head_rf.py` | `rf/` + `head-requisitos-funcionales.md` |
| `004_rnf-base.md` | `extract_rnf.py` + `gen_head_rnf.py` | `rnf/` + `head-requisitos-no-funcionales.md` |
| `005_modelo-dominio.md` | `extract_bc.py` + `gen_head_bc.py` | `bc/` + `head-modelo-dominio.md` |
| `006_adrs.md` | `extract_adr.py` + `gen_head_adr.py` | `adr/` + `head-adrs.md` |
| `007_stack.md` | `extract_stack.py` + `gen_head_stack.py` | `stack/` + `head-stack.md` |
| `008_rnf-tecnicos.md` | `extract_rnft.py` + `gen_head_rnft.py` | `rnft/` + `head-requisitos-no-funcionales-tech.md` |
| `009_user-stories.md` | `extract_us.py` + `gen_head_us.py` | `us/` + `head-user-stories.md` |
| `010_casos-uso.md` | `extract_uc.py` + `gen_head_uc.py` | `uc/` + `head-use-cases.md` |

## Scripts

Todos los scripts están en `scripts/` y comparten `lib_parser.py`:

| Script | Función |
|---|---|
| `lib_parser.py` | Librería compartida: parsing de metadatos, split por heading, code fence handling |
| `extract_*.py` | Fragmentan un documento spec/ en archivos individuales |
| `gen_head_*.py` | Generan los archivos head-*.md (metadatos + índice + resúmenes) |
| `generate_all.py` | Orquestador: limpia + genera + valida |
| `validate_references.py` | Validación de integridad post-generación |
| `test_lib_parser.py` | 108 tests para lib_parser.py |

### Ejecutar tests

```bash
python3 .agents/skills/doc-spec-generator/scripts/test_lib_parser.py
```

## Añadir un Nuevo Documento spec/

Si se añade un nuevo archivo a `spec/` (ej. `011_nuevo.md`):

1. Crear `extract_nuevo.py` siguiendo el patrón de los existentes.
2. Crear `gen_head_nuevo.py` siguiendo el patrón de los existentes.
3. Registrar ambos en `generate_all.py` (imports + listas de ejecución).
4. Añadir la validación en `validate_references.py`.
5. Actualizar el SKILL.md de `doc-spec-manager` con el nuevo tipo.
6. Ejecutar `generate_all.py` y verificar que pasa la validación.

## Principios de Diseño

- **Determinismo**: sin LLM, sin timestamps, sin estado. Idempotente.
- **spec/ es la fuente de verdad**: nunca editar references/ directamente.
- **Code fence awareness**: el parser ignora `#` dentro de bloques ```.
- **Lowercase kebab-case**: todos los nombres de archivo generados.
