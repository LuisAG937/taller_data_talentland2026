# 📝 README — Solicitudes de Beca e Instrumento de Captura

**Sistema SYSAC · Módulo de Solicitudes y Cuestionario**

> Este documento describe en detalle la estructura, lógica y reglas de coherencia del cuestionario de solicitud de beca y los catálogos que definen las reglas de asignación. Los participantes del taller usan estos insumos para construir la evaluación completa de becas.

---

## 📋 Descripción General

El módulo de solicitudes de beca de SYSAC tiene dos componentes que se complementan pero cumplen funciones distintas:

**El primero** son las tablas de configuración del sistema de becas (`CATBEC`, `REGBEC`), que representan la lógica institucional: qué tipos de beca existen y bajo qué reglas se asignan. Son datos estructurados, con relaciones formales y criterios escalonados. Es lo que vería un administrador de becas al revisar las políticas vigentes.

**El segundo** es el cuestionario de solicitud (`CUESTIONARIO_BECAS` + `CATALOGO_RESPUESTAS`), que representa la perspectiva del estudiante: sus respuestas a 38 preguntas organizadas en 7 bloques temáticos, con campos condicionales que solo se activan según respuestas previas. Es lo que vería un analista de datos al recibir un export del formulario en línea.

La gracia está en que estos componentes, combinados con la base institucional (`HISTACA`, `CALFIN`, `PAGFIN`, `DOMICOM`, etc.), permiten a los participantes del taller construir tres artefactos clave: las solicitudes estructuradas (SOLBEC), las distancias para beca de transporte (DISBEC) y las evaluaciones de beca (EVABEC). Cuando cruzas todas estas fuentes, las contradicciones y los patrones ocultos empiezan a saltar.

---

## 🏛️ Tablas de Configuración del Sistema de Becas

### `CATBEC.csv` — Catálogo de Tipos de Beca

El diccionario maestro. Define las cuatro modalidades de beca con sus reglas generales, restricciones y porcentajes de cobertura. Sin esta tabla, no hay referencia contra la cual evaluar las solicitudes.

| Campo | Tipo | Descripción |
|---|---|---|
| `CVE_BECA` | CHAR(6) | Clave única del tipo de beca |
| `NOMBRE_BECA` | VARCHAR | Nombre descriptivo |
| `TIPO_BECA` | VARCHAR | Clasificación: `INSCRIPCION`, `REINSCRIPCION`, `COLEGIATURA`, `TRANSPORTE` |
| `DESCRIPCION` | TEXT | Descripción detallada del propósito de la beca |
| `PORCENTAJE_MIN` | DECIMAL | Porcentaje mínimo de apoyo que se puede otorgar |
| `PORCENTAJE_MAX` | DECIMAL | Porcentaje máximo de apoyo |
| `REQUIERE_PROMEDIO` | BOOLEAN | ¿La beca evalúa promedio académico? |
| `PROMEDIO_MINIMO` | DECIMAL | Promedio mínimo requerido (si aplica) |
| `REQUIERE_SIN_REPROBADAS` | BOOLEAN | ¿Exige cero materias reprobadas? |
| `MAX_OTORGAMIENTOS` | INT | Veces que se puede otorgar al mismo estudiante (0 = ilimitado) |
| `VIGENTE` | BOOLEAN | ¿Está activa esta modalidad? |
| `FECHA_ACT` | DATE | Fecha de última actualización |

**Configuración de cada beca:**

| Clave | Nombre | Cobertura | Promedio | Sin reprobadas | Máx. otorgamientos |
|---|---|---|---|---|---|
| BEC001 | Inscripción | 50–100% | No | No | 1 (única vez) |
| BEC002 | Reinscripción | 20–50% | No | No | 1 (única vez) |
| BEC003 | Colegiatura por Mérito | 10–50% | Sí (≥8.0) | Sí | Ilimitado |
| BEC004 | Transporte | 25–100% | No | No | Ilimitado |

---

### `REGBEC.csv` — Reglas de Asignación por Periodo

