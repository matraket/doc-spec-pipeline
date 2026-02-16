Quiero crear **2 skills** para gestionar esta documentación.

---

## Contexto

Dado que la documentación es **extensa y densa**, es necesario un skill que defina la capacidad de navegar por ella y optimice la forma en que el agente buscará la información cuando sea necesario.

Por otro lado, esta documentación está **viva** y, en el futuro, sufrirá modificaciones y/o ampliaciones. Aquí surge la necesidad de crear un **segundo skill**.

---

## Objetivo

Dada la complejidad de la documentación, es necesario estructurarla correctamente. Se propone una estructura similar a la siguiente:

```
doc-spec-manager/
├── SKILL.md
└── references/
     ├── head-requisitos-funcionales.md
     ├── head-requisitos-no-funcionales.md
     ├── head-modelo-dominio.md
     ├── head-adrs.md
     ├── head-stack.md
     ├── head-requisitos-no-funcionales-tech.md
     ├── head-user-stories.md
     ├── head-use-cases.md
     ├── rf/
     │    ├── N2RF01.md
     │    ├── N2RF02.md
     │    └── ...
     ├── rnf/
     │    ├── RNF-001.md
     │    ├── RNF-002.md
     │    └── ...
     ├── bc/
     │    ├── BC-Identity.md
     │    ├── BC-Membership.md
     │    └── ...
     ├── adr/
     │    ├── ADR-001.md
     │    ├── ADR-002.md
     │    └── ...
     ├── stack/
     │    └── ? Aquí tengo dudas sobre cómo organizarlo
     │           La lógica actual del documento es:
     │               backend.md
     │               frontend.md
     │               base-de-datos.md
     │               infraestructura.md
     │               testing.md
     │               devops-ci_cd.md
     │               herramientas-desarrollo.md
     │               servicios-externos.md
     │           No estoy seguro de que sea la mejor opción.
     ├── rnft/
     │    ├── RNFT-001.md
     │    ├── RNFT-002.md
     │    └── ...
     ├── us/
     │    ├── US-001.md
     │    ├── US-002.md
     │    └── ...
     └── uc/
          ├── UC-001.md
          ├── UC-002.md
          └── ...
```

La idea es que la carpeta `spec/` sea la documentación destinada a humanos y, a partir de ella, disponer de:

---

## 1️⃣ Primer skill: `doc-spec-generator`

Su objetivo es, partiendo del estado actual de la documentación, generar y actualizar todos los archivos de la carpeta `references/` del segundo skill (`doc-spec-manager`).

### Para ello será necesario:

### a) Scripts de fragmentación

Crear scripts que dividan la documentación de forma que cada documento genere su estructura correspondiente. Por ejemplo:

```
003_requisitos-funcionales.md --> extract-rf.sh
                                   └──> doc-spec-manager/references/
                                          └── rf/
                                               ├── N2RF01.md
                                               ├── N2RF02.md
                                               └── ...
```

> **Nota:** Esto deberá aplicarse a todos los documentos correspondientes.

---

### b) Prompts y templates para generar archivos `head-*.md`

Ejemplo:

```
003_requisitos-funcionales.md
prompt_requisitos-funcionales.md
template_requisitos-funcionales.md
        └──> LLM ──> doc-spec-manager/references/
                        └── head-requisitos-funcionales.md
```

---

### c) Prompt + template para generar `SKILL.md`

Ejemplo:

```
doc-spec-manager/references/*.md
prompt_requisitos-funcionales.md
template_requisitos-funcionales.md
        └──> LLM ──> SKILL.md
```

---

## Flujo esperado

Los humanos mantendrán la documentación en `spec/` y, cuando lo estimen oportuno, invocarán el skill `doc-spec-generator`, que actualizará `doc-spec-manager`.

Los **prompts + templates** deben diseñarse para que los outputs del LLM que generen los archivos `head-*.md` y `SKILL.md` sean lo más deterministas posible.

Es decir, si se invoca el skill sin haber modificado la documentación, el resultado debe ser —si no idéntico— al menos **muy similar**.

---

## Recursos disponibles

* Un skill (`skill-creator`) que proporciona capacidades para la creación de skills.
* Un análisis de la documentación (`spec/analisis-documentacion.md`).
* Un mapa de la documentación (`spec/mapa-documentacion.md`).

Estos recursos permiten comprender:

* Qué representa cada archivo.
* Cómo se relacionan entre sí.
* Cómo se ha codificado.
* La trazabilidad entre las diferentes entidades documentales.

---

## Solicitud

Analiza minuciosamente el objetivo planteado y genera un reporte fundamentado en markdown (`analisis-skills-doc.md`) que contenga:

* Todo lo que está bien en el planteamiento y por qué.
* Todo lo que está mal y cómo corregirlo.
* Planificación detallada de cómo implementarlo.
* Cómo ampliar y/o modificar las instrucciones del archivo `CLAUDE.md` para este cometido.
