# Documentación Técnica

## Índice

1. [Introducción](#1-introducción)
2. [Planteamiento del Problema](#2-planteamiento-del-problema)
   - [2.1 Descripción de la Problemática](#21-descripción-de-la-problemática)
   - [2.2 Levantamiento de Requerimientos](#22-levantamiento-de-requerimientos)
3. [Diseño Conceptual](#3-diseño-conceptual)
   - [3.1 Identificación de Entidades](#31-identificación-de-entidades)
   - [3.2 Relaciones y Cardinalidades](#32-relaciones-y-cardinalidades)
   - [3.3 Jerarquía ISA](#33-jerarquía-isa)
   - [3.4 Entidades Débiles](#34-entidades-débiles)

   # 🐾 Veterinaria Patitas Sanas — Documentación de Base de Datos

> Sistema de Gestión Clínica Veterinaria · Versión 1.0  
> Única sucursal: Colonia Emiliano Zapata, Los Reyes La Paz

---

## 1. Introducción y Planteamiento del Problema

Este documento presenta la documentación técnica del sistema de gestión veterinaria patitas sanas, desarrollado como proyecto final para la materia de Bases de Datos por el equipo del Grupo XX.

### 1.1 Descripción de la Problemática

La **Veterinaria Patitas Sanas**, dirigida por el **Dr. Alejandro Ortega López**, operaba hasta la implementación de este sistema bajo un modelo completamente manual: los registros de pacientes se llevaban en libretas físicas y hojas de cálculo sueltas sin ningún esquema formal. Este modelo generaba tres consecuencias críticas:

- **Pérdida de historiales médicos**: Sin un repositorio centralizado, la información clínica de las mascotas (diagnósticos, tratamientos, vacunas) se extraviaba o quedaba incompleta.
- **Desorganización de la agenda**: La programación de citas se realizaba sin trazabilidad, provocando empalmes, citas duplicadas y falta de seguimiento del estado de cada atención.
- **Riesgo en situaciones de emergencia**: La imposibilidad de localizar rápidamente los datos de contacto de un tutor ante una urgencia clínica representaba un riesgo real para la vida de las mascotas.

Este proyecto resuelve la problemática mediante un **esquema relacional normalizado** que digitaliza y estructura la operación completa de la clínica: registro de tutores y mascotas, agenda de citas, historial clínico, control de vacunación/desparasitación y registro de pagos.

---

### 1.2 Levantamiento de Requerimientos

Los siguientes requerimientos fueron extraídos directamente de la entrevista con el **Dr. Alejandro Ortega López**:

**Requerimientos Generales y de Acceso**

- El sistema debe soportar al menos dos **perfiles de usuario**: `veterinario` y `recepcionista`, cada uno con responsabilidades diferenciadas dentro del flujo de atención.
- El sistema está diseñado para operar en una **única sucursal**.
- Debe existir un catálogo de **servicios** ofrecidos (consulta, cirugía, rayos X, ultrasonido, cremación, entre otros).

**Módulo de Tutores (Clientes)**

- Registrar a cada responsable como **Tutor**, almacenando: nombre completo, dirección, correo electrónico, número(s) de teléfono y fecha de registro.
- Un tutor puede tener **una o varias mascotas** registradas; cada mascota pertenece a un único tutor.

**Módulo de Mascotas (Pacientes)**

- Generar un **ID único** (`id_mascota`) por cada mascota registrada.
- Registrar: nombre, especie, raza, sexo, edad y un identificador de carnet de salud (`id_uk_carnet`).
- Restringir las **especies válidas** a: `'reptil'`, `'felino'`, `'canino'` y `'ave'`, excluyendo ganado y animales exóticos mayores (restricción `CHECK` en la columna `especie`).

**Módulo de Citas y Agenda**

- Registrar citas con: fecha, hora, motivo, monto y referencia al veterinario que atenderá.
- La tabla `agenda` vincula de forma ternaria a la `cita`, la `recepcionista` que la programa y el `tutor` que la solicita.
- Permitir atenciones **sin cita previa** (walk-in), así como clasificar el estado de cada atención.

**Módulo de Servicios / Historial Clínico**

- Cada servicio (`servicio`) está asociado a una cita específica (relación 1:1) y a una mascota.
- El tipo de servicio está restringido mediante `CHECK` a: `'consulta'`, `'cirugia'`, `'rayos x'`, `'ultrasonido'` y `'creamacion'`.
- Registrar el **motivo** de cada servicio.

**Módulo de Vacunación y Desparasitación**

- Guardar el historial de **vacunas** (`vacuna`) por mascota: tipo de vacuna, fecha de aplicación y fecha de próxima dosis.
- Guardar el historial de **desparasitaciones** (`desparasitacion`) por mascota: producto y fecha de aplicación.
- Estos registros permiten al sistema calcular y avisar sobre próximas dosis.

**Módulo de Pagos**

- Registrar el **monto** de cada cita/servicio con su fecha.
- Registrar el **método de pago** (efectivo, tarjeta, transferencia).
- El campo `id_uk_transaccion` en la tabla `cita` prevé la integración futura con un sistema de facturación formal.

---

## 2. Diseño Conceptual y Arquitectura de Datos

### 2.1 Identificación de Entidades

La siguiente tabla documenta **todas** las tablas declaradas en el script SQL y su rol dentro del dominio veterinario:

| Entidad (Tabla) | Descripción |
|---|---|
| **`empleado`** | Entidad padre (supertipo) de todo el personal. Almacena los atributos compartidos: `id_empleado`, `nombre`, `telefono` y `rfc`. Actúa como base de la jerarquía ISA. |
| **`veterinario`** | Especialización de `empleado`. Extiende al personal médico con `cedula_profesional` y `especialidad`. Su PK (`id_empleado`) es simultáneamente FK hacia `empleado`. |
| **`recepcionista`** | Especialización de `empleado`. Representa al personal administrativo. Incorpora `id_caja` como PK propia y mantiene `id_empleado` como FK hacia `empleado`. |
| **`tutor`** | Entidad independiente que representa al dueño o responsable legal de una o varias mascotas. Almacena datos de contacto: `nombre`, `direccion`, `correo_electronico`, `telefono` y `fecha_de_registo`. |
| **`mascota`** | Entidad central del sistema clínico. Representa al paciente. Almacena datos biográficos (`nombre`, `especie`, `sexo`, `edad`) y referencia su carnet de salud mediante `id_uk_carnet`. |
| **`tiene`** | Tabla asociativa (relación N:M) que vincula a `tutor` con `mascota`, formalizando la propiedad o tutela de uno o varios animales. |
| **`cita`** | Registra el encuentro programado entre un paciente y el veterinario. Contiene `fecha`, `hora`, `monto`, referencia al `veterinario` y un campo único `id_uk_transaccion` para pagos. |
| **`agenda`** | Tabla asociativa ternaria que une a `cita`, `recepcionista` y `tutor`. Materializa el acto administrativo de agendar una cita: quién la programa, para quién y cuándo. |
| **`servicio`** | Representa el acto médico o procedimiento realizado durante una cita. Tiene relación 1:1 con `cita` y N:1 con `mascota`. Restringe los tipos válidos mediante `CHECK`. |
| **`vacuna`** | Entidad débil por existencia. Registra cada vacuna aplicada a una mascota: tipo, `fecha_aplicacion` y `proxima_dosis` para control preventivo. Depende de `mascota`. |
| **`desparasitacion`** | Entidad débil por existencia. Registra cada desparasitación aplicada a una mascota con su `fecha_aplicacion`. Depende de `mascota`. |

---

### 2.2 Relaciones y Cardinalidades

Las siguientes relaciones se deducen **estrictamente** de las cláusulas `FOREIGN KEY` del script SQL:

| Relación | Tipo | Cardinalidad (Min, Max) | Descripción |
|---|---|---|---|
| **`empleado`** → **`veterinario`** | Especialización ISA (herencia de PK) | (0, 1) : (1, 1) | Un empleado puede especializarse como veterinario. El `id_empleado` en `veterinario` es PK y FK simultánea. |
| **`empleado`** → **`recepcionista`** | Especialización ISA (herencia de PK) | (0, 1) : (1, 1) | Un empleado puede especializarse como recepcionista. El `id_empleado` en `recepcionista` es FK con restricción `UNIQUE NOT NULL`. |
| **`tutor`** — **`mascota`** | N:M (vía `tiene`) | (0, N) : (1, 1) | Un tutor puede tener cero o muchas mascotas; una mascota pertenece a exactamente un tutor. La tabla `tiene` materializa esta relación. |
| **`veterinario`** → **`cita`** | 1:N | (1, 1) : (0, N) | Un veterinario atiende muchas citas; cada cita es asignada a exactamente un veterinario (`id_empleado` FK en `cita`). |
| **`cita`** → **`servicio`** | 1:1 | (1, 1) : (0, 1) | Cada cita genera a lo sumo un registro de servicio/acto médico. La columna `id_cita` en `servicio` es `UNIQUE`, garantizando cardinalidad máxima de 1. |
| **`mascota`** → **`servicio`** | 1:N | (1, 1) : (0, N) | Una mascota puede ser objeto de múltiples servicios a lo largo del tiempo. `id_uk_mascota` en `servicio` es FK hacia `mascota`. |
| **`cita`** + **`recepcionista`** + **`tutor`** → **`agenda`** | Relación ternaria N:M:M | (0, N) : (0, N) : (0, N) | La tabla `agenda` es una entidad asociativa ternaria. Su PK compuesta (`id_cita`, `id_empleado`, `id_cliente`) impide duplicados y vincula los tres participantes del acto de agendar. |
| **`mascota`** → **`vacuna`** | 1:N | (1, 1) : (0, N) | Una mascota acumula múltiples registros de vacunación. `id_mascota` en `vacuna` es FK; sin la mascota, la vacuna no tiene entidad propia. |
| **`mascota`** → **`desparasitacion`** | 1:N | (1, 1) : (0, N) | Una mascota puede tener múltiples desparasitaciones registradas. `id_mascota` en `desparasitacion` es FK; igual que `vacuna`, depende de la existencia de la mascota. |

---

### 2.3 Jerarquía ISA (Especialización / Generalización)

El personal de la clínica se modela mediante una **jerarquía de especialización** de tres niveles, implementada con el patrón de **tabla por subtipo**:

```
empleado  (supertipo)
   ├── veterinario  (subtipo A)
   └── recepcionista  (subtipo B)
```

**Implementación física:**

- La tabla **`empleado`** actúa como supertipo y concentra los atributos compartidos: `id_empleado` (PK), `nombre`, `telefono` y `rfc`.
- La tabla **`veterinario`** usa `id_empleado` como **PK y FK simultánea** hacia `empleado`. Añade los atributos diferenciadores: `cedula_profesional` y `especialidad`.
- La tabla **`recepcionista`** usa `id_empleado` como FK con restricción `UNIQUE NOT NULL`, y declara `id_caja` como su propia PK. Esto refleja que la recepcionista tiene una identidad adicional vinculada a su caja de cobro.

**Restricción `DEFERRABLE INITIALLY IMMEDIATE`:**

Todas las FK de la jerarquía se declaran con `DEFERRABLE INITIALLY IMMEDIATE`. Esto significa que la validación de integridad referencial es **inmediata por defecto** (se evalúa al finalizar cada sentencia DML), pero puede diferirse explícitamente dentro de una transacción usando `SET CONSTRAINTS DEFERRED`. Esta flexibilidad permite insertar primero el registro en `empleado` y luego en `veterinario` o `recepcionista` dentro de la misma transacción sin violar la FK.

**Tipo de especialización:**

- **Disjunta**: Un empleado no puede ser simultáneamente `veterinario` y `recepcionista` en el diseño actual, dado que cada subtipo ocupa un `id_empleado` único.
- **Parcial**: No todo `empleado` requiere tener una fila correspondiente en `veterinario` ni en `recepcionista`; podría existir un empleado genérico sin subtipo asignado (por ejemplo, durante el proceso de alta).

---

### 2.4 Entidades Débiles y Dependencias

En el modelo implementado se identifican las siguientes **entidades con dependencia de existencia** o **dependencia de identidad**:

**Dependencia de existencia — `vacuna` y `desparasitacion`**

Las tablas **`vacuna`** y **`desparasitacion`** son entidades débiles respecto a **`mascota`**:

- Ambas incluyen `id_mascota` como **FK no nula**, heredado de la tabla `mascota`.
- Sin la existencia previa de una fila en `mascota`, ningún registro de vacuna ni de desparasitación puede existir. Si se eliminara una mascota, los registros asociados perderían su referencia.
- A nivel físico, la dependencia se implementa mediante la cláusula `ALTER TABLE ... ADD FOREIGN KEY (id_mascota) REFERENCES mascota(id_mascota) DEFERRABLE INITIALLY IMMEDIATE`.
- Sus claves primarias (`id_vacuna`, `id_desparasitacion`) son generadas independientemente, pero la **existencia** del registro depende del ciclo de vida de la mascota.

**Dependencia de identidad — `veterinario` y `recepcionista`**

Las tablas **`veterinario`** y **`recepcionista`** son subtypes con **dependencia de identidad** respecto a **`empleado`**:

- `veterinario` no puede tener un `id_empleado` que no exista previamente en `empleado`; su propia identidad (PK) es heredada.
- `recepcionista` mantiene el mismo patrón: su `id_empleado` debe referenciar siempre a un `empleado` existente.
- La eliminación de un registro en `empleado` sin eliminar antes su correspondiente fila en `veterinario` o `recepcionista` violaría la integridad referencial.

**Dependencia funcional — `servicio` respecto a `cita`**

La tabla **`servicio`** no puede existir de forma autónoma: su columna `id_cita` es `UNIQUE` y FK hacia `cita`. Un servicio médico solo tiene sentido en el contexto de una cita registrada, lo que convierte a `cita` en la entidad padre del acto clínico y a `servicio` en su extensión dependiente.

**Tabla asociativa pura — `tiene` y `agenda`**

Las tablas **`tiene`** y **`agenda`** no almacenan atributos propios más allá de sus claves foráneas. Su rol es exclusivamente relacionar entidades:

- **`tiene`**: PK compuesta (`id_cliente`, `id_mascota`), resuelve la relación N:M entre `tutor` y `mascota`.
- **`agenda`**: PK compuesta ternaria (`id_cita`, `id_empleado`, `id_cliente`), resuelve la relación ternaria entre `cita`, `recepcionista` y `tutor`.

### 3.5 Diagrama EER

<img src="https://raw.github.com/ctlu-l/pagina-web/blob/main/EER.drawio.png" alt="Diagrama EER del sistema Los Consentidos" loading="lazy">