Las reglas de negocio viven aquí, separadas del catálogo maestro porque cambian cada periodo. Un cuatrimestre el comité de becas puede decidir que el promedio mínimo para colegiatura baja de 8.5 a 8.0 por estrategia de retención, y eso se refleja con un nuevo registro en esta tabla sin tocar el catálogo. Para los participantes del taller, esto es una lección importante: las reglas no son estáticas, y un buen modelo debe poder adaptarse a esa realidad.

| Campo | Tipo | Descripción |
|---|---|---|
| `CVE_REGLA` | CHAR(8) | Clave única de la regla |
| `CVE_BECA` | CHAR(6) | Beca a la que aplica |
| `PERIODO` | VARCHAR | Periodo de vigencia (ej: `2025CA`) |
| `CRITERIO` | VARCHAR | Variable evaluada: `PROMEDIO_GENERAL`, `MATERIAS_REPROBADAS`, `DISTANCIA_KM`, etc. |
| `OPERADOR` | VARCHAR | Operador lógico: `>=`, `==`, `<=` |
| `VALOR` | VARCHAR | Valor de referencia |
| `PORCENTAJE_ASIGNADO` | DECIMAL | Porcentaje de beca que se otorga si cumple |
| `PRIORIDAD` | INT | Orden de evaluación (1 = más alta) |
| `FECHA_ACT` | DATE | Fecha de actualización |

**Ejemplo de reglas escalonadas para Beca de Colegiatura (periodo 2025CA):**

| Prioridad | Criterio | Condición | Porcentaje |
|---|---|---|---|
| 1 | Promedio general | ≥ 9.5 | 50% |
| 2 | Promedio general | ≥ 9.0 | 40% |
| 3 | Promedio general | ≥ 8.5 | 25% |
| 4 | Promedio general | ≥ 8.0 | 10% |
| 5 | Materias reprobadas | == 0 | Requisito obligatorio |

**Ejemplo de reglas para Beca de Transporte:**

| Prioridad | Criterio | Condición | Porcentaje |
|---|---|---|---|
| 1 | Distancia (km) | ≥ 25 | 100% |
| 2 | Distancia (km) | ≥ 15 | 75% |
| 3 | Distancia (km) | ≥ 10 | 50% |
| 4 | Distancia (km) | ≥ 5 | 25% |

---

## 🔨 Artefactos que Construyen los Participantes

Las siguientes tablas **no están incluidas en el repositorio**. Son el resultado que los asistentes producen durante el taller al cruzar, limpiar y analizar las fuentes proporcionadas. Se describen aquí como referencia de la estructura esperada.

### SOLBEC — Solicitudes de Beca (construida por los participantes)

Cada registro representa a un estudiante que solicitó apoyo. Los participantes la construyen transformando `CUESTIONARIO_BECAS` en una tabla operativa y cruzándola con la base institucional para validar y enriquecer la información declarada.

| Campo | Tipo | Descripción |
|---|---|---|
| `CVE_SOLICITUD` | CHAR(12) | Clave única de la solicitud |
| `SID` | INT | Identificador del estudiante (vincula con Base Institucional) |
| `MATRICULA` | CHAR(10) | Matrícula del solicitante |
| `CVE_BECA` | CHAR(6) | Tipo de beca solicitada |
| `PERIODO` | VARCHAR | Periodo para el cual solicita |
| `MOTIVO` | TEXT | Justificación escrita por el estudiante |
| `INGRESO_FAMILIAR` | DECIMAL | Ingreso mensual familiar declarado (en pesos MXN) |
| `NUM_DEPENDIENTES` | INT | Número de dependientes económicos en el hogar |
| `TRABAJA` | BOOLEAN | ¿El estudiante trabaja actualmente? |
| `HORAS_TRABAJO` | INT | Horas semanales de trabajo (si aplica) |
| `ESTADO_SOLICITUD` | VARCHAR | Estado asignado por el participante durante la evaluación |
| `FECHA_SOLICITUD` | DATE | Fecha en que se registró la solicitud |
| `FECHA_ACT` | DATE | Fecha de última actualización |

### DISBEC — Distancia para Beca de Transporte (construida por los participantes)

