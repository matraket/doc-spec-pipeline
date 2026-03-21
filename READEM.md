# Associated Doc Skills

Este repositorio existe para documentar y mantener dos skills complementarios orientados al trabajo con especificaciones fragmentadas.

## Objetivo del repositorio

El objetivo real de este repo no es conservar la documentación de negocio en bruto, sino proveer un flujo claro para que agentes de IA puedan trabajar con ella de forma confiable y escalable:

`spec/` → `doc-spec-generator` → `doc-spec-manager`

- `spec/` actúa como **fuente de entrada**.
- `doc-spec-generator` transforma esa fuente en artefactos atómicos y navegables.
- `doc-spec-manager` consume esos artefactos para permitir consultas, trazabilidad y alineamiento con la especificación.

## Fuera de alcance

El directorio `spec/` queda **fuera del scope principal** de este repositorio como producto final.

Su función es servir como **artefacto de trabajo** y fuente documental para la generación posterior. En otras palabras:

- `spec/` **no** es la salida consumible por agentes.
- `spec/` **no** representa el valor principal del repo.
- El valor del repo está en los **dos skills** y en el flujo que habilitan.

## Skills incluidos

### 1. `doc-spec-generator`

Skill determinista que toma la documentación fuente ubicada en `spec/` y la atomiza en documentos pequeños, consistentes y manejables para consumo automatizado.

#### Responsabilidades

- Fragmentar documentos grandes en archivos atómicos.
- Generar archivos índice (`head-*.md`) para navegación.
- Mantener consistencia estructural entre documentos fuente y artefactos generados.
- Validar integridad de referencias generadas.
- Guiar la creación o extensión de documentos en `spec/` para que sigan un formato parseable.

#### Características clave

- Implementación determinista.
- Basado en scripts Python, sin dependencia de LLM para la generación.
- Idempotente: la misma entrada produce la misma salida.
- Elimina artefactos huérfanos y reconstruye el conjunto de referencias.
- Trata `spec/` como única fuente de verdad.

#### Entradas y salidas

**Entrada**
- Documentos fuente dentro de `spec/`.

**Salida**
- Fragmentos atómicos dentro de `references/`.
- Archivos cabecera `head-*.md` para cada tipo documental.

#### Scripts principales

- `generate_all.py`: limpia, genera y valida.
- `validate_references.py`: valida integridad.
- `extract_*.py`: extraen fragmentos por tipo documental.
- `gen_head_*.py`: generan archivos índice y cabeceras.
- `lib_parser.py`: librería compartida de parsing.

#### Cuándo usarlo

Usá este skill cuando:

- cambie cualquier archivo en `spec/`;
- necesites regenerar referencias fragmentadas;
- quieras validar integridad de los artefactos;
- necesites crear o extender documentación fuente siguiendo reglas parseables.

---

### 2. `doc-spec-manager`

Skill orientado a la navegación eficiente de la especificación ya fragmentada. Permite a agentes de IA consultar, recorrer y verificar alineamiento con la documentación del proyecto sin depender de archivos monolíticos.

#### Responsabilidades

- Navegar la especificación fragmentada por tipo de entidad.
- Consultar requisitos, bounded contexts, ADRs, stack, user stories y casos de uso.
- Verificar trazabilidad entre artefactos documentales.
- Ayudar a implementar funcionalidades alineadas con la especificación.
- Servir como interfaz de consulta sobre la salida generada por `doc-spec-generator`.

#### Modos principales de uso

1. **Implementación alineada**
   - Verificar que código y diseño respetan la especificación.
2. **Extensión de especificación**
   - Crear o ampliar entidades documentales manteniendo trazabilidad.
3. **Consulta**
   - Buscar y navegar información de forma rápida y precisa.

#### Tipos de artefactos que navega

- Requisitos funcionales (`RF`)
- Requisitos no funcionales (`RNF`)
- Requisitos no funcionales técnicos (`RNFT`)
- Bounded Contexts (`BC`)
- Architectural Decision Records (`ADR`)
- Stack tecnológico
- User Stories (`US`)
- Use Cases (`UC`)

#### Cuándo usarlo

Usá este skill cuando:

- vayas a implementar una funcionalidad y necesites alinearte con la spec;
- quieras seguir la cadena de trazabilidad entre requisitos y casos de uso;
- necesites consultar rápidamente entidades documentales específicas;
- quieras validar consistencia documental antes de desarrollar.

## Flujo de trabajo recomendado

### Paso 1. Mantener la fuente

La documentación se edita en `spec/` únicamente cuando hace falta modificar la fuente.

### Paso 2. Generar artefactos navegables

Se ejecuta `doc-spec-generator` para transformar la fuente en fragmentos atómicos.

### Paso 3. Consumir la documentación fragmentada

Se usa `doc-spec-manager` para navegar, consultar y verificar la especificación generada.

## Estructura conceptual del repositorio

- `skills/doc-spec-generator/`: definición del skill y scripts de generación.
- `skills/doc-spec-manager/`: definición del skill de navegación.
- `spec/`: insumo documental de trabajo, fuera del scope principal del repo.

## Idea central

Este repositorio no busca publicar una colección de documentos estáticos, sino establecer un pipeline documental para agentes:

1. una fuente editable;
2. una transformación determinista;
3. una capa de consumo optimizada para IA.

## Resumen

Si hubiese que resumir el propósito del repo en una sola línea:

**convertir documentación fuente extensa en conocimiento navegable y operativo para agentes de inteligencia artificial.**
