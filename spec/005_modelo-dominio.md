# Bounded Contexts y Modelo de Dominio

**Proyecto:** Associated - ERP Ligero para Colectividades Españolas  
**Versión:** 1.5  
**Fecha:** Febrero 2026  
**Inputs:** KB-001 (Propuesta TFM), KB-002 (Análisis de Necesidades), KB-003 (Requisitos Funcionales), KB-004 (RNF Base)  
**Estado:** Validado  
**Total BCs:** 6 (3 Core + 3 Supporting) + 1 Extensión Transversal

---

## Índice

1. [Visión General del Dominio](#1-visión-general-del-dominio)
2. [Identificación de Bounded Contexts](#2-identificación-de-bounded-contexts)
3. [BC-Membresia: Gestión de Socios](#3-bc-membresia-gestión-de-socios)
4. [BC-Tesoreria: Gestión Económica](#4-bc-tesoreria-gestión-económica)
5. [BC-Eventos: Actividades y Participación](#5-bc-eventos-actividades-y-participación)
6. [BC-Comunicacion: Notificaciones y Mensajería](#6-bc-comunicacion-notificaciones-y-mensajería)
7. [BC-Documentos: Gestión Documental](#7-bc-documentos-gestión-documental)
7 bis. [Extensión Transversal: Cumplimiento Normativo](#7-bis-extensión-transversal-cumplimiento-normativo)
8. [BC-Identidad: Acceso y Autorización](#8-bc-identidad-acceso-y-autorización)
9. [Context Map](#9-context-map)
10. [Consideraciones Multi-Tenant](#10-consideraciones-multi-tenant)
11. [Glosario del Dominio (Ubiquitous Language)](#11-glosario-del-dominio-ubiquitous-language)

---

## 1. Visión General del Dominio

### 1.1 Problema de Negocio

Las colectividades españolas (asociaciones culturales, cofradías, clubes deportivos, peñas festeras) gestionan su operativa con herramientas inadecuadas (Excel, papel, WhatsApp). Los tesoreros voluntarios dedican horas a tareas administrativas que deberían estar automatizadas.

### 1.2 Subdominios Identificados

| Subdominio | Tipo | Descripción | Bounded Context |
|------------|------|-------------|-----------------|
| Gestión de Membresía | **Core** | Registro, estados, tipos de socio, antigüedad | BC-Membresia |
| Gestión Económica | **Core** | Cuotas, cobros, remesas SEPA, contabilidad | BC-Tesoreria |
| Gestión de Actividades | **Core** | Eventos, inscripciones, asistencia | BC-Eventos |
| Comunicación | Supporting | Notificaciones, emails, tablón de anuncios | BC-Comunicacion |
| Gestión Documental | Supporting | Repositorio, actas, documentos oficiales | BC-Documentos |
| Identidad y Acceso | Generic | Autenticación, autorización, roles | BC-Identidad |

### 1.3 Alcance MVP

Según KB-001, el MVP se centra en **2-3 bounded contexts core**:
- ✅ **BC-Membresia** (completo)
- ✅ **BC-Tesoreria** (completo)
- ⚠️ **BC-Eventos** (simplificado: sin específicos de cofradías/clubes)
- ⚠️ **BC-Comunicacion** (mínimo: notificaciones automáticas)
- ⚠️ **BC-Documentos** (mínimo: repositorio básico)
- ✅ **BC-Identidad** (completo: requerido para seguridad)

---

## 2. Identificación de Bounded Contexts

### 2.1 Mapeo RF → Bounded Context

| Sección RF | Bounded Context | RFs Incluidos |
|------------|-----------------|---------------|
| N2: Arquitectura/Acceso | BC-Identidad | N2RF01-N2RF08 |
| N3: Socios/Miembros | BC-Membresia | N3RF01-N3RF34 |
| N4: Tesorería/Finanzas | BC-Tesoreria | N4RF01-N4RF38 |
| N5: Eventos | BC-Eventos | N5RF01-N5RF30 |
| N6: Comunicación | BC-Comunicacion | N6RF01-N6RF23 |
| N7: Gestión Documental | BC-Documentos | N7RF01-N7RF12 |
| N8: Import/Export | Transversal | N8RF01-N8RF13 |
| N9: Reporting | Transversal | N9RF01-N9RF12 |
| N10: Portal Socio | Transversal (UI) | N10RF01-N10RF15 |
| N11: Cumplimiento | Transversal | N11RF01-N11RF17 |
| N12: Específicas | Extensiones BC | N12RF01-N12RF15 |
| N13: Aragonés | Extensiones BC | N13RF01-N13RF04 |

### 2.2 Principios de Diseño

1. **Autonomía**: Cada BC puede evolucionar independientemente
2. **Cohesión**: Conceptos relacionados permanecen juntos
3. **Acoplamiento bajo**: Comunicación entre BCs vía Domain Events
4. **Lenguaje ubicuo**: Cada BC tiene su propio vocabulario preciso

---

## 3. BC-Membresia: Gestión de Socios

### 3.1 Descripción

Responsable del ciclo de vida completo del socio: desde la solicitud de alta hasta la baja, incluyendo estados, tipos, antigüedad y documentación de identidad (carnets).

### 3.2 Aggregates

#### 3.2.1 Aggregate: Socio (Aggregate Root)

```
┌─────────────────────────────────────────────────────────────┐
│ SOCIO (Aggregate Root)                                      │
├─────────────────────────────────────────────────────────────┤
│ Identity: SocioId                                           │
│                                                             │
│ Value Objects:                                              │
│   - DatosPersonales (nombre, apellidos, fechaNacimiento)    │
│   - DatosContacto (email, telefono, direccion)              │
│   - DocumentoIdentidad (tipo, numero)                       │
│   - DatosBancarios (iban) [cifrado]                         │
│   - NumeroSocio                                             │
│                                                             │
│ Entities:                                                   │
│   - HistorialEstados[] (cambios de estado con fecha/motivo) │
│   - CamposPersonalizados[] (clave-valor por tipo entidad)   │
│                                                             │
│ State:                                                      │
│   - estadoActual: EstadoSocio                               │
│   - tipoSocio: TipoSocioId                                  │
│   - fechaAlta: Date                                         │
│   - fechaBaja: Date?                                        │
│                                                             │
│ Invariants:                                                 │
│   - DNI/NIE único dentro del tenant                         │
│   - Email único dentro del tenant                           │
│   - Fecha de nacimiento <= hoy - edad mínima del tipo       │
│   - IBAN válido según algoritmo de verificación             │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `registrar(datos, tipo)` → valida, crea con estado Aspirante/Activo
- `cambiarEstado(nuevoEstado, motivo)` → valida transición, registra historial
- `cambiarTipo(nuevoTipo)` → valida elegibilidad, ajusta derechos
- `actualizarDatos(datos)` → valida, actualiza, emite evento
- `calcularAntiguedad()` → retorna años/meses considerando bajas
- `verificarDerechoVoto()` → evalúa antigüedad y estado
- `darDeBaja(tipo, motivo)` → transición a estado de baja

#### 3.2.2 Aggregate: TipoSocio

```
┌─────────────────────────────────────────────────────────────┐
│ TIPO_SOCIO (Aggregate Root)                                 │
├─────────────────────────────────────────────────────────────┤
│ Identity: TipoSocioId                                       │
│                                                             │
│ Value Objects:                                              │
│   - RangoEdad (minima, maxima)                              │
│   - ConfiguracionCuota (importe, periodicidad, formula?)    │
│                                                             │
│ Properties:                                                 │
│   - codigo: string                                          │
│   - nombre: string                                          │
│   - descripcion: string                                     │
│   - derechoVoto: boolean                                    │
│   - elegibleCargos: boolean                                 │
│   - antiguedadMinimaVoto: int (años)                        │
│   - antiguedadMinimaCargos: int (años)                      │
│   - transicionAutomatica: TipoSocioId? (al cumplir edad)    │
│   - activo: boolean                                         │
│                                                             │
│ Invariants:                                                 │
│   - Código único dentro del tenant                          │
│   - Edad mínima < edad máxima (si ambas definidas)          │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.3 Aggregate: SolicitudAlta

```
┌─────────────────────────────────────────────────────────────┐
│ SOLICITUD_ALTA (Aggregate Root)                             │
├─────────────────────────────────────────────────────────────┤
│ Identity: SolicitudId                                       │
│                                                             │
│ Value Objects:                                              │
│   - DatosSolicitante (datos personales del aspirante)       │
│                                                             │
│ Entities:                                                   │
│   - DocumentoPendiente[] (tipo, entregado, fechaLimite)     │
│   - Aval[] (avalista: SocioId, fecha) [para cofradías]      │
│                                                             │
│ State:                                                      │
│   - estado: EstadoSolicitud                                 │
│   - fechaSolicitud: Date                                    │
│   - tipoSocioSolicitado: TipoSocioId                        │
│   - prioridad: int (para lista de espera)                   │
│                                                             │
│ Invariants:                                                 │
│   - Avales deben ser socios con antigüedad mínima           │
│   - No puede haber solicitud duplicada (mismo DNI activa)   │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.4 Aggregate: Carnet

```
┌─────────────────────────────────────────────────────────────┐
│ CARNET (Aggregate Root)                                     │
├─────────────────────────────────────────────────────────────┤
│ Identity: CarnetId                                          │
│                                                             │
│ Value Objects:                                              │
│   - CodigoQR (valor único, hash del carnet)                 │
│                                                             │
│ Properties:                                                 │
│   - socioId: SocioId                                        │
│   - ejercicioId: EjercicioId                                │
│   - fechaEmision: Date                                      │
│   - fechaValidez: Date                                      │
│   - estado: EstadoCarnet (activo, anulado)                  │
│                                                             │
│ Invariants:                                                 │
│   - Un socio solo puede tener un carnet activo por ejercicio│
│   - QR único en todo el sistema                             │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.5 Aggregate: Ejercicio

```
┌─────────────────────────────────────────────────────────────┐
│ EJERCICIO (Aggregate Root)                                  │
├─────────────────────────────────────────────────────────────┤
│ Identity: EjercicioId                                       │
│                                                             │
│ Value Objects:                                              │
│   - PeriodoEjercicio (fechaInicio, fechaFin)                │
│                                                             │
│ Properties:                                                 │
│   - nombre: string (ej: "2026", "Temporada 2025-26")        │
│   - estado: EstadoEjercicio (preparacion, activo, cerrado)  │
│   - ejercicioAnterior: EjercicioId?                         │
│                                                             │
│ Invariants:                                                 │
│   - Solo un ejercicio activo a la vez                       │
│   - Fechas no pueden solaparse con otros ejercicios         │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.6 Aggregate: ListaEspera

```
┌─────────────────────────────────────────────────────────────┐
│ LISTA_ESPERA (Aggregate Root)                               │
├─────────────────────────────────────────────────────────────┤
│ Identity: ListaEsperaId                                     │
│                                                             │
│ Value Objects:                                              │
│   - PosicionEspera (numero de orden cronológico)            │
│   - FechaInscripcion (fecha y hora de entrada en lista)     │
│                                                             │
│ Properties:                                                 │
│   - solicitudId: SolicitudId                                │
│   - tipoSocioId: TipoSocioId                                │
│   - posicion: int (calculado, orden cronológico)            │
│   - fechaEntrada: DateTime                                  │
│   - fechaSalida: DateTime? (al procesar)                    │
│   - motivo: MotivoSalidaLista? (aprobado, rechazado, etc.)  │
│   - estado: EstadoListaEspera (activo, procesado)           │
│                                                             │
│ Invariants:                                                 │
│   - Posición única en lista activa                          │
│   - No puede haber solicitudes duplicadas (mismo DNI)       │
│   - Solo socios en estado aspirante pueden estar en lista   │
│   - Orden cronológico estricto por fechaEntrada             │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `agregarALista(solicitud)` → añade al final, calcula posición
- `procesarSiguiente()` → retorna siguiente en cola, marca procesado
- `retirarDeLista(motivo)` → saca de la cola (voluntario, expiración)
- `recalcularPosiciones()` → reordena tras bajas

#### 3.2.7 Aggregate: ExpedienteDisciplinario

```
┌─────────────────────────────────────────────────────────────┐
│ EXPEDIENTE_DISCIPLINARIO (Aggregate Root)                   │
├─────────────────────────────────────────────────────────────┤
│ Identity: ExpedienteId                                      │
│                                                             │
│ Value Objects:                                              │
│   - TipoInfraccion (enum: leve, grave, muy grave)           │
│   - TipoSancion (enum: amonestacion, suspension, expulsion) │
│   - EstadoExpediente (enum: abierto, en_revision, cerrado)  │
│                                                             │
│ Entities:                                                   │
│   - InfraccionRegistrada[] (detalle de cada infracción)     │
│                                                             │
│ Properties:                                                 │
│   - socioId: SocioId                                        │
│   - numeroExpediente: string (correlativo)                  │
│   - fechaApertura: Date                                     │
│   - fechaCierre: Date?                                      │
│   - sancionAplicada: TipoSancion?                           │
│   - diasSuspension: int? (si sancion es suspension)         │
│   - motivoCierre: string?                                   │
│   - resolvidoPor: UsuarioId?                                │
│                                                             │
│ Invariants:                                                 │
│   - Número de expediente único dentro del tenant            │
│   - Fecha sanción <= fecha actual                           │
│   - Expediente cerrado no puede modificarse                 │
│   - Si diasSuspension definido, sancion debe ser suspension │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `abrirExpediente(socio, infraccion)` → crea con estado abierto
- `añadirInfraccion(detalle)` → registra nueva infracción al expediente
- `aplicarSancion(tipo, dias?)` → asigna sanción y cierra expediente
- `archivar(motivo)` → cierra sin sanción (sobreseimiento)

### 3.3 Value Objects

| Value Object | Atributos | Validaciones |
|--------------|-----------|--------------|
| `SocioId` | uuid | UUID v4 válido |
| `NumeroSocio` | valor: string | Formato configurable por tenant |
| `DatosPersonales` | nombre, apellidos, fechaNacimiento | Nombre no vacío, fecha pasada |
| `DatosContacto` | email, telefono, direccion | Email válido, teléfono formato ES |
| `DocumentoIdentidad` | tipo (DNI/NIE/Pasaporte), numero | Algoritmo validación según tipo |
| `DatosBancarios` | iban | IBAN válido (algoritmo mod 97) |
| `EstadoSocio` | enum | Activo, PendientePago, Suspendido, BajaVoluntaria, BajaImpago, BajaDisciplinaria, Aspirante, Fallecido |
| `RangoEdad` | minima, maxima | min >= 0, max > min (si definidos) |
| `CodigoQR` | valor: string | Hash único, no predecible |
| `PosicionEspera` | numero: int | Orden cronológico en lista espera |
| `FechaInscripcion` | fecha: DateTime | Timestamp de entrada en lista |
| `MotivoSalidaLista` | enum | Aprobado, Rechazado, Expirado, Voluntario |
| `EstadoListaEspera` | enum | Activo, Procesado |
| `TipoInfraccion` | enum | Leve, Grave, MuyGrave |
| `TipoSancion` | enum | Amonestacion, Suspension, Expulsion |
| `EstadoExpediente` | enum | Abierto, EnRevision, Cerrado |

### 3.4 Domain Events

| Evento | Trigger | Payload | Consumidores |
|--------|---------|---------|--------------|
| `SocioRegistrado` | Alta completada | socioId, tipoSocio, fecha | BC-Tesoreria (generar cuota), BC-Comunicacion (bienvenida) |
| `SocioDadoDeBaja` | Baja cualquier tipo | socioId, tipoBaja, motivo, fecha | BC-Tesoreria (anular pendientes), BC-Comunicacion (notificar) |
| `EstadoSocioCambiado` | Cambio de estado | socioId, estadoAnterior, estadoNuevo, motivo | BC-Tesoreria (suspender/reactivar cobros) |
| `TipoSocioCambiado` | Cambio de categoría | socioId, tipoAnterior, tipoNuevo | BC-Tesoreria (ajustar cuota) |
| `DatosSocioActualizados` | Modificación datos | socioId, camposModificados | BC-Tesoreria (si IBAN), BC-Comunicacion (si email) |
| `CarnetValidado` | Escaneo QR exitoso | carnetId, socioId, timestamp, ubicacion? | BC-Eventos (check-in automático) |
| `EjercicioAbierto` | Apertura ejercicio | ejercicioId, periodo | BC-Tesoreria (generar cuotas), BC-Membresia (arrastrar socios) |
| `EjercicioCerrado` | Cierre ejercicio | ejercicioId, estadisticas | BC-Documentos (generar memoria) |
| `SolicitudAltaIniciada` | Nueva solicitud | solicitudId, datos | BC-Comunicacion (notificar junta) |
| `SolicitudAltaAprobada` | Aprobación | solicitudId, socioId | BC-Comunicacion (notificar aspirante) |
| `TipoSocioCreado` | Creación de tipo de socio | tipoSocioId, nombre, descripcion, tenantId | BC-Tesoreria (vincular planes de cuota) |
| `BajaPorImpago` | Baja automática por morosidad | socioId, deudaTotal, fechaBaja | BC-Comunicacion (notificar), BC-Tesoreria (cerrar cuenta) |

### 3.5 Trazabilidad RF

| RF | Elemento de Dominio |
|----|---------------------|
| N3RF01 | Socio (DatosPersonales, DatosContacto, DatosBancarios) |
| N3RF02-05 | Socio.CamposPersonalizados (extensión por tipo colectividad) |
| N3RF06 | EstadoSocio (Value Object enum) |
| N3RF07-10 | TipoSocio (Aggregate) |
| N3RF11 | TipoSocio.configuracion (reglas) |
| N3RF12-13 | Socio.HistorialEstados, calcularAntiguedad() |
| N3RF14 | Domain Service: EstadisticasSocios |
| N3RF15-19 | Ejercicio (Aggregate) |
| N3RF20-23 | SolicitudAlta (Aggregate 3.2.3), workflow estados, ListaEspera (Aggregate 3.2.6) |
| N3RF24-27 | Socio.darDeBaja(), eventos de baja, ExpedienteDisciplinario (Aggregate 3.2.7) |
| N3RF28-29 | SolicitudAlta con prioridad (lista espera), ListaEspera (Aggregate 3.2.6) |
| N3RF30-32 | Carnet (Aggregate) |
| N3RF33-34 | Carnet específico cofradías (PapeletaSitio - extensión) |

---

## 4. BC-Tesoreria: Gestión Económica

### 4.1 Descripción

Responsable de toda la gestión económica: definición de cuotas, generación de cargos, registro de cobros, gestión de morosidad, generación de remesas SEPA y contabilidad básica.

### 4.2 Aggregates

#### 4.2.1 Aggregate: CuentaSocio

```
┌─────────────────────────────────────────────────────────────────┐
│ CUENTA_SOCIO (Aggregate Root)                                   │
├─────────────────────────────────────────────────────────────────┤
│ Identity: CuentaSocioId                                         │
│                                                                 │
│ References:                                                     │
│   - socioId: SocioId (referencia a BC-Membresia)                │
│                                                                 │
│ Entities:                                                       │
│   - SuscripcionCuota[] (planes de cuota activos e histórico)    │  ← AÑADIR
│   - Cargo[] (deudas pendientes o histórico)                     │
│   - Pago[] (cobros realizados)                                  │
│   - MandatoSepa? (autorización domiciliación)                   │
│                                                                 │
│ Computed:                                                       │
│   - saldoPendiente: Dinero                                      │
│   - estadoMorosidad: EstadoMorosidad                            │
│                                                                 │
│ Invariants:                                                     │
│   - Un socio tiene exactamente una cuenta                       │
│   - Saldo pendiente = suma(cargos) - suma(pagos)                │
│   - Máximo una suscripción periódica activa por plan            │
└─────────────────────────────────────────────────────────────────┘
```

#### 4.2.2 Entity: Cargo (dentro de CuentaSocio)

```
┌─────────────────────────────────────────────────────────────────┐
│ CARGO (Entity)                                                  │
├─────────────────────────────────────────────────────────────────┤
│ Identity: CargoId                                               │
│                                                                 │
│ Value Objects:                                                  │
│   - Dinero (importeBase - antes de prorrateo)                   │
│   - Dinero (importeFinal - importe efectivo a cobrar)           │
│   - ConceptoCargo (descripcion, ejercicio)                      │
│                                                                 │
│ Properties:                                                     │
│   - suscripcionId: SuscripcionId? (NULL si cargo manual)        │
│   - periodoMes: int? (1-12, NULL para cargos únicos/manuales)   │
│   - fechaEmision: Date                                          │
│   - fechaVencimiento: Date                                      │
│   - estado: EstadoCargo                                         │
│   - importePagado: Dinero (para pagos parciales)                │
│   - esProrrateo: boolean                                        │
│   - esManual: boolean (true si cargo directo sin suscripción)   │
│                                                                 │
│ Invariants:                                                     │
│   - ImporteFinal > 0                                            │
│   - FechaVencimiento >= FechaEmision                            │
│   - ImportePagado <= ImporteFinal                               │
│   - Si esManual=true, suscripcionId debe ser NULL               │
│   - Si esManual=false, suscripcionId debe existir               │
└─────────────────────────────────────────────────────────────────┘
```

#### 4.2.3 Entity: Pago (dentro de CuentaSocio)

```
┌─────────────────────────────────────────────────────────────┐
│ PAGO (Entity)                                               │
├─────────────────────────────────────────────────────────────┤
│ Identity: PagoId                                            │
│                                                             │
│ Value Objects:                                              │
│   - Dinero (importe, moneda)                                │
│   - MetodoPago (tipo, referencia)                           │
│                                                             │
│ Properties:                                                 │
│   - cargoId: CargoId (cargo que liquida)                    │
│   - fechaPago: Date                                         │
│   - fechaRegistro: Date                                     │
│   - registradoPor: UsuarioId                                │
│   - justificanteId: DocumentoId? (referencia BC-Documentos) │
│   - estado: EstadoPago                                      │
│                                                             │
│ Invariants:                                                 │
│   - Importe > 0                                             │
│   - FechaPago <= FechaRegistro                              │
└─────────────────────────────────────────────────────────────┘
```

#### 4.2.4 Entity: MandatoSepa (dentro de CuentaSocio)

```
┌─────────────────────────────────────────────────────────────┐
│ MANDATO_SEPA (Entity)                                       │
├─────────────────────────────────────────────────────────────┤
│ Identity: MandatoId                                         │
│                                                             │
│ Value Objects:                                              │
│   - ReferenciaMandato (único por acreedor)                  │
│   - DatosBancarios (iban del deudor)                        │
│                                                             │
│ Properties:                                                 │
│   - fechaFirma: Date                                        │
│   - fechaUltimoAdeudo: Date?                                │
│   - estado: EstadoMandato (activo, revocado, caducado)      │
│   - documentoFirmado: DocumentoId?                          │
│                                                             │
│ Invariants:                                                 │
│   - Caduca si fechaUltimoAdeudo > 36 meses                  │
│   - Referencia única por acreedor                           │
└─────────────────────────────────────────────────────────────┘
```

#### 4.2.5 Aggregate: PlanCuota

```
┌─────────────────────────────────────────────────────────────────┐
│ PLAN_CUOTA (Aggregate Root)                                     │
├─────────────────────────────────────────────────────────────────┤
│ Identity: PlanCuotaId                                           │
│                                                                 │
│ Value Objects:                                                  │
│   - Dinero (importe)                                            │
│   - Periodicidad (enum orientativo: Mensual, Trimestral,        │
│                   Semestral, Anual, Personalizada)              │
│                                                                 │
│ Properties:                                                     │
│   - codigo: string                                              │
│   - nombre: string                                              │
│   - descripcion: string?                                        │
│   - tipo: TipoPlan (UNICA | PERIODICA)                          │
│   - mesesCobro: int[] (ej: [1,4,7,10] para trimestral)          │
│   - activo: boolean                                             │
│                                                                 │
│ Invariants:                                                     │
│   - Código único dentro del tenant                              │
│   - Si tipo=PERIODICA, mesesCobro no puede estar vacío          │
│   - Si tipo=UNICA, mesesCobro debe estar vacío                  │
│   - Importe >= 0 (puede ser 0 para planes especiales)           │
└─────────────────────────────────────────────────────────────────┘
```

#### 4.2.6 Entity: TipoSocioPlanCuota (Relación N:M)

```
┌─────────────────────────────────────────────────────────────────┐
│ TIPO_SOCIO_PLAN_CUOTA (Entity - tabla intermedia)               │
├─────────────────────────────────────────────────────────────────┤
│ Identity: Composite (tipoSocioId + planCuotaId)                 │
│                                                                 │
│ Properties:                                                     │
│   - tipoSocioId: TipoSocioId (ref BC-Membresia)                 │
│   - planCuotaId: PlanCuotaId                                    │
│   - esDefault: boolean (plan por defecto para este tipo)        │
│   - orden: int (orden de presentación en UI)                    │
│   - activo: boolean                                             │
│                                                                 │
│ Invariants:                                                     │
│   - Solo un plan puede ser default por tipo de socio            │
│   - Orden único por tipo de socio                               │
└─────────────────────────────────────────────────────────────────┘
```

#### 4.2.6 Aggregate: RemesaSepa

```
┌─────────────────────────────────────────────────────────────┐
│ REMESA_SEPA (Aggregate Root)                                │
├─────────────────────────────────────────────────────────────┤
│ Identity: RemesaId                                          │
│                                                             │
│ Value Objects:                                              │
│   - IdentificadorAcreedor (CIF + sufijo Banco España)       │
│   - Dinero (importeTotal)                                   │
│                                                             │
│ Entities:                                                   │
│   - AdeudoSepa[] (cada cobro individual de la remesa)       │
│                                                             │
│ Properties:                                                 │
│   - fechaCreacion: Date                                     │
│   - fechaCargo: Date (fecha valor en banco)                 │
│   - estado: EstadoRemesa                                    │
│   - ficheroXml: string? (path al fichero generado)          │
│                                                             │
│ Invariants:                                                 │
│   - FechaCargo >= FechaCreacion + 3 días hábiles            │
│   - ImporteTotal = suma(adeudos.importe)                    │
│   - Todos los adeudos deben tener mandato válido            │
└─────────────────────────────────────────────────────────────┘
```

#### 4.2.7 Entity: SuscripcionCuota (dentro de CuentaSocio)

```
┌─────────────────────────────────────────────────────────────────┐
│ SUSCRIPCION_CUOTA (Entity)                                      │
├─────────────────────────────────────────────────────────────────┤
│ Identity: SuscripcionId                                         │
│                                                                 │
│ Value Objects:                                                  │
│   - Dinero (importeEfectivo - importe final tras descuentos)    │
│                                                                 │
│ Properties:                                                     │
│   - planCuotaId: PlanCuotaId                                    │
│   - fechaAlta: Date                                             │
│   - fechaBaja: Date?                                            │
│   - descuento: decimal? (porcentaje: 0.30 = 30%)                │
│   - motivoBaja: MotivoBajaSuscripcion?                          │
│                                                                 │
│ Computed:                                                       │
│   - importeEfectivo = planCuota.importe * (1 - descuento)       │
│                                                                 │
│ Invariants:                                                     │
│   - fechaBaja >= fechaAlta (si existe)                          │
│   - descuento entre 0 y 1 (si existe)                           │
│   - Si planCuota.tipo=UNICA, fechaBaja se asigna automáticamente│
│     al generar el cargo                                         │
└─────────────────────────────────────────────────────────────────┘

MotivoBajaSuscripcion (enum):
  - CAMBIO_PLAN: Socio cambió a otra modalidad de pago
  - BAJA_SOCIO: Socio dado de baja de la entidad
  - EXENCION: Socio exento de pago
  - FIN_CUOTA_UNICA: Cuota única completada (automático)
```

#### 4.2.8 Entity: AdeudoSepa (dentro de RemesaSepa)

```
┌─────────────────────────────────────────────────────────────┐
│ ADEUDO_SEPA (Entity)                                        │
├─────────────────────────────────────────────────────────────┤
│ Identity: AdeudoId                                          │
│                                                             │
│ Value Objects:                                              │
│   - Dinero (importe)                                        │
│   - SecuenciaSepa (FRST, RCUR, OOFF, FNAL)                  │
│                                                             │
│ Properties:                                                 │
│   - cargoId: CargoId                                        │
│   - mandatoId: MandatoId                                    │
│   - estado: EstadoAdeudo                                    │
│   - motivoDevolucion: string?                               │
│   - fechaDevolucion: Date?                                  │
│                                                             │
│ Invariants:                                                 │
│   - Secuencia correcta según historial del mandato          │
└─────────────────────────────────────────────────────────────┘
```

#### 4.2.9 Aggregate: Movimiento (Contabilidad)

```
┌─────────────────────────────────────────────────────────────┐
│ MOVIMIENTO (Aggregate Root)                                 │
├─────────────────────────────────────────────────────────────┤
│ Identity: MovimientoId                                      │
│                                                             │
│ Value Objects:                                              │
│   - Dinero (importe)                                        │
│   - CategoriaContable (cuenta según plan ENL si aplica)     │
│                                                             │
│ Properties:                                                 │
│   - tipo: TipoMovimiento (ingreso, gasto)                   │
│   - concepto: string                                        │
│   - fecha: Date                                             │
│   - ejercicioId: EjercicioId                                │
│   - pagoId: PagoId? (si viene de un pago)                   │
│   - justificanteId: DocumentoId?                            │
│   - registradoPor: UsuarioId                                │
│                                                             │
│ Invariants:                                                 │
│   - Importe > 0                                             │
│   - Fecha dentro del ejercicio                              │
└─────────────────────────────────────────────────────────────┘
```

#### 4.2.10 Aggregate: EnlacePago

```
┌─────────────────────────────────────────────────────────────┐
│ ENLACE_PAGO (Aggregate Root)                                │
├─────────────────────────────────────────────────────────────┤
│ Identity: EnlacePagoId                                      │
│                                                             │
│ Value Objects:                                              │
│   - URLFirmada (token único criptográfico)                  │
│   - EstadoEnlace (enum: pendiente, pagado, expirado)        │
│                                                             │
│ Properties:                                                 │
│   - cargoId: CargoId (cargo a liquidar)                     │
│   - socioId: SocioId                                        │
│   - urlPublica: string (URL completa con token)             │
│   - token: string (hash único no predecible)                │
│   - fechaCreacion: DateTime                                 │
│   - fechaExpiracion: DateTime (por defecto 48h)             │
│   - fechaPago: DateTime? (cuando se completa)               │
│   - pagoId: PagoId? (pago asociado al completar)            │
│   - estado: EstadoEnlace                                    │
│   - intentosAcceso: int (contador de seguridad)             │
│                                                             │
│ Invariants:                                                 │
│   - URL con token único en todo el sistema                  │
│   - FechaExpiracion > FechaCreacion                         │
│   - Un solo pago exitoso por enlace                         │
│   - Estado pagado requiere pagoId definido                  │
│   - Enlace expirado o pagado no puede reutilizarse          │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `generarEnlace(cargo, validezHoras)` → crea URL firmada
- `validarAcceso(token)` → verifica vigencia y estado
- `marcarComoPagado(pago)` → cierra enlace tras pago exitoso
- `expirar()` → marca como expirado tras vencimiento

#### 4.2.11 Aggregate: TurnoCaja

```
┌─────────────────────────────────────────────────────────────┐
│ TURNO_CAJA (Aggregate Root)                                 │
├─────────────────────────────────────────────────────────────┤
│ Identity: TurnoCajaId                                       │
│                                                             │
│ Value Objects:                                              │
│   - Dinero (importeApertura, importeCierre)                 │
│                                                             │
│ Entities:                                                   │
│   - MovimientoCaja[] (operaciones del turno)                │
│                                                             │
│ Properties:                                                 │
│   - encargado: SocioId (responsable del turno)              │
│   - fechaApertura: DateTime                                 │
│   - fechaCierre: DateTime?                                  │
│   - importeApertura: Dinero (efectivo inicial)              │
│   - importeCierre: Dinero? (efectivo final)                 │
│   - estado: EstadoTurno (abierto, cerrado, cuadrado)        │
│   - diferenciaContable: Dinero? (descuadre si existe)       │
│   - observaciones: string?                                  │
│                                                             │
│ Computed:                                                   │
│   - importeEsperado = apertura + ingresos - reintegros      │
│                                                             │
│ Invariants:                                                 │
│   - Solo un turno abierto por caja simultáneamente          │
│   - FechaCierre >= FechaApertura (si definida)              │
│   - ImporteCierre debe estar definido si estado=cerrado     │
│   - DiferenciaContable = importeEsperado - importeCierre    │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `abrirTurno(encargado, importeInicial)` → inicia turno
- `registrarMovimiento(tipo, importe)` → añade operación
- `cerrarTurno(importeFinal)` → calcula diferencia y cierra
- `cuadrarTurno(ajuste, motivo)` → ajusta descuadres

#### 4.2.12 Aggregate: CategoriaContable

```
┌─────────────────────────────────────────────────────────────┐
│ CATEGORIA_CONTABLE (Aggregate Root)                         │
├─────────────────────────────────────────────────────────────┤
│ Identity: CategoriaContableId                               │
│                                                             │
│ Value Objects:                                              │
│   - CodigoCuenta (según plan contable ENL o personalizado)  │
│                                                             │
│ Properties:                                                 │
│   - codigo: string (ej: "7.1", "4.2.3")                     │
│   - nombre: string                                          │
│   - descripcion: string?                                    │
│   - tipo: TipoCategoria (ingreso, gasto, activo, pasivo)    │
│   - categoriaPadre: CategoriaContableId? (jerarquía)        │
│   - nivel: int (profundidad en árbol)                       │
│   - esImputable: boolean (permite asignar movimientos)      │
│   - esSistema: boolean (predefinida, no editable)           │
│   - activa: boolean                                         │
│                                                             │
│ Invariants:                                                 │
│   - Código único dentro del tenant                          │
│   - No ciclos en jerarquía de categorías                    │
│   - Categorías sistema no pueden eliminarse ni editarse     │
│   - Solo categorías hoja (sin hijos) pueden ser imputables  │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `crearCategoria(codigo, nombre, padre?)` → valida jerarquía
- `marcarComoImputable()` → permite asignar movimientos
- `desactivar()` → impide nuevas asignaciones

#### 4.2.13 Aggregate: EjercicioContable

```
┌─────────────────────────────────────────────────────────────┐
│ EJERCICIO_CONTABLE (Aggregate Root)                         │
├─────────────────────────────────────────────────────────────┤
│ Identity: EjercicioContableId                               │
│                                                             │
│ Value Objects:                                              │
│   - PeriodoContable (fechaInicio, fechaFin)                 │
│   - Dinero (saldoInicial, saldoFinal)                       │
│                                                             │
│ Properties:                                                 │
│   - nombre: string (ej: "Ejercicio 2026")                   │
│   - fechaInicio: Date                                       │
│   - fechaFin: Date                                          │
│   - estado: EstadoEjercicioContable (abierto, cerrado)      │
│   - saldoInicial: Dinero                                    │
│   - saldoFinal: Dinero? (calculado al cierre)               │
│   - totalIngresos: Dinero (acumulado)                       │
│   - totalGastos: Dinero (acumulado)                         │
│   - ejercicioAnterior: EjercicioContableId?                 │
│   - fechaCierre: Date? (cuando se cierra)                   │
│   - cerradoPor: UsuarioId?                                  │
│                                                             │
│ Computed:                                                   │
│   - resultado = totalIngresos - totalGastos                 │
│                                                             │
│ Invariants:                                                 │
│   - Solo un ejercicio contable abierto simultáneamente      │
│   - FechaFin > FechaInicio                                  │
│   - Periodos no pueden solaparse con otros ejercicios       │
│   - Ejercicio cerrado no admite nuevos movimientos          │
│   - SaldoFinal = saldoInicial + resultado (al cierre)       │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `abrirEjercicio(periodo, saldoInicial)` → inicia ejercicio
- `registrarMovimiento(tipo, importe)` → actualiza acumulados
- `cerrarEjercicio()` → calcula saldo final, congela
- `reaperturar(motivo)` → reabre por correcciones (auditoría)

### 4.3 Value Objects

| Value Object | Atributos | Validaciones |
|--------------|-----------|--------------|
| `Dinero` | cantidad: decimal, moneda: string | cantidad >= 0, moneda ISO 4217 (default EUR) |
| `MetodoPago` | tipo: enum, referencia: string | Tipos: Efectivo, Transferencia, Domiciliacion, Bizum, Tarjeta |
| `EstadoCargo` | enum | Pendiente, Pagado, PagadoParcial, Anulado |
| `EstadoPago` | enum | Confirmado, Devuelto, Anulado |
| `EstadoMorosidad` | enum | AlCorriente, MorosidadLeve, MorosidadGrave, Suspendido |
| `SecuenciaSepa` | enum | FRST, RCUR, OOFF, FNAL |
| `EstadoRemesa` | enum | Borrador, Generada, Enviada, Procesada, ConDevoluciones |
| `IdentificadorAcreedor` | valor: string | Formato ES + 2 dígitos + sufijo (14 chars) |
| `Periodicidad` | enum | Mensual, Trimestral, Semestral, Anual, Personalizada (orientativo, la configuración real está en mesesCobro[]) |
| `TipoPlan` | enum | UNICA, PERIODICA |
| `MotivoBajaSuscripcion` | enum | CAMBIO_PLAN, BAJA_SOCIO, EXENCION, FIN_CUOTA_UNICA |
| `MesesCobro` | int[] | Array de meses (1-12) en que se generan cargos. Vacío para planes UNICA. |
| `URLFirmada` | token: string | Hash criptográfico único para enlaces de pago |
| `EstadoEnlace` | enum | Pendiente, Pagado, Expirado |
| `EstadoTurno` | enum | Abierto, Cerrado, Cuadrado |
| `CodigoCuenta` | codigo: string | Según plan ENL o personalizado (ej: "7.1") |
| `TipoCategoria` | enum | Ingreso, Gasto, Activo, Pasivo |
| `EstadoEjercicioContable` | enum | Abierto, Cerrado |
| `PeriodoContable` | fechaInicio: Date, fechaFin: Date | Periodo fiscal del ejercicio |

### 4.4 Domain Events

| Evento | Trigger | Payload | Consumidores |
|--------|---------|---------|--------------|
| `CargoGenerado` | Creación de cargo | cargoId, socioId, importe, concepto | BC-Comunicacion (aviso) |
| `PagoRegistrado` | Cobro confirmado | pagoId, cargoId, socioId, importe, metodo | BC-Membresia (actualizar estado si procede) |
| `PagoDevuelto` | Devolución bancaria | pagoId, cargoId, motivo | BC-Comunicacion (notificar), BC-Membresia (marcar morosidad) |
| `MorosidadDetectada` | Cargo vencido sin pago | socioId, cargoId, diasVencido | BC-Comunicacion (workflow avisos) |
| `PlanCuotaCreado` | Creación de plan | planCuotaId, codigo, nombre, tipo, importe | BC-Membresia (invalidar caché) |
| `PlanCuotaModificado` | Modificación plan | planCuotaId, camposModificados | BC-Membresia (invalidar caché) |
| `PlanCuotaVinculadoATipoSocio` | Vinculación N:M | planCuotaId, tipoSocioId, esDefault | BC-Membresia (invalidar caché) |
| `CargoPagado` | Pago de cargo | cargoId, socioId, importe, fechaPago, metodoPago | BC-Comunicacion (enviar recibo), BC-Membresia (actualizar morosidad) |
| `CargoCobrado` | Cobro efectivo de cargo | cargoId, socioId, importe, fechaCobro, remesaId? | BC-Comunicacion (confirmar pago) |
| `CargoMarcadoReintento` | Marcado para reintento de cobro | cargoId, socioId, intentoNumero, proximaFecha | BC-Comunicacion (avisar socio) |
| `ReciboGenerado` | Generación de recibo PDF | reciboId, pagoId, numeroRecibo, fechaEmision | BC-Comunicacion (enviar por email), BC-Documentos (archivar) |
| `MandatoSepaRegistrado` | Registro mandato SEPA | mandatoId, socioId, iban, fechaFirma, estado | BC-Tesoreria (habilitar domiciliación) |
| `MandatoSepaRevocado` | Revocación mandato SEPA | mandatoId, socioId, motivoRevocacion, fechaRevocacion | BC-Tesoreria (deshabilitar domiciliación), BC-Comunicacion (notificar) |
| `RemesaSepaGenerada` | Generación fichero SEPA XML | remesaId, fechaCobro, totalAdeudos, importeTotal, identificadorAcreedor | BC-Comunicacion (avisar socios 2 días antes) |
| `EnlacePagoGenerado` | Generación enlace pago online | cargoId, socioId, url, fechaExpiracion | BC-Comunicacion (enviar email con enlace) |
| `MorosidadRegularizada` | Regularización de morosidad | socioId, importePagado, fechaRegularizacion | BC-Membresia (restaurar estado), BC-Comunicacion (confirmar) |
| `CertificadoDescubiertoGenerado` | Certificado de descubierto | certificadoId, socioId, deudaTotal, fechaEmision | BC-Documentos (archivar), BC-Comunicacion (notificar socio) |
| `SuscripcionCreada` | Creación suscripción cuota | suscripcionId, socioId, planCuotaId, fechaInicio, estado | BC-Tesoreria (programar generación mensual) |
| `SuscripcionModificada` | Modificación suscripción | suscripcionId, camposModificados[], fechaModificacion | BC-Tesoreria (recalcular próximos cargos) |
| `SuscripcionCerrada` | Cierre de suscripción | suscripcionId, motivoCierre, fechaCierre | BC-Tesoreria (detener generación) |
| `GeneracionMensualCompletada` | Generación mensual de cuotas | ejercicioId, mes, totalCargosGenerados, importeTotal | BC-Comunicacion (notificar tesorero), Sistema de auditoría |
| `DescuadreDetectado` | Detección de descuadre | diferencia, cuentaId, fechaDeteccion | BC-Comunicacion (alertar tesorero) |

### 4.5 Domain Services

| Servicio | Responsabilidad |
|----------|-----------------|
| `GeneradorCargos` | Genera cargos para suscripciones activas cuyo plan incluya el mes actual en mesesCobro. Proceso mensual automatizable. |
| `GeneradorCargoManual` | Crea cargos puntuales sin suscripción asociada (derramas, penalizaciones, ajustes) |
| `CalculadorProrrateo` | Calcula cuota proporcional para altas a mitad de ejercicio |
| `GeneradorRemesaSepa` | Crea fichero XML ISO 20022 pain.008.001.08 |
| `GestorMorosidad` | Evalúa y ejecuta workflow de morosidad |
| `ConciliadorPagos` | Asocia pagos de pasarela con cargos pendientes |
| `GestorSuscripciones` | Gestiona altas, bajas y cambios de modalidad de pago |

### 4.6 Trazabilidad RF

| RF | Elemento de Dominio |
|----|---------------------|
| N4RF01 | PlanCuota (Aggregate), TipoSocioPlanCuota |
| N4RF02 | Domain Service: GeneradorCargos (basado en suscripciones activas y mesesCobro) |
| N4RF03 | Domain Service: CalculadorProrrateo |
| N4RF04-05 | SuscripcionCuota.descuento, exenciones |
| N4RF06 | SuscripcionCuota (selección modalidad al alta) |
| N4RF07 | SuscripcionCuota.motivoBaja=CAMBIO_PLAN |
| N4RF08 | Domain Service: GeneradorCargoManual, Cargo.esManual=true |
| N4RF09-11 | Pago (Entity), MetodoPago, EstadoPago |
| N4RF12-13 | Pago.justificanteId, generación recibo |
| N4RF14-16 | Domain Service: GestorMorosidad, eventos morosidad |
| N4RF17-23 | RemesaSepa, AdeudoSepa, MandatoSepa |
| N4RF24-27 | Integración pasarela (Application Service), EnlacePago (Aggregate 4.2.10) |
| N4RF28-33 | Movimiento (Aggregate 4.2.9), CategoriaContable (Aggregate 4.2.12), EjercicioContable (Aggregate 4.2.13) |
| N4RF34-38 | Extensión: TurnoCaja (Aggregate 4.2.11, específico peñas) |

---

## 5. BC-Eventos: Actividades y Participación

### 5.1 Descripción

Gestiona el ciclo de vida de eventos y actividades: planificación, inscripciones, control de aforo y registro de asistencia.

### 5.2 Aggregates

#### 5.2.1 Aggregate: Evento

```
┌─────────────────────────────────────────────────────────────┐
│ EVENTO (Aggregate Root)                                     │
├─────────────────────────────────────────────────────────────┤
│ Identity: EventoId                                          │
│                                                             │
│ Value Objects:                                              │
│   - PeriodoEvento (fechaInicio, fechaFin)                   │
│   - Ubicacion (direccion, coordenadas?, sala?)              │
│   - ConfiguracionInscripcion (fechaApertura, fechaCierre,   │
│                               requierePago, precio?)        │
│                                                             │
│ Entities:                                                   │
│   - Inscripcion[] (participantes inscritos)                 │
│                                                             │
│ Properties:                                                 │
│   - tipoEvento: TipoEventoId                                │
│   - nombre: string                                          │
│   - descripcion: string                                     │
│   - aforo: int? (null = ilimitado)                          │
│   - estado: EstadoEvento                                    │
│   - ejercicioId: EjercicioId                                │
│   - organizadorId: UsuarioId                                │
│                                                             │
│ Computed:                                                   │
│   - plazasDisponibles: int                                  │
│   - inscripcionAbierta: boolean                             │
│                                                             │
│ Invariants:                                                 │
│   - FechaFin >= FechaInicio                                 │
│   - Inscripciones.count <= Aforo (si definido)              │
│   - FechaCierreInscripcion <= FechaInicio                   │
└─────────────────────────────────────────────────────────────┘
```

#### 5.2.2 Entity: Inscripcion (dentro de Evento)

```
┌─────────────────────────────────────────────────────────────┐
│ INSCRIPCION (Entity)                                        │
├─────────────────────────────────────────────────────────────┤
│ Identity: InscripcionId                                     │
│                                                             │
│ Value Objects:                                              │
│   - DatosInscripcion (campos configurables por evento)      │
│                                                             │
│ Properties:                                                 │
│   - socioId: SocioId? (null si externo)                     │
│   - datosExterno: DatosPersonales? (si no es socio)         │
│   - fechaInscripcion: Date                                  │
│   - estado: EstadoInscripcion                               │
│   - cargoId: CargoId? (si requiere pago)                    │
│   - asistencia: Asistencia?                                 │
│                                                             │
│ Invariants:                                                 │
│   - O socioId o datosExterno, no ambos                      │
│   - Si requierePago, cargoId debe existir                   │
└─────────────────────────────────────────────────────────────┘
```

#### 5.2.3 Value Object: Asistencia

```
┌─────────────────────────────────────────────────────────────┐
│ ASISTENCIA (Value Object)                                   │
├─────────────────────────────────────────────────────────────┤
│ Properties:                                                 │
│   - confirmada: boolean                                     │
│   - horaEntrada: DateTime?                                  │
│   - metodoCheckin: MetodoCheckin (QR, Manual)               │
│   - registradoPor: UsuarioId?                               │
└─────────────────────────────────────────────────────────────┘
```

#### 5.2.4 Aggregate: TipoEvento

```
┌─────────────────────────────────────────────────────────────┐
│ TIPO_EVENTO (Aggregate Root)                                │
├─────────────────────────────────────────────────────────────┤
│ Identity: TipoEventoId                                      │
│                                                             │
│ Properties:                                                 │
│   - codigo: string                                          │
│   - nombre: string                                          │
│   - color: string (para calendario)                         │
│   - requiereInscripcion: boolean                            │
│   - camposInscripcion: CampoFormulario[]                    │
│   - activo: boolean                                         │
│                                                             │
│ Invariants:                                                 │
│   - Código único dentro del tenant                          │
└─────────────────────────────────────────────────────────────┘
```

#### 5.2.5 Aggregate: EventoComida

```
┌─────────────────────────────────────────────────────────────┐
│ EVENTO_COMIDA (Aggregate Root)                              │
├─────────────────────────────────────────────────────────────┤
│ Identity: EventoComidaId                                    │
│                                                             │
│ Value Objects:                                              │
│   - OpcionMenu (nombre, precio)                             │
│   - DatosRestaurante (nombre, direccion, telefono)          │
│                                                             │
│ Entities:                                                   │
│   - ReservaComida[] (inscripciones con selección menú)      │
│                                                             │
│ Properties:                                                 │
│   - eventoId: EventoId (referencia al evento base)          │
│   - restaurante: DatosRestaurante                           │
│   - opcionesMenu: OpcionMenu[] (diferentes menús)           │
│   - fechaLimiteReserva: Date                                │
│   - numeroComensales: int (total confirmado)                │
│   - precioBase: Dinero                                      │
│   - requisitosAlimentarios: string[] (alergias, vegano...)  │
│                                                             │
│ Invariants:                                                 │
│   - FechaLimiteReserva < fechaInicio del evento             │
│   - Al menos una opción de menú disponible                  │
│   - NumeroComensales = suma de reservas confirmadas         │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `agregarOpcionMenu(nombre, precio)` → añade opción al listado
- `realizarReserva(socio, opcionMenu)` → registra inscripción con menú
- `modificarReserva(reserva, nuevaOpcion)` → cambia selección
- `confirmarComensales()` → cierra reservas y genera factura

#### 5.2.6 Aggregate: Cuadrilla

```
┌─────────────────────────────────────────────────────────────┐
│ CUADRILLA (Aggregate Root)                                  │
├─────────────────────────────────────────────────────────────┤
│ Identity: CuadrillaId                                       │
│                                                             │
│ Value Objects:                                              │
│   - ColorIdentificativo (hex o nombre)                      │
│                                                             │
│ Entities:                                                   │
│   - MiembroCuadrilla[] (socios asignados con rol)           │
│                                                             │
│ Properties:                                                 │
│   - nombre: string (ej: "Tambores", "Trompetas")            │
│   - descripcion: string?                                    │
│   - responsable: SocioId (coordinador)                      │
│   - colorIdentificativo: string                             │
│   - capacidadMaxima: int?                                   │
│   - tipoActividad: string (procesion, desfile, etc.)        │
│   - activa: boolean                                         │
│                                                             │
│ Computed:                                                   │
│   - numeroMiembros: int (total activos)                     │
│                                                             │
│ Invariants:                                                 │
│   - Nombre único dentro del tenant                          │
│   - Responsable debe ser miembro de la cuadrilla            │
│   - NumeroMiembros <= capacidadMaxima (si definida)         │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `crearCuadrilla(nombre, responsable)` → inicializa cuadrilla
- `agregarMiembro(socio, rol)` → añade socio validando capacidad
- `removerMiembro(socio, motivo)` → da de baja del grupo
- `asignarAEvento(evento)` → asocia cuadrilla a actividad

#### 5.2.7 Aggregate: Partido

```
┌─────────────────────────────────────────────────────────────┐
│ PARTIDO (Aggregate Root)                                    │
├─────────────────────────────────────────────────────────────┤
│ Identity: PartidoId                                         │
│                                                             │
│ Value Objects:                                              │
│   - ResultadoPartido (golesLocal, golesVisitante)           │
│   - DatosRival (nombre, escudo?)                            │
│   - Ubicacion (campo, direccion)                            │
│                                                             │
│ Entities:                                                   │
│   - Convocatoria[] (jugadores convocados)                   │
│   - EstadisticaJugador[] (goles, tarjetas, minutos)         │
│                                                             │
│ Properties:                                                 │
│   - eventoId: EventoId (referencia al evento base)          │
│   - equipoRival: DatosRival                                 │
│   - esLocal: boolean                                        │
│   - campo: Ubicacion                                        │
│   - categoria: string (senior, juvenil, infantil...)        │
│   - competicion: string (liga, copa, amistoso)              │
│   - resultado: ResultadoPartido?                            │
│   - estado: EstadoPartido (convocado, jugado, suspendido)   │
│   - observaciones: string?                                  │
│                                                             │
│ Invariants:                                                 │
│   - Resultado solo definido si estado=jugado                │
│   - Jugadores convocados deben ser socios activos           │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `crearPartido(rival, fecha, campo)` → programa encuentro
- `convocarJugadores(socios[])` → establece lista de convocados
- `registrarResultado(golesLocal, golesVisitante)` → cierra partido
- `registrarEstadistica(jugador, tipo, valor)` → añade dato individual

### 5.3 Value Objects

| Value Object | Atributos | Validaciones |
|--------------|-----------|--------------|
| `PeriodoEvento` | fechaInicio: DateTime, fechaFin: DateTime | FechaFin >= FechaInicio |
| `Ubicacion` | direccion: string, coordenadas?: LatLng, sala?: string | Dirección no vacía |
| `ConfiguracionInscripcion` | fechaApertura: Date, fechaCierre: Date, requierePago: boolean, precio?: Dinero | FechaCierre <= fechaInicio evento |
| `EstadoEvento` | enum | Borrador, Publicado, Inscripciones Abiertas, Inscripciones Cerradas, Realizado, Cancelado |
| `EstadoInscripcion` | enum | Confirmada, ListaEspera, Cancelada, Asistencia Registrada |
| `MetodoCheckin` | enum | QR, Manual, NFC |
| `OpcionMenu` | nombre: string, precio: Dinero | Nombre no vacío |
| `DatosRestaurante` | nombre: string, direccion: string, telefono: string | Todos obligatorios |
| `ColorIdentificativo` | valor: string | Color hex o nombre CSS válido |
| `ResultadoPartido` | golesLocal: int, golesVisitante: int | >= 0 ambos |
| `DatosRival` | nombre: string, escudo?: URL | Nombre obligatorio |
| `EstadoPartido` | enum | Convocado, Jugado, Suspendido, Aplazado |

### 5.4 Domain Events

| Evento | Trigger | Payload | Consumidores |
|--------|---------|---------|--------------|
| `EventoCreado` | Creación evento | eventoId, tipo, fecha | BC-Comunicacion (publicar) |
| `EventoPublicado` | Apertura inscripciones | eventoId | BC-Comunicacion (notificar socios) |
| `EventoCancelado` | Cancelación | eventoId, motivo | BC-Comunicacion (notificar inscritos), BC-Tesoreria (reembolsos) |
| `InscripcionRealizada` | Nueva inscripción | inscripcionId, eventoId, socioId | BC-Tesoreria (generar cargo si precio), BC-Comunicacion (confirmación) |
| `InscripcionCancelada` | Cancelación inscripción | inscripcionId, eventoId | BC-Tesoreria (anular cargo) |
| `AforoCompletado` | Aforo lleno | eventoId | BC-Comunicacion (activar lista espera) |
| `PlazaLiberada` | Baja de inscrito | eventoId, posicionListaEspera | BC-Comunicacion (notificar siguiente) |
| `ValoracionesEventoSolicitadas` | Solicitud valoraciones post-evento | eventoId, sociosInscritos[], fechaSolicitud | BC-Comunicacion (enviar formulario) |
| `ProblemaRecurrenteDetectado` | Detección patrón de problemas | eventoId, tipoProblema, frecuencia | BC-Comunicacion (alertar organizadores) |

### 5.5 Trazabilidad RF

| RF | Elemento de Dominio |
|----|---------------------|
| N5RF01-02 | Evento (Aggregate), TipoEvento |
| N5RF03-04 | Evento con calendario, exportación iCal |
| N5RF05-09 | Inscripcion (Entity), control aforo, lista espera |
| N5RF10-11 | ConfiguracionInscripcion, DatosInscripcion |
| N5RF12-16 | Asistencia (VO), check-in QR/manual |
| N5RF17-19 | Extensión: EventoComida (específico peñas), Aggregate 5.2.5 |
| N5RF20-26 | Extensión: Procesion, Costaleros (específico cofradías) |
| N5RF27-30 | Extensión: Competicion (específico clubes), Aggregate 5.2.7 Partido, Aggregate 5.2.6 Cuadrilla |

---

## 6. BC-Comunicacion: Notificaciones y Mensajería

### 6.1 Descripción

Gestiona el envío de comunicaciones a socios: emails, notificaciones push, SMS y el tablón de anuncios interno. Procesa eventos de otros BCs para generar notificaciones automáticas.

### 6.2 Aggregates

#### 6.2.1 Aggregate: Comunicacion

```
┌─────────────────────────────────────────────────────────────┐
│ COMUNICACION (Aggregate Root)                               │
├─────────────────────────────────────────────────────────────┤
│ Identity: ComunicacionId                                    │
│                                                             │
│ Value Objects:                                              │
│   - Contenido (asunto, cuerpo, formato)                     │
│   - SegmentoDestinatarios (criterios de filtrado)           │
│                                                             │
│ Entities:                                                   │
│   - Envio[] (cada envío individual a un destinatario)       │
│                                                             │
│ Properties:                                                 │
│   - canal: CanalComunicacion                                │
│   - tipo: TipoComunicacion (manual, automatica)             │
│   - plantillaId: PlantillaId?                               │
│   - fechaProgramada: DateTime?                              │
│   - estado: EstadoComunicacion                              │
│   - creadoPor: UsuarioId                                    │
│                                                             │
│ Invariants:                                                 │
│   - Al menos un destinatario                                │
└─────────────────────────────────────────────────────────────┘
```

#### 6.2.2 Entity: Envio

```
┌─────────────────────────────────────────────────────────────┐
│ ENVIO (Entity)                                              │
├─────────────────────────────────────────────────────────────┤
│ Identity: EnvioId                                           │
│                                                             │
│ Properties:                                                 │
│   - socioId: SocioId                                        │
│   - destino: string (email, teléfono según canal)           │
│   - estado: EstadoEnvio                                     │
│   - fechaEnvio: DateTime?                                   │
│   - fechaApertura: DateTime? (tracking email)               │
│   - errorMensaje: string?                                   │
└─────────────────────────────────────────────────────────────┘
```

#### 6.2.3 Aggregate: Plantilla

```
┌─────────────────────────────────────────────────────────────┐
│ PLANTILLA (Aggregate Root)                                  │
├─────────────────────────────────────────────────────────────┤
│ Identity: PlantillaId                                       │
│                                                             │
│ Properties:                                                 │
│   - codigo: string                                          │
│   - nombre: string                                          │
│   - canal: CanalComunicacion                                │
│   - asunto: string (con placeholders)                       │
│   - cuerpo: string (con placeholders)                       │
│   - variablesDisponibles: string[]                          │
│   - esSistema: boolean (no editable si true)                │
│   - activa: boolean                                         │
│                                                             │
│ Invariants:                                                 │
│   - Código único dentro del tenant                          │
│   - Plantillas sistema no modificables                      │
└─────────────────────────────────────────────────────────────┘
```

#### 6.2.4 Aggregate: Anuncio

```
┌─────────────────────────────────────────────────────────────┐
│ ANUNCIO (Aggregate Root)                                    │
├─────────────────────────────────────────────────────────────┤
│ Identity: AnuncioId                                         │
│                                                             │
│ Properties:                                                 │
│   - titulo: string                                          │
│   - contenido: string                                       │
│   - fechaPublicacion: Date                                  │
│   - fechaExpiracion: Date?                                  │
│   - destacado: boolean                                      │
│   - publicadoPor: UsuarioId                                 │
│   - estado: EstadoAnuncio                                   │
│                                                             │
│ Invariants:                                                 │
│   - FechaExpiracion > FechaPublicacion (si definida)        │
└─────────────────────────────────────────────────────────────┘
```

### 6.3 Domain Events

BC-Comunicacion emite eventos relacionados con el ciclo de vida de las comunicaciones:
| Evento | Trigger | Payload | Consumidores |
|--------|---------|---------|--------------|
| `ComunicacionEnviada` | Envío completado | comunicacionId, totalDestinatarios, canal, fechaEnvio | - |
| `EmailRebotado` | Email rebota (bounce) | envioId, socioId, email, tipoBounce (hard/soft), motivo | BC-Membresia (marcar email inválido si hard bounce) |
| `NotificacionBienvenidaEnviada` | Email bienvenida enviado a nuevo socio | socioId, email, fechaEnvio, plantillaId | - |
| `RecordatorioPagoEnviado` | Recordatorio de pago enviado | socioId, email, cargoId, importe, fechaLimite | - |
| `AvisoMorosidadEnviado` | Aviso de morosidad enviado | socioId, email, deudaTotal, fechaEnvio | - |
| `AvisoDomiciliacionEnviado` | Aviso pre-remesa enviado | socioId, email, remesaId, importe, fechaCargo | - |
| `ConfirmacionInscripcionEnviada` | Confirmación de inscripción a evento | socioId, email, eventoId, inscripcionId | - |

**Notas:**
- `ComunicacionEnviada`: Marca la finalización del proceso de envío masivo
- `EmailAbierto` y `EnlaceClicado`: Eventos de tracking para métricas internas (no requieren consumidores externos)
- `EmailRebotado`: Permite a BC-Membresia actualizar la validez de emails de socios según tipo de bounce (hard/soft)

Este BC **también es consumidor** de eventos de otros BCs:

| Evento Origen | Acción en BC-Comunicacion |
|---------------|---------------------------|
| `SocioRegistrado` | Enviar email bienvenida |
| `CargoGenerado` | Enviar aviso de cuota próxima |
| `MorosidadDetectada` | Iniciar workflow de avisos |
| `EventoPublicado` | Notificar a socios según preferencias |
| `InscripcionRealizada` | Enviar confirmación |

### 6.4 Trazabilidad RF

| RF | Elemento de Dominio |
|----|---------------------|
| N6RF01-04 | Comunicacion (Aggregate), canales |
| N6RF05-07 | SegmentoDestinatarios, Plantilla |
| N6RF08-09 | Envio (Entity), tracking |
| N6RF10-16 | Plantillas sistema (notificaciones automáticas) |
| N6RF17-23 | Extensión: Acta (específico - ver BC-Documentos) |

---

## 7. BC-Documentos: Gestión Documental

### 7.1 Descripción

Repositorio centralizado de documentos de la entidad: estatutos, actas, facturas, justificantes. Incluye gestión del libro de actas obligatorio.

### 7.2 Aggregates

#### 7.2.1 Aggregate: Documento

```
┌─────────────────────────────────────────────────────────────┐
│ DOCUMENTO (Aggregate Root)                                  │
├─────────────────────────────────────────────────────────────┤
│ Identity: DocumentoId                                       │
│                                                             │
│ Value Objects:                                              │
│   - MetadatosDocumento (nombre, descripcion, etiquetas)     │
│   - ArchivoFisico (path, tamaño, mimeType, hash)            │
│                                                             │
│ Properties:                                                 │
│   - categoriaId: CategoriaId                                │
│   - fechaDocumento: Date                                    │
│   - fechaSubida: DateTime                                   │
│   - subidoPor: UsuarioId                                    │
│   - version: int                                            │
│   - documentoAnterior: DocumentoId? (para versionado)       │
│   - estado: EstadoDocumento                                 │
│                                                             │
│ Invariants:                                                 │
│   - Tamaño <= límite del tenant                             │
│   - MimeType en lista permitida                             │
└─────────────────────────────────────────────────────────────┘
```

#### 7.2.2 Aggregate: Categoria

```
┌─────────────────────────────────────────────────────────────┐
│ CATEGORIA (Aggregate Root)                                  │
├─────────────────────────────────────────────────────────────┤
│ Identity: CategoriaId                                       │
│                                                             │
│ Properties:                                                 │
│   - nombre: string                                          │
│   - descripcion: string                                     │
│   - categoriaPadre: CategoriaId? (jerarquía)                │
│   - orden: int                                              │
│   - rolesAcceso: RolId[] (quién puede ver)                  │
│   - esSistema: boolean                                      │
│                                                             │
│ Invariants:                                                 │
│   - No ciclos en jerarquía                                  │
│   - Categorías sistema no eliminables                       │
└─────────────────────────────────────────────────────────────┘
```

#### 7.2.3 Aggregate: Acta

```
┌─────────────────────────────────────────────────────────────┐
│ ACTA (Aggregate Root)                                       │
├─────────────────────────────────────────────────────────────┤
│ Identity: ActaId                                            │
│                                                             │
│ Value Objects:                                              │
│   - NumeroActa (correlativo automático)                     │
│                                                             │
│ Entities:                                                   │
│   - Asistente[] (socioId, firma?)                           │
│   - Acuerdo[] (descripcion, votacion?)                      │
│                                                             │
│ Properties:                                                 │
│   - tipoReunion: TipoReunion                                │
│   - fecha: Date                                             │
│   - lugar: string                                           │
│   - horaInicio: Time                                        │
│   - horaFin: Time                                           │
│   - ordenDelDia: string[]                                   │
│   - estado: EstadoActa                                      │
│   - documentoId: DocumentoId? (PDF generado)                │
│   - firmadaPor: UsuarioId? (secretario)                     │
│   - vistobuenoPor: UsuarioId? (presidente)                  │
│                                                             │
│ Invariants:                                                 │
│   - Número correlativo único                                │
│   - Quórum mínimo según tipo reunión                        │
└─────────────────────────────────────────────────────────────┘
```

### 7.3 Domain Events

BC-Documentos emite eventos relacionados con la generación y gestión de documentos:
| Evento | Trigger | Payload | Consumidores |
|--------|---------|---------|--------------|
| `InformeAsambleaGenerado` | Informe de Asamblea General generado | informeId, ejercicioId, tipoInforme, fechaGeneracion, generadoPor | BC-Documentos (almacenar en repositorio) |

**Notas:**
- `InformeAsambleaGenerado`: Genera informe de Asamblea General con datos económicos y de membresía del ejercicio

### 7.4 Trazabilidad RF

| RF | Elemento de Dominio |
|----|---------------------|
| N7RF01-03 | Documento (Aggregate), MetadatosDocumento |
| N7RF04-05 | ArchivoFisico, previsualización |
| N7RF06-07 | Búsqueda por metadatos, Categoria.rolesAcceso |
| N7RF08 | Límite almacenamiento (configuración tenant) |
| N7RF09-12 | Versionado, OCR, búsqueda full-text (avanzado) |
| N6RF17-23 | Acta (Aggregate) - movido desde BC-Comunicacion |

---

## 7 bis. Extensión Transversal: Cumplimiento Normativo

**Nota:** Los requisitos N11 (Cumplimiento Normativo) se implementan como Application Services transversales que operan sobre los BCs existentes. Sin embargo, se documenta aquí un Aggregate específico para gestión de alertas legales que podría implementarse como extensión futura.

### 7 bis.1 Aggregate: AlertaLegal

```
┌─────────────────────────────────────────────────────────────┐
│ ALERTA_LEGAL (Aggregate Root)                               │
├─────────────────────────────────────────────────────────────┤
│ Identity: AlertaLegalId                                     │
│                                                             │
│ Value Objects:                                              │
│   - TipoNormativa (enum: RGPD, Ley_Asociaciones, Laboral)  │
│   - NivelCriticidad (enum: info, advertencia, critica)      │
│   - PlazoLimite (fechaLimite, diasAviso)                    │
│                                                             │
│ Properties:                                                 │
│   - titulo: string                                          │
│   - descripcion: string                                     │
│   - normativaAfectada: TipoNormativa                        │
│   - nivelCriticidad: NivelCriticidad                        │
│   - fechaDeteccion: Date                                    │
│   - fechaLimite: Date? (plazo legal si aplica)              │
│   - estado: EstadoAlerta (pendiente, en_revision, resuelta) │
│   - accionRequerida: string                                 │
│   - responsable: UsuarioId? (asignado para resolución)      │
│   - fechaResolucion: Date?                                  │
│   - observaciones: string?                                  │
│                                                             │
│ Invariants:                                                 │
│   - FechaLimite > FechaDeteccion (si definida)              │
│   - Estado resuelto requiere fechaResolucion definida       │
│   - Alertas críticas requieren responsable asignado         │
└─────────────────────────────────────────────────────────────┘
```

**Comportamientos:**
- `generarAlerta(tipo, descripcion, criticidad)` → crea alerta
- `asignarResponsable(usuario)` → delega resolución
- `marcarComoResuelta(observaciones)` → cierra alerta
- `escalar()` → aumenta criticidad si plazo vencido

**Trazabilidad RF:**
| RF | Elemento de Dominio |
|----|---------------------|
| N11RF01-17 | AlertaLegal (Aggregate) - Extensión futura |

**Ubicación propuesta:** BC-Identidad (como parte de auditoría) o módulo transversal dedicado.

---

## 8. BC-Identidad: Acceso y Autorización

### 8.1 Descripción

Gestiona la autenticación de usuarios, autorización basada en roles y la estructura multi-tenant. Es un BC genérico/de soporte que todos los demás BCs utilizan.

### 8.2 Aggregates

#### 8.2.1 Aggregate: Usuario

```
┌─────────────────────────────────────────────────────────────┐
│ USUARIO (Aggregate Root)                                    │
├─────────────────────────────────────────────────────────────┤
│ Identity: UsuarioId                                         │
│                                                             │
│ Value Objects:                                              │
│   - Credenciales (email, passwordHash)                      │
│   - TokenRecuperacion (valor, expiracion)                   │
│                                                             │
│ Properties:                                                 │
│   - nombre: string                                          │
│   - estado: EstadoUsuario                                   │
│   - fechaCreacion: DateTime                                 │
│   - ultimoAcceso: DateTime?                                 │
│   - intentosFallidos: int                                   │
│   - bloqueadoHasta: DateTime?                               │
│                                                             │
│ Invariants:                                                 │
│   - Email único global (cross-tenant)                       │
│   - Password cumple política complejidad                    │
└─────────────────────────────────────────────────────────────┘
```

#### 8.2.2 Aggregate: Tenant

```
┌─────────────────────────────────────────────────────────────┐
│ TENANT (Aggregate Root)                                     │
├─────────────────────────────────────────────────────────────┤
│ Identity: TenantId                                          │
│                                                             │
│ Value Objects:                                              │
│   - ConfiguracionTenant (limites, features habilitadas)     │
│   - DatosEntidad (nombre, CIF, direccion, tipo)             │
│                                                             │
│ Properties:                                                 │
│   - slug: string (identificador URL-friendly)               │
│   - plan: PlanSuscripcion                                   │
│   - estado: EstadoTenant                                    │
│   - fechaAlta: Date                                         │
│   - baseDatosId: string (conexión específica - RNF-004)     │
│                                                             │
│ Invariants:                                                 │
│   - Slug único global                                       │
│   - CIF válido y único                                      │
└─────────────────────────────────────────────────────────────┘
```

#### 8.2.3 Aggregate: Membresia (Usuario-Tenant)

```
┌─────────────────────────────────────────────────────────────┐
│ MEMBRESIA_TENANT (Aggregate Root)                           │
├─────────────────────────────────────────────────────────────┤
│ Identity: MembresiaId                                       │
│                                                             │
│ Properties:                                                 │
│   - usuarioId: UsuarioId                                    │
│   - tenantId: TenantId                                      │
│   - rolId: RolId                                            │
│   - socioId: SocioId? (vínculo con ficha de socio)          │
│   - fechaAsignacion: Date                                   │
│   - asignadoPor: UsuarioId                                  │
│   - activo: boolean                                         │
│                                                             │
│ Invariants:                                                 │
│   - Un usuario puede tener múltiples membresías (tenants)   │
│   - Solo una membresía activa por usuario-tenant            │
└─────────────────────────────────────────────────────────────┘
```

#### 8.2.4 Aggregate: Rol

```
┌─────────────────────────────────────────────────────────────┐
│ ROL (Aggregate Root)                                        │
├─────────────────────────────────────────────────────────────┤
│ Identity: RolId                                             │
│                                                             │
│ Properties:                                                 │
│   - codigo: string                                          │
│   - nombre: string                                          │
│   - descripcion: string                                     │
│   - permisos: Permiso[]                                     │
│   - esSistema: boolean (roles predefinidos)                 │
│   - tenantId: TenantId? (null = global)                     │
│                                                             │
│ Invariants:                                                 │
│   - Roles sistema no modificables                           │
│   - Código único dentro del tenant (o global)               │
└─────────────────────────────────────────────────────────────┘
```

### 8.3 Roles Predefinidos (Sistema)

| Código | Nombre | Permisos Principales |
|--------|--------|---------------------|
| `PRESIDENTE` | Presidente/Hermano Mayor | Todo + aprobaciones críticas |
| `SECRETARIO` | Secretario | Socios, actas, documentación |
| `TESORERO` | Tesorero | Economía, cuotas, remesas |
| `VOCAL` | Vocal | Según área asignada |
| `SOCIO` | Socio | Solo lectura propia vía portal |

### 8.4 Domain Events

| Evento | Trigger | Payload |
|--------|---------|---------|
| `UsuarioCreado` | Registro | usuarioId, email |
| `UsuarioAutenticado` | Login exitoso | usuarioId, tenantId, email, rol, ipAddress, userAgent, timestamp |
| `AutenticacionFallida` | Login fallido | email, intentos, ip |
| `TenantProvisionado` | Provisión completa de tenant | tenantId, nombreColectividad, tipoColectividad, adminUserId, adminEmail, cif |

### 8.5 Trazabilidad RF

| RF | Elemento de Dominio |
|----|---------------------|
| N2RF01 | Tenant (Aggregate), aislamiento por BD |
| N2RF02 | Usuario con múltiples MembresiaTenant |
| N2RF03 | ConfiguracionTenant |
| N2RF04 | Rol predefinidos (sistema) |
| N2RF05 | Rol personalizados (tenantId != null) |
| N2RF06 | Domain Events de auditoría |
| N2RF07 | RolAsignado event, histórico |
| N2RF08 | Validación edad en MembresiaTenant para cargos |

---

## 8.5 Domain Events Transversales

Eventos emitidos por Application Services que operan sobre múltiples BCs (requisitos N8-N11):

| Evento | Trigger | Payload | Consumidores |
|--------|---------|---------|--------------|
| `DashboardVisualized` | Usuario visualiza dashboard | usuarioId, tenantId, timestamp, seccionesVistas | - (analytics interno) |
| `ExportacionSolicitada` | Usuario solicita exportación de datos | exportacionId, usuarioId, tipoEntidad, formato, filtros | Application Service Export |
| `ExportacionFallida` | Exportación falla por error | exportacionId, tipoError, mensaje, timestamp | BC-Comunicacion (notificar usuario) |
| `CommunicationHistoryExported` | Histórico de comunicaciones exportado | exportacionId, fechaInicio, fechaFin, formato, totalRegistros | - |
| `EconomicReportGenerated` | Informe económico generado | informeId, tipo, ejercicioId, fechaGeneracion, generadoPor | BC-Documentos (almacenar) |
| `Modelo182Generated` | Modelo 182 (AEAT) generado | modeloId, ejercicioId, totalSocios, totalImporte, fechaGeneracion | BC-Documentos (almacenar) |
| `PlanAmpliado` | Plan de suscripción ampliado | tenantId, planAnterior, planNuevo, limites, fechaCambio | BC-Identidad (actualizar tenant config) |
| `TurnoCajaAbierto` | Turno de caja abierto (peñas festeras) | turnoId, usuarioId, fechaApertura, saldoInicial | BC-Tesoreria (registrar apertura) |
| `TurnoCajaCerrado` | Turno de caja cerrado | turnoId, usuarioId, fechaCierre, saldoFinal, diferencia | BC-Tesoreria (conciliar caja) |

**Notas:**
- Estos eventos son emitidos por Application Services que implementan requisitos transversales (N8: Import/Export, N9: Reporting, N10: Portal Socio, N11: Cumplimiento)
- No pertenecen a un BC específico pero pueden ser consumidos por múltiples BCs
- `TurnoCajaAbierto` y `TurnoCajaCerrado` son features específicas de peñas festeras (N12RF06-10) pero se documentan aquí por su naturaleza transversal

---

## 9. Context Map

### 9.1 Diagrama de Relaciones

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              CONTEXT MAP                                   │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│    ┌──────────────┐         ┌──────────────┐         ┌──────────────┐      │
│    │              │         │              │         │              │      │
│    │ BC-Identidad │◄───ACL──┤ BC-Membresia │───PUB───► BC-Tesoreria │      │
│    │   (Generic)  │         │    (Core)    │         │    (Core)    │      │
│    │              │         │              │         │              │      │
│    └──────┬───────┘         └──────┬───────┘         └──────┬───────┘      │
│           │                        │                        │              │
│           │                        │ PUB                    │ PUB          │
│           │                        ▼                        ▼              │
│           │                 ┌──────────────┐         ┌──────────────┐      │
│           │                 │              │         │              │      │
│           └────ACL─────────►│  BC-Eventos  │◄──SUB───┤BC-Comunicac. │      │
│                             │    (Core)    │         │ (Supporting) │      │
│                             │              │         │              │      │
│                             └──────┬───────┘         └──────┬───────┘      │
│                                    │                        │              │
│                                    │ PUB                    │              │
│                                    ▼                        │              │
│                             ┌──────────────┐                │              │
│                             │              │◄───────────────┘              │
│                             │BC-Documentos │                               │
│                             │ (Supporting) │                               │
│                             │              │                               │
│                             └──────────────┘                               │
│                                                                            │
│  Leyenda:                                                                  │
│    ───ACL───► : Anticorruption Layer (traduce conceptos)                   │
│    ───PUB───► : Publisher (emite Domain Events)                            │
│    ◄──SUB─── : Subscriber (consume Domain Events)                          │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Relaciones Detalladas

| Upstream | Downstream | Tipo | Descripción |
|----------|------------|------|-------------|
| BC-Identidad | BC-Membresia | ACL | Membresia traduce Usuario a contexto de socio |
| BC-Identidad | BC-Eventos | ACL | Eventos usa UsuarioId pero no conoce detalles |
| BC-Membresia | BC-Tesoreria | Pub/Sub | Eventos de socio disparan generación de cargos |
| BC-Membresia | BC-Comunicacion | Pub/Sub | Eventos de socio disparan notificaciones |
| BC-Tesoreria | BC-Comunicacion | Pub/Sub | Eventos de pago/morosidad disparan avisos |
| BC-Tesoreria | BC-Membresia | Pub/Sub | Morosidad grave cambia estado del socio |
| BC-Eventos | BC-Tesoreria | Pub/Sub | Inscripción con precio genera cargo |
| BC-Eventos | BC-Comunicacion | Pub/Sub | Eventos de actividad disparan notificaciones |
| BC-Eventos | BC-Documentos | Pub/Sub | Check-in puede adjuntar justificantes |
| BC-Comunicacion | BC-Documentos | Customer/Supplier | Comunicación solicita adjuntar documentos |

### 9.3 Shared Kernel

Los siguientes elementos son compartidos entre BCs (librería común):

```
shared-kernel/
├── value-objects/
│   ├── SocioId
│   ├── TenantId
│   ├── UsuarioId
│   ├── EjercicioId
│   ├── Dinero
│   └── Email
├── events/
│   └── DomainEvent (base)
└── interfaces/
    └── AggregateRoot
```

---

## 10. Consideraciones Multi-Tenant

### 10.1 Estrategia de Aislamiento (RNF-004)

Según RNF-004, el aislamiento multi-tenant se implementa por **base de datos separada**:

```
┌─────────────────────────────────────────────────────────────┐
│                    ARQUITECTURA MULTI-TENANT                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐      ┌─────────────────────────────────┐  │
│   │             │      │        TENANT RESOLUTION        │  │
│   │   Usuario   │─────►│  1. Extraer tenant de request   │  │
│   │   Request   │      │  2. Obtener conexión BD tenant  │  │
│   │             │      │  3. Inyectar en contexto        │  │
│   └─────────────┘      └─────────────────────────────────┘  │
│                                    │                        │
│                                    ▼                        │
│   ┌────────────────────────────────────────────────────┐    │
│   │                   BASES DE DATOS                   │    │
│   ├────────────────────────────────────────────────────┤    │
│   │                                                    │    │
│   │  ┌───────────┐  ┌──────────┐  ┌──────────┐         │    │
│   │  │ DB-Main   │  │DB-Tenant1│  │DB-Tenant2│  ...    │    │
│   │  │(Identidad)│  │(user_t1) │  │(user_t2) │         │    │
│   │  └───────────┘  └──────────┘  └──────────┘         │    │
│   │                                                    │    │
│   │  - Usuarios     - Socios      - Socios             │    │
│   │  - Tenants      - Cuotas      - Cuotas             │    │
│   │  - Membresías   - Eventos     - Eventos            │    │
│   │                 - Docs        - Docs               │    │
│   │                                                    │    │
│   └────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 10.2 Ubicación de Datos por BC

| BC | Base de Datos | Justificación |
|----|---------------|---------------|
| BC-Identidad | DB-Main | Cross-tenant: usuarios, tenants, membresías |
| BC-Membresia | DB-TenantX | Datos aislados por entidad |
| BC-Tesoreria | DB-TenantX | Datos aislados por entidad |
| BC-Eventos | DB-TenantX | Datos aislados por entidad |
| BC-Comunicacion | DB-TenantX | Datos aislados por entidad |
| BC-Documentos | DB-TenantX | Datos aislados + almacenamiento ficheros |

### 10.3 Referencias Cross-Tenant

Las únicas referencias cross-tenant permitidas son:

1. **UsuarioId** → referenciado desde membresías en DB-TenantX
2. **TenantId** → almacenado en contexto de ejecución, no en datos

Nunca se almacenan datos de un tenant en la BD de otro.

---

## 11. Glosario del Dominio (Ubiquitous Language)

### Términos Generales

| Término | Definición | BC |
|---------|------------|-----|
| **Socio** | Persona física que pertenece a la colectividad | Membresia |
| **Hermano** | Sinónimo de socio en cofradías | Membresia |
| **Peñista** | Sinónimo de socio en peñas | Membresia |
| **Aspirante** | Persona en proceso de alta, aún no es socio pleno | Membresia |
| **Ejercicio** | Período temporal de gestión (año natural o temporada) | Membresia |
| **Antigüedad** | Tiempo acumulado como socio, determina derechos | Membresia |
| **Carnet** | Documento de identificación del socio | Membresia |

### Términos Económicos

| Término | Definición | BC |
|---------|------------|-----|
| **Cuota** | Importe periódico que debe abonar el socio | Tesoreria |
| **Cargo** | Deuda generada al socio por cualquier concepto | Tesoreria |
| **Remesa** | Conjunto de cobros enviados al banco | Tesoreria |
| **Mandato SEPA** | Autorización firmada para domiciliar cobros | Tesoreria |
| **Morosidad** | Situación de impago de cuotas | Tesoreria |

### Términos de Actividades

| Término | Definición | BC |
|---------|------------|-----|
| **Evento** | Actividad organizada por la colectividad | Eventos |
| **Inscripción** | Registro de participación en un evento | Eventos |
| **Aforo** | Capacidad máxima de asistentes a un evento | Eventos |
| **Check-in** | Registro de asistencia efectiva | Eventos |

### Términos Organizativos

| Término | Definición | BC |
|---------|------------|-----|
| **Junta Directiva** | Órgano de gobierno de la colectividad | Identidad |
| **Asamblea** | Reunión de todos los socios con derecho a voto | Documentos |
| **Acta** | Documento oficial que recoge acuerdos de reunión | Documentos |
| **Tenant** | Cada colectividad/entidad en el sistema multi-tenant | Identidad |

---

## Trazabilidad

### Matriz BC → Secciones RF

| BC | Secciones RF | Total RFs |
|----|--------------|-----------|
| BC-Identidad | N2 | 8 |
| BC-Membresia | N3 | 34 |
| BC-Tesoreria | N4 | 35 |
| BC-Eventos | N5 | 30 |
| BC-Comunicacion | N6 | 23 |
| BC-Documentos | N7 | 12 |
| Transversal | N8, N9, N10, N11 | 57 |
| Extensiones | N12, N13 | 19 |
| **Total** | | **218** |

### Matriz BC → RNF Relevantes

| BC | RNFs Clave |
|----|------------|
| BC-Identidad | RNF-001 a RNF-003 (autenticación), RNF-004 (multi-tenant), RNF-013 (mínimo privilegio) |
| BC-Membresia | RNF-006 (cifrado datos), RNF-025-030 (RGPD), RNF-035 (datos religiosos) |
| BC-Tesoreria | RNF-006 (cifrado IBAN), RNF-018 (rendimiento masivas), RNF-042 (gestión errores) |
| BC-Eventos | RNF-006 (aforo), RNF-015-016 (tiempos respuesta) |
| BC-Comunicacion | RNF-010 (rate limiting), RNF-034 (privacidad diseño) |
| BC-Documentos | RNF-009 (ficheros seguros), RNF-022 (carga ficheros), RNF-032 (retención) |

---

## 9.5 Event Nomenclature: Business vs Internal Events

### Propósito

Esta sección clarifica la distinción entre **Eventos de Negocio** (Business Events) para integración cross-BC y **Eventos Internos** (Internal Events) del ciclo de vida de Aggregates, para evitar confusión durante la documentación de UCs y la implementación del sistema.

### Eventos de Negocio (Business Events) - Integración Cross-BC

**Características:**
- Publicados para consumo por otros Bounded Contexts
- Representan **operaciones de negocio completadas** con contexto completo
- Documentados en las tablas de eventos de los UCs (sección "Eventos Publicados")
- Nomenclatura: `<Entidad><AcciónCompletaConContexto>` (ej. `RemesaSepaGenerada`, `TenantProvisionado`)

**Propósito:**
- Integración y orquestación cross-BC
- Trigger de workflows en otros contextos
- Notificaciones visibles al usuario (vía BC-Comunicacion)

**Ejemplo:** `TenantProvisionado`
- Trigger: Tras completar el provisionamiento completo del tenant (UC-001 paso 6)
- Payload: Información completa (tenantId, nombreColectividad, tipoColectividad, adminUserId, cif)
- Consumidores: BC-Comunicacion (email bienvenida), Sistema de Monitorización

### Eventos Internos (Internal Events) - Ciclo de Vida de Aggregates

**Características:**
- Emitidos durante cambios de estado internos del Aggregate
- Representan **operaciones técnicas de grano fino**
- NO documentados en tablas de eventos de UCs (solo en KB-005)
- Nomenclatura: `<Entidad><AcciónSimple>` (ej. `RemesaGenerada`, `TenantCreado`)

**Propósito:**
- Auditoría y trazabilidad interna
- Implementación futura de event sourcing
- Tracking del ciclo de vida del Aggregate

**Ejemplo:** `TenantCreado`
- Trigger: Cuando se crea por primera vez el Aggregate Tenant (UC-001 paso 2)
- Payload: Información técnica (tenantId, datos básicos)
- Consumidores: Ninguno (solo uso interno)
- Evento de Negocio: `TenantProvisionado` se emite posteriormente cuando el provisionamiento se completa

### Mapeo: Eventos Internos → Eventos de Negocio

| Evento Interno (KB-005) | Evento de Negocio (UCs) | Contexto |
|-------------------------|-------------------------|----------|
| `TenantCreado` | `TenantProvisionado` | UC-001: Provisionamiento de tenant |
| `RemesaGenerada` | `RemesaSepaGenerada` | UC-023: Generación de remesa SEPA |
| `RemesaProcesada` | `RemesaSepaEnviada` | UC-023: Procesamiento de remesa SEPA |
| `MandatoCreado` | `MandatoSepaRegistrado` | UC-023: Registro de mandato SEPA |
| `MandatoCaducado` | (Implícito en UC-023 FA-3) | UC-023: Caducidad de mandato tras 36 meses |
| `SocioSuspendidoPorImpago` | `EstadoSocioCambiado` | UC-007/UC-022: Evento genérico de cambio de estado |

### Guías de Implementación

**Al documentar UCs:**
1. Incluir SOLO Eventos de Negocio en las tablas "Eventos Publicados"
2. Los Eventos Internos pueden mencionarse en las descripciones de flujo pero no en tablas de eventos
3. Si existen ambos tipos de eventos, emitir primero el Interno, luego el de Negocio

**Al implementar Event-Driven Architecture:**
1. Publicar Eventos de Negocio en message broker (ej. RabbitMQ, Kafka)
2. Almacenar Eventos Internos en event store para auditoría (opcional)
3. Los consumidores externos NO deben depender NUNCA de Eventos Internos

---

## Changelog

- v1.5 (Feb 2026): Clasificación de eventos internos vs eventos de negocio
  - **Nueva sección 9.5:** Event Nomenclature - Business vs Internal Events
    - Definición formal de eventos de negocio (cross-BC integration)
    - Definición formal de eventos internos (Aggregate lifecycle)
    - Tabla de mapeo: eventos internos → eventos de negocio
    - Guías de implementación para UCs y Event-Driven Architecture
  - **Marcados 6 eventos internos** en tablas de Domain Events:
    - **BC-Identidad (1):** `TenantCreado` [Interno] → evento negocio: `TenantProvisionado`
    - **BC-Tesoreria (5):** 
      - `SocioSuspendidoPorImpago` [Interno] → evento negocio: `EstadoSocioCambiado`
      - `RemesaGenerada` [Interno] → evento negocio: `RemesaSepaGenerada`
      - `RemesaProcesada` [Interno] → evento negocio: `RemesaSepaEnviada`
      - `MandatoCreado` [Interno] → evento negocio: `MandatoSepaRegistrado`
      - `MandatoCaducado` [Interno] → evento negocio: implícito en UC-023 FA-3
  - **Propósito:** Clarificar qué eventos deben documentarse en tablas de UCs (solo eventos de negocio)
  - **Total eventos:** 87 (81 eventos de negocio + 6 eventos internos)
  - **Script actualizado:** `inventario_eventos.py` v2.1 detecta marcador `[Interno]`

- v1.4 (Feb 2026): Corrección de nomenclatura de eventos para alineación con UCs
  - **BC-Membresia:** Corregidos 2 nombres de eventos para consistencia con UC-012 y UC-015
    - `CarnetEmitido` → `CarnetGenerado` (alineado con UC-015, línea 2805)
    - `SolicitudAltaCreada` → `SolicitudAltaIniciada` (alineado con UC-012, línea 2121)
  - Justificación: Nomenclatura de UCs es más descriptiva del proceso
  - `CarnetGenerado` refleja mejor el proceso técnico (generación de QR/PDF)
  - `SolicitudAltaIniciada` describe mejor el workflow de cofradías (inicio de proceso de 7 fases)
  - Total eventos: 85 (sin cambios en cantidad, solo nomenclatura)

- v1.3 (Feb 2026): Limpieza de eventos duplicados
  - **Eliminados 8 eventos duplicados** (manteniendo cobertura 100%)
  - **BC-Tesoreria:** Consolidadas tablas de eventos (eliminados 7 duplicados)
    - Se mantuvo tabla original, eliminada segunda tabla redundante con eventos repetidos
    - Duplicados eliminados: `CargoGenerado`, `PagoRegistrado`, `PagoDevuelto`, `MorosidadDetectada`, `PlanCuotaCreado`, `PlanCuotaModificado`, `PlanCuotaVinculadoATipoSocio`
  - **BC-Eventos:** Eliminado `InscripcionRealizada` duplicado (mantenida definición completa)
  - **BC-Comunicacion:** Conservada tabla "Eventos Consumidos" (NO es duplicación)
    - Tabla documenta acciones al CONSUMIR eventos de otros BCs (diferente propósito)
  - Total Domain Events únicos: **85 eventos** (antes 93 con duplicados)
  - Scripts de análisis: `detectar_duplicados_kb005.py`, `eliminar_duplicados_kb005.py`
  - Verificado: **Cobertura 100%** mantenida (60/60 eventos de UCs documentados)

- v1.2 (Feb 2026): Sincronización completa de Domain Events con Casos de Uso
  - **Añadidos 46 eventos faltantes** (100% cobertura respecto a UC v2.5)
  - **BC-Membresia:** 3 eventos añadidos
    - `SolicitudAltaAprobada`, `CarnetGenerado`, `SocioInscritoEnCuadrilla`
  - **BC-Tesoreria:** 25 eventos añadidos
    - Incluyendo `PagoOnlineRealizado`, `EnlacePagoGenerado`, `TurnoCajaAbierto`, `DeudaSaldada`, etc.
  - **BC-Eventos:** 4 eventos añadidos
    - `InscripcionRealizada`, `InscripcionCancelada`, `PlazasAgotadas`, `AsistenciaRegistrada`
  - **BC-Identidad:** 8 eventos añadidos/actualizados
    - Incluye eventos de autenticación, roles y permisos
  - **BC-Comunicacion:** 6 eventos añadidos
    - `NotificacionBienvenidaEnviada`, `RecordatorioPagoEnviado`, `AvisoMorosidadEnviado`, etc.
  - **BC-Documentos:** Nueva sección 7.3 Domain Events
    - `InformeAsambleaGenerado`
  - **Nueva sección 8.5:** Domain Events Transversales (9 eventos)
    - Eventos de Application Services cross-BC: exports, reporting, analytics, caja
  - Total Domain Events documentados: 146 (de 100 a 146)
  - Verificado con `inventario_eventos.py`: **Cobertura 100%**

- v1.1 (Feb 2026): Actualización con Aggregates críticos
  - **BC-Membresia:** Añadidos 2 Aggregates
    - ListaEspera (3.2.6): Gestión de cola de aspirantes
    - ExpedienteDisciplinario (3.2.7): Registro de infracciones y sanciones
  - **BC-Tesoreria:** Añadidos 4 Aggregates
    - EnlacePago (4.2.10): Enlaces temporales para pagos online
    - TurnoCaja (4.2.11): Gestión de turnos de caja (específico peñas)
    - CategoriaContable (4.2.12): Clasificación de movimientos contables
    - EjercicioContable (4.2.13): Periodos fiscales y cierre contable
  - **BC-Eventos:** Añadidos 3 Aggregates
    - EventoComida (5.2.5): Gestión de comidas y menús (específico peñas)
    - Cuadrilla (5.2.6): Agrupaciones de socios para actividades
    - Partido (5.2.7): Encuentros deportivos (específico clubes)
  - **Extensión Transversal:** Añadida sección 7 bis
    - AlertaLegal: Gestión de alertas de cumplimiento normativo (extensión futura)
  - Actualizadas tablas de Value Objects en BC-Membresia, BC-Tesoreria y BC-Eventos
  - Actualizadas referencias de trazabilidad RF para los nuevos Aggregates
  - Total Aggregates documentados: 29 (de 19 iniciales a 29)

- v1.0 (Feb 2026): Versión inicial
  - 6 Bounded Contexts identificados (3 Core + 3 Supporting)
  - Aggregates, Entities, Value Objects para cada BC
  - Domain Events con productores y consumidores
  - Context Map con relaciones
  - Consideraciones multi-tenant según RNF-004
  - Glosario del dominio (Ubiquitous Language)