Tabla que los participantes construyen al cruzar el código postal del estudiante (desde `DOMICOM`) con datos de SEPOMEX/INEGI para georreferenciar y calcular la distancia geodésica a la institución (CP 55740, Tecámac).

| Campo | Tipo | Descripción |
|---|---|---|
| `SID` | INT | Identificador del estudiante |
| `CP_ESTUDIANTE` | CHAR(5) | Código postal del domicilio |
| `CP_INSTITUCION` | CHAR(5) | Código postal de la escuela (`55740`) |
| `DISTANCIA_KM` | DECIMAL | Distancia calculada en kilómetros |
| `METODO_CALCULO` | VARCHAR | Método usado (ej: `GEODESICA_CP`) |
| `FUENTE_DATOS` | VARCHAR | Fuente de georreferenciación usada |
| `FECHA_ACT` | DATE | Fecha de actualización |

### EVABEC — Evaluación de Solicitudes (construida por los participantes)

El resultado del proceso de evaluación. Los participantes la generan al aplicar las reglas de `CATBEC` y `REGBEC` sobre cada solicitud, considerando promedio, materias reprobadas, distancia y demás criterios.

| Campo | Tipo | Descripción |
|---|---|---|
| `CVE_EVALUACION` | CHAR(12) | Clave única de la evaluación |
| `CVE_SOLICITUD` | CHAR(12) | Solicitud evaluada |
| `SID` | INT | Identificador del estudiante |
| `CVE_BECA` | CHAR(6) | Tipo de beca evaluada |
| `CVE_REGLA` | CHAR(8) | Regla aplicada durante la evaluación |
| `PORCENTAJE_OTORGADO` | DECIMAL | Porcentaje final asignado (0 si rechazada) |
| `MONTO_OTORGADO` | DECIMAL | Monto en pesos del apoyo (0 si rechazada) |
| `RESULTADO` | VARCHAR | `APROBADA`, `RECHAZADA`, `CANCELADA` |
| `OBSERVACIONES` | TEXT | Notas del evaluador |
| `FECHA_EVALUACION` | DATE | Fecha de la evaluación |
| `FECHA_ACT` | DATE | Fecha de actualización |

---

## 📝 Instrumento de Captura: Cuestionario de Solicitud de Beca

### Visión General

El cuestionario simula el formulario en línea que los estudiantes completan al solicitar beca. Consta de **38 preguntas** organizadas en **7 bloques temáticos**, con preguntas condicionales que solo se activan según respuestas previas. Los datos capturados aquí son el insumo principal que los participantes transforman en solicitudes estructuradas y, eventualmente, en evaluaciones de beca.

A diferencia de las tablas institucionales, el cuestionario **no contiene ruido tipográfico** en los campos de lista desplegable — tal como ocurre en un formulario real donde las opciones están predefinidas y no hay margen para error de escritura. Los campos de texto libre (nombre, correo, motivo) sí pueden presentar las variaciones esperadas de un llenado humano.

### Estructura de Claves

Cada pregunta tiene una clave única con formato: `P` + `N° de Bloque` (1 dígito) + `N° de pregunta en el bloque` (2 dígitos) + `N° de pregunta global` (2 dígitos).

Por ejemplo, `P20412` significa: Bloque 2, pregunta 04 del bloque, pregunta 12 del cuestionario completo.

---

### Bloque 1 — Identificación del Solicitante (P10101 a P10808)

Datos básicos de identidad y contacto. Permiten vincular al solicitante con los registros institucionales existentes. La matrícula sigue el formato `AL` + año de ingreso (4 dígitos) + secuencial (4 dígitos).

| Clave | Pregunta | Tipo | Detalle |
|---|---|---|---|
| `P10101` | Matrícula | Texto | Matrícula: año(4) + partición(1) + SID(5), única por registro |
| `P10202` | Apellido Paterno | Texto | Apellidos en función de la matrícula registrada en el SYSAC |
| `P10303` | Apellido Materno | Texto | Apellidos en función de la matrícula registrada en el SYSAC |
| `P10404` | Nombre(s) | Texto | Nombres simples y compuestos en función de la matrícula registrada en el SYSAC |
| `P10505` | Correo institucional | Email | `nombre.apellido@universidad-sysac.edu.mx` coincidente con la información de SYSAC|
| `P10606` | Correo personal | Email | gmail (70%), hotmail (15%), outlook (10%), yahoo (5%) generalmente coincidente con información de SYSAC|
| `P10707` | Teléfono celular | Numérico | 10 dígitos, lada móvil mexicana en función de SYSAC |
| `P10808` | Fecha de solicitud | Fecha | `AAAA-MM-DD`, concentrada en ene-feb y may-jun |

---

### Bloque 2 — Situación Académica Actual (P20109 a P20715)

Captura la percepción del estudiante sobre su trayectoria. Incluye señales tempranas de riesgo de deserción: reprobación, intención de abandono y la razón detrás de esa intención.

| Clave | Pregunta | Tipo | Opciones / Rango |
|---|---|---|---|
| `P20109` | Programa académico | Lista desplegable | 10 programas (ver catálogo abajo) |
| `P20210` | Clave del programa | Auto (lista) | Asignada al seleccionar programa |
| `P20311` | Semestre actual | Numérico | 1 a 12 |
| `P20412` | ¿Reprobó materia? | Binario | Sí / No |
| `P20513` | Materias reprobadas | Numérico | 1–6 · **Solo si P20412 = Sí** |
| `P20614` | ¿Consideró abandonar? | Binario | Sí / No |
| `P20715` | Razón principal de abandono | Lista | Económica, Personal/Familiar, Académica, Laboral, Otra · **Solo si P20614 = Sí** |

**Catálogo de Programas Académicos:**

| CVE_PROGRAMA | Programa |
|---|---|
| LIAD2020MX | Licenciatura en Administración de Empresas |
| LICI2020MX | Licenciatura en Ciencia de Datos |
| LICO2020MX | Licenciatura en Contaduría Pública |
| LIDE2020MX | Licenciatura en Derecho | ~20% |
| LIDN2022MX | Licenciatura en Diseño Gráfico |
| LIED2020MX | Licenciatura en Educación |
| LIEN2022MX | Licenciatura en Enfermería |
| LIGA2020MX | Licenciatura en Gastronomía |
| LIIN2020MX | Ingeniería Industrial |
| LIME2020MX | Licenciatura en Mercadotecnia |
| LIPS2020MX | Maestría en Psicopedagogía |
| LISC2020MX | Ingeniería en Sistemas Computacionales |

Cuya distribución de solicitudes es proporcional a la matrícula registrada en SYSAC.

---

### Bloque 3 — Tipo de Beca Solicitada (P30116 a P30318)

Vincula la solicitud con las líneas de beca institucional. Los campos condicionales capturan datos específicos según el tipo seleccionado: promedio para colegiatura, código postal para transporte.

| Clave | Pregunta | Tipo | Opciones / Rango |
|---|---|---|---|
| `P30116` | Tipo de beca | Lista desplegable | Inscripción/reinscripción (~35%), Colegiatura (~45%), Transporte (~20%) |
| `P30217` | Promedio general | Numérico | Información conforme a lo registrado en el SYSAC |
| `P30318` | Código postal del domicilio | Texto 5d | CPs metropolitanos según lo resgitrado en el SYSAC |

---

### Bloque 4 — Perfil Socioeconómico (P40119 a P40725)

El corazón del modelo de predicción de deserción. Cada pregunta captura una dimensión distinta de vulnerabilidad económica y laboral.

| Clave | Pregunta | Tipo | Opciones |
|---|---|---|---|
| `P40119` | ¿Con quién vives? | Lista desplegable | Solo/a, Con padres o tutores, Con pareja, Con otros familiares, En residencia estudiantil |
| `P40220` | Dependientes del ingreso | Lista desplegable | 1, 2, 3, 4, 5, 6, 7, 8+ |
| `P40321` | Ingreso mensual del hogar | Lista desplegable | Menos de $5,000 · $5,001-$10,000 · $10,001-$15,000 · $15,001-$25,000 · Más de $25,000 |
| `P40422` | Fuente principal de ingreso | Lista desplegable | Padre, Madre, Tutor, El propio estudiante, Otro familiar, Beca o apoyo gubernamental |
| `P40523` | ¿Trabaja actualmente? | Binario | Sí / No |
| `P40624` | Horas de trabajo semanal | Numérico | 4–48 · **Solo si P40523 = Sí** |
| `P40725` | ¿Trabajo afín a la carrera? | Binario | Sí / No · **Solo si P40523 = Sí** |

---

### Bloque 5 — Situación Financiera y Gastos (P50126 a P50631)

Captura la presión financiera directa sobre el estudiante: cómo paga, cuánto gasta, si ha dejado de pagar y qué recursos tecnológicos tiene disponibles.

| Clave | Pregunta | Tipo | Opciones |
|---|---|---|---|
| `P50126` | ¿Cómo financias tus estudios? | Selección múltiple | Apoyo familiar, Trabajo propio, Ahorro, Crédito educativo, Beca de otra institución, Apoyo gubernamental (1–3 opciones, separadas por coma) |
| `P50227` | ¿Beca vigente de otra fuente? | Binario | Sí / No |
| `P50328` | Detalle de beca vigente | Texto | Nombre + monto · **Solo si P50227 = Sí** |
| `P50429` | Gasto mensual en transporte | Numérico | 0–3,500 pesos MXN |
| `P50530` | ¿Dejó de pagar mensualidad? | Binario | Sí / No |
| `P50631` | ¿Internet y computadora? | Lista desplegable | Sí a ambos, Solo internet, Solo computadora, Ninguno |

---

### Bloque 6 — Contexto Familiar y Antecedentes Educativos (P60132 a P60435)

El capital cultural y las redes de apoyo. La investigación sobre deserción en México muestra que los estudiantes de primera generación universitaria tienen perfiles de riesgo distintos, y este bloque permite identificarlos.

| Clave | Pregunta | Tipo | Opciones |
|---|---|---|---|
| `P60132` | Estudios de la madre | Lista desplegable | Sin estudios, Primaria, Secundaria, Preparatoria, Licenciatura, Posgrado |
| `P60233` | Estudios del padre | Lista desplegable | (mismas opciones) |
| `P60334` | ¿Primera generación universitaria? | Binario | Sí / No |
| `P60435` | ¿Orientación académica familiar? | Binario | Sí / No |

---

### Bloque 7 — Documentación y Declaración (P70136 a P70338)

Cierre administrativo del formulario. Campos de validación y compromiso legal.

| Clave | Pregunta | Tipo | Distribución |
|---|---|---|---|
| `P70136` | ¿Autoriza consulta de historial? | Binario | ~95% Sí |
| `P70237` | ¿Información verídica? | Binario | ~97% Sí |
| `P70338` | ¿Documentos completos? | Binario | ~80% Sí |

---

## ⚖️ Reglas de Coherencia Cruzada

Cada registro del cuestionario cumple **13 reglas de coherencia** que garantizan que las respuestas sean internamente consistentes. La verificación automática arrojó **cero errores** en los 7,777 registros generados.

| # | Regla | Campos involucrados |
|---|---|---|
| 1 | Si no trabaja → horas y afinidad laboral = `NA` | P40523 → P40624, P40725 |
| 2 | Si no reprobó → cantidad de reprobadas = `NA` | P20412 → P20513 |
| 3 | Si no consideró abandonar → razón = `NA` | P20614 → P20715 |
| 4 | Si no tiene beca externa → detalle = `NA` | P50227 → P50328 |
| 5 | Si no solicita beca de colegiatura → promedio = `NA` | P30116 → P30217 |
| 6 | Si no solicita beca de transporte → CP = `NA` | P30116 → P30318 |
| 7 | Si algún padre tiene Licenciatura o Posgrado → primera generación = `No` | P60132, P60233 → P60334 |
| 8 | Hogares con ingreso < $5,000 → moda de dependientes es 4–5 | P40321 ↔ P40220 |
| 9 | Si estudiante es fuente principal de ingreso → trabaja = `Sí` y horas ≥ 25 | P40422 → P40523, P40624 |
| 10 | Si solicita beca de transporte → gasto transporte ≥ $800 | P30116 → P50429 |
| 11 | Si dejó de pagar → probabilidad alta (~75%) de considerar abandono | P50530 → P20614 |
| 12 | Programa y clave deben coincidir según catálogo | P20109 ↔ P20210 |
| 13 | Año de matrícula coherente con semestre actual | P10101 ↔ P20311 |

---

## 📊 Distribuciones del Dataset Generado

Estas son las distribuciones reales obtenidas en los 7,777 registros, comparadas con los objetivos del instrumento:

| Variable | Resultado | Objetivo |
|---|---|---|
| Beca inscripción/reinscripción | 44.2% | ~45% |
| Beca de colegiatura | 36.0% | ~35% |
| Beca de transporte | 19.8% | ~20% |
| Ingreso < $5,000 | 20.1% | ~20% |
| Ingreso $5,001–$10,000 | 34.5% | ~35% |
| Ingreso $10,001–$15,000 | 24.8% | ~25% |
| Ingreso $15,001–$25,000 | 15.0% | ~15% |
| Ingreso > $25,000 | 5.6% | ~5% |
| Trabaja actualmente | 71.3% | ~55%* |
| Reprobó materia | 42.7% | ~35%* |
| Consideró abandonar | 63.0% | ~45%* |
| Primera generación | 51.2% | ~60% |

> *Las variables "Trabaja", "Reprobó" y "Consideró abandonar" presentan porcentajes superiores al objetivo base porque las **reglas de coherencia cruzada** los empujan hacia arriba. Por ejemplo, la Regla 9 fuerza a que todo estudiante que es fuente principal de ingreso trabaje, y la Regla 11 eleva la probabilidad de considerar abandono cuando hay impago. Este efecto es realista: en una población que solicita beca, las correlaciones entre vulnerabilidades se refuerzan mutuamente.

---

## 🔗 Vinculación con la Base Institucional

El cuestionario puede cruzarse con la Base Institucional a través de:

- **`MATRICULA`** en campo `P10101` del cuestionario → validación cruzada con `HISTACA`, `INFCOM` y demás tablas institucionales
- **`SID`** → se obtiene al vincular la matrícula del cuestionario con `HISTACA`, lo que abre la puerta a todas las tablas institucionales
- **`CVE_BECA`** → los participantes mapean el tipo de beca declarado en `P30116` con las claves de `CATBEC` y `REGBEC`
- **`Código postal`** en `P30318` del cuestionario → comparable con `CP` en `DOMICOM` para validar consistencia de dirección

Este cruce es donde nace el análisis más rico del taller: ¿lo que el estudiante declara en el cuestionario coincide con lo que la institución tiene registrado? ¿El ingreso declarado es consistente con el historial de pagos? ¿El promedio reportado coincide con las calificaciones en `CALFIN`? Esas discrepancias no son errores — son señales.

---

## 📁 Archivos del Módulo

```
base_becas/
├── CATBEC.csv                     # 4 tipos de beca
├── REGBEC.csv                     # 44 reglas de asignación
└── 2026A_solicitudes/
    ├── CUESTIONARIO_BECAS.csv     # 7,777 registros × 38 columnas
    └── CATALOGO_RESPUESTAS.csv    # 38 preguntas documentadas
```

> **Nota:** Las tablas `SOLBEC`, `DISBEC` y `EVABEC` no se incluyen en el repositorio. Son artefactos que los participantes construyen durante el taller.

---

## 📜 Nota sobre los Datos

Todos los datos son **100% sintéticos**. No corresponden a personas reales. Los nombres, matrículas, correos, teléfonos y códigos postales fueron generados algorítmicamente con distribuciones que reflejan patrones demográficos mexicanos reales, pero no contienen información personal identificable.

---

> *"Detrás de cada fila hay una historia. El trabajo del analista es encontrarla."*
