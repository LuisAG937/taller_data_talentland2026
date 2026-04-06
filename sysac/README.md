# 🏛️ README — Base Institucional SYSAC

**Sistema SYSAC · Módulo de Información Institucional del Estudiante**

> Este documento describe la estructura, campos y características de las 8 tablas que conforman la base institucional del Centro de Estudios Superiores y Posgrado de México (CESUPOM). Aquí vive todo lo que la escuela ya sabe del estudiante antes de que solicite una beca.

---

## 📋 Descripción General

La base institucional representa la **perspectiva de la universidad**. Son los datos que el CESUPOM recopila como parte de su operación cotidiana: quién se inscribió, en qué programa, qué calificaciones obtuvo, cuánto ha pagado, dónde vive, cómo contactarlo. En un sistema real, esta información viviría dispersa en diferentes módulos de un ERP académico — y rara vez se cruza de forma deliberada.

Para el taller, estas 8 tablas son el primer bloque que los participantes cargan y exploran. Contienen **ruido intencional** que simula la realidad de datos institucionales: fechas con formatos inconsistentes, valores de texto con variaciones de escritura, registros duplicados, campos vacíos y códigos postales malformados. Parte del reto consiste en detectar, documentar y resolver estos problemas antes de que el análisis pueda avanzar.

La llave maestra que conecta todas las tablas es el **`SID`** (Student ID), un identificador numérico ascendente que aparece en cada una. Además, campos como `CVE_PROGRAMA`, `CVE_MATERIA` y `PERIODO` permiten cruzar la trayectoria académica con los pagos y las calificaciones de cada estudiante.

---

## 📊 Panorama General

| Tabla | Descripción | Registros | Campos |
|---|---|---:|---:|
| `HISTACA` | Historial académico | 10,568 | 6 |
| `INFCOM` | Información personal | 10,509 | 6 |
| `DOMICOM` | Domicilios | 10,369 | 7 |
| `CORCOM` | Correos electrónicos | 18,626 | 4 |
| `TELCOM` | Teléfonos | 14,493 | 4 |
| `PLANACA` | Plan académico (catálogo de materias) | 260 | 6 |
| `CALFIN` | Calificaciones finales | 205,442 | 7 |
| `PAGFIN` | Pagos financieros | 53,263 | 7 |

**Total: ~323,530 registros** distribuidos en 8 archivos CSV.

---

## 📑 Detalle de Cada Tabla

### `HISTACA.csv` — Historial Académico

La tabla que define quién es cada estudiante dentro del sistema. Cada registro representa la inscripción de un estudiante a un programa en un periodo determinado. Un mismo estudiante puede tener múltiples registros si ha estado inscrito en varios periodos, lo cual explica por qué esta tabla tiene más filas (10,568) que el total de estudiantes únicos (10,369). Es la tabla raíz desde donde se construye la trayectoria de cada persona.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador único del estudiante | 1 |
| `MATRICULA` | CHAR(10) | Matrícula: año(4) + partición(1) + SID(5) | 2022A00001 |
| `CVE_PROGRAMA` | VARCHAR | Clave del programa académico | LIME2020MX |
| `ESTADO` | VARCHAR | Estado académico del estudiante | Activo |
| `PERIODO` | VARCHAR | Periodo de inscripción | 2024CC |
| `FECHA_ACT` | DATE | Fecha de actualización | 2025-01-24 |

**Valores del campo ESTADO:**

La distribución real de estados académicos refleja una universidad en operación, donde la mayoría de los registros son estudiantes activos, pero existe un porcentaje significativo de egresos, titulaciones y bajas. Los estados son: `Activo`, `Egresado`, `Titulado`, `Baja Definitiva` y `Baja Temporal`.

**Ruido presente:** El campo `ESTADO` contiene variaciones intencionales como `"activo"`, `"ACTIVO"`, `"Activo "` (con espacio al final) y `" Activo"` (con espacio al inicio). También hay una pequeña proporción (~2%) de registros duplicados. Este desorden es pedagógico: normalizar estos valores es uno de los primeros ejercicios de limpieza del taller.

**Catálogo de Programas Académicos (12 claves):**

| CVE_PROGRAMA | Programa |
|---|---|
| LIAD2020MX | Licenciatura en Administración de Empresas |
| LICI2020MX | Licenciatura en Ciencia de Datos |
| LICO2020MX | Licenciatura en Contaduría Pública |
| LIDE2020MX | Licenciatura en Derecho |
| LIDN2022MX | Licenciatura en Diseño Gráfico |
| LIED2020MX | Licenciatura en Educación |
| LIEN2022MX | Licenciatura en Enfermería |
| LIGA2020MX | Licenciatura en Gastronomía |
| LIIN2020MX | Ingeniería Industrial |
| LIME2020MX | Licenciatura en Mercadotecnia |
| LIPS2020MX | Maestría en Psicopedagogía |
| LISC2020MX | Ingeniería en Sistemas Computacionales |

> **Nota sobre la estructura de la clave:** El formato es `LI` (nivel) + siglas del programa (2 caracteres) + año de registro SEP (4 dígitos) + sede (`MX`). Esta codificación simula cómo las instituciones registran sus programas ante la autoridad educativa.

---

### `INFCOM.csv` — Información Complementaria

Los datos personales de toda la comunidad del CESUPOM. Esta tabla no solo contiene estudiantes: también incluye **profesores y administrativos**, mezclados en los mismos registros. Eso es exactamente como funciona en muchos sistemas reales — una sola tabla de "personas" — y obliga a los participantes a filtrar por tipo antes de cualquier análisis estudiantil. Si no lo hacen, sus conteos van a salir inflados y sus promedios distorsionados.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador único | 1 |
| `NOMBRE` | VARCHAR | Nombre completo (nombre + apellidos) | Saúl Trejo García |
| `FECHA_NAC` | DATE | Fecha de nacimiento | 1993-08-02 |
| `GENERO` | VARCHAR | Género del registro | M |
| `CURP` | VARCHAR | Clave Única de Registro de Población | TEGS930802HDFRRL61 |
| `FECHA_ACT` | DATE | Fecha de actualización | 2024-11-24 |

**Ruido presente:** Esta tabla es particularmente rica en imperfecciones intencionales. El campo `GENERO` mezcla formatos como `"M"`, `"F"`, `"H"`, `"Masculino"`, `"Femenino"`, `"m"`, `"f"`, `"O"` y valores vacíos. Los CURPs incluyen algunos truncados, en minúsculas o con valor `"PENDIENTE"`. Además, hay registros fantasma (~1.5%) donde el SID viene vacío o nulo — exactamente el tipo de anomalía que un analista debe detectar y decidir cómo manejar.

---

### `DOMICOM.csv` — Domicilios de la Comunidad

La dirección registrada de cada miembro de la comunidad. Esta tabla es crítica para el taller porque el **código postal** (`CP`) es la pieza que permite calcular la distancia a la institución (CP 55740, Tecámac) y, por lo tanto, evaluar la elegibilidad para la beca de transporte. También permite vincular con el Índice de Rezago Social de CONEVAL a nivel municipal.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador único | 1 |
| `CALLE` | VARCHAR | Nombre de la calle y número | Eje 6 Sur 649 |
| `COLONIA` | VARCHAR | Colonia | Escandón |
| `MUNICIPIO` | VARCHAR | Municipio o alcaldía | Coyoacán |
| `ENTIDAD` | VARCHAR | Entidad federativa | Ciudad de México |
| `CP` | VARCHAR | Código postal | 03100 |
| `FECHA_ACT` | DATE | Fecha de actualización | 2025-01-01 |

**Nota sobre `ENTIDAD`:** Este campo se nombró deliberadamente así (y no `ESTADO`) para evitar ambigüedad con el campo `ESTADO` de `HISTACA`, que se refiere al estatus académico. En el contexto mexicano, "entidad federativa" es el término oficial que usa el INEGI.

**Ruido presente:** Algunos códigos postales están truncados (4 dígitos en lugar de 5), otros tienen la letra `"O"` en lugar del número `"0"`, y hay campos con espacios en blanco al inicio o al final. Los participantes deberán limpiar estos valores antes de poder cruzar con SEPOMEX.

---

### `CORCOM.csv` — Correos de la Comunidad

Tabla de correos electrónicos. Cada persona puede tener múltiples registros — típicamente uno institucional y uno personal — lo que explica por qué esta tabla tiene 18,626 filas para una comunidad de ~10,500 personas. El correo institucional sigue el formato `nombre.apellido@universidad-sysac.edu.mx`.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador único | 1 |
| `CORREO` | VARCHAR | Dirección de correo electrónico | saul.trejo@universidad-sysac.edu.mx |
| `TIPO_CORREO` | VARCHAR | Tipo de correo: Institucional / Personal | Institucional |
| `FECHA_ACT` | DATE | Fecha de actualización | 2024-04-05 |

**Dominios de correos personales:** Los correos personales usan dominios reales como gmail.com, icloud.com, live.com.mx, hotmail.com, outlook.com y yahoo.com.mx, con distribuciones que reflejan el uso real en México.

**Ruido presente:** El campo `TIPO_CORREO` viene con las variaciones clásicas de datos no controlados: `"Personal"`, `"personal"`, `"PERSONAL"`, `"personal "` (con espacio). Los correos institucionales están limpios porque los genera el sistema; los personales son declarados por el usuario y ahí es donde aparecen las irregularidades.

---

### `TELCOM.csv` — Teléfonos de la Comunidad

Tabla de números telefónicos. Cada persona puede tener varios registros (celular, casa, fijo), generando 14,493 filas. Todos los estudiantes tienen al menos un celular registrado.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador único | 1 |
| `TELEFONO` | VARCHAR | Número telefónico | 5515879193 |
| `TIPO_TEL` | VARCHAR | Tipo: Celular / Casa / Fijo | Celular |
| `FECHA_ACT` | DATE | Fecha de actualización | 2024-06-04 |

**Ruido presente:** El campo `TIPO_TEL` mezcla `"Casa"`, `"CASA"`, `"casa"`, `"Fijo"` como variantes del mismo concepto. Algunos teléfonos pueden tener formato irregular: paréntesis, guiones, dígitos de más o de menos. La lada no está separada como campo independiente — los participantes que quieran analizarla deberán extraerla ellos mismos.

---

### `PLANACA.csv` — Plan Académico

El catálogo maestro de materias. Define qué materias pertenecen a cada programa, con sus créditos y el cuatrimestre/semestre en que se cursan. Esta tabla **no contiene ruido** porque simula un catálogo administrado desde el sistema, donde las opciones están predefinidas y controladas.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `CVE_PROGRAMA` | VARCHAR | Clave del programa | LIAD2020MX |
| `CVE_MATERIA` | CHAR(8) | Clave de la materia | ADSEM010 |
| `NOMBRE_MATERIA` | VARCHAR | Nombre completo de la materia | Seminario de Titulación |
| `CREDITOS` | INT | Créditos de la materia | 6 |
| `CUATRIMESTRE` | INT | Cuatrimestre o semestre al que pertenece | 1 |
| `FECHA_ACT` | DATE | Fecha de actualización | 2022-03-31 |

**Distribución de materias:** El catálogo contiene **260 materias** distribuidas en 12 programas. Los programas de posgrado (`LIPS2020MX`, `LICO2020MX`) tienen 20 materias cada uno, mientras que los demás programas de licenciatura manejan 22 materias. La clave de materia sigue el formato: 2 caracteres del programa + abreviatura del nombre + secuencial de 3 dígitos.

> **Nota sobre el campo `CUATRIMESTRE`:** A pesar de su nombre, este campo representa el periodo en el mapa curricular tanto para programas cuatrimestrales como semestrales. Los participantes deben tener en cuenta que el número en este campo no implica la misma duración temporal en todos los programas.

---

### `CALFIN.csv` — Calificaciones Finales

La tabla más voluminosa del sistema con **205,442 registros**. Cada fila representa la calificación final de un estudiante en una materia durante un periodo específico, asignada por un profesor identificado. Es la materia prima para calcular promedios, detectar reprobación y construir indicadores de rendimiento académico.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador del estudiante | 1 |
| `CVE_PROGRAMA` | VARCHAR | Clave del programa | LIME2020MX |
| `CVE_PROFESOR` | INT | Identificador del profesor (SID del profesor) | 2087 |
| `CVE_MATERIA` | CHAR(8) | Clave de la materia | MESEM010 |
| `PERIODO` | VARCHAR | Periodo en que se cursó | 2022CA |
| `CALIFICACION` | DECIMAL/VARCHAR | Calificación final (0 a 10) | 8.3 |
| `FECHA_ACT` | DATE | Fecha de registro | 2022-02-18 |

**Escala y aprobación:** Las calificaciones van de 0 a 10. La mínima aprobatoria es **6.0 para licenciaturas** y **8.0 para posgrados**. Esta diferencia es un *feature* implícito: un 7.5 significa algo muy distinto para un estudiante de Sistemas Computacionales que para uno de Psicopedagogía.

**Ruido presente:** El campo `CALIFICACION` incluye valores no numéricos como `"N/A"`, `"NULL"` o campos vacíos. Algunas fechas tienen formatos inconsistentes. Los participantes deberán limpiar estos valores antes de poder calcular promedios y conteos de reprobación.

---

### `PAGFIN.csv` — Pagos Financieros

El historial financiero de cada estudiante: **53,263 transacciones** que registran inscripciones, reinscripciones y colegiaturas. El campo `SALDO` es particularmente revelador — un saldo acumulado pendiente es una de las señales más fuertes de riesgo de deserción por motivos económicos.

| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `SID` | INT | Identificador del estudiante | 1 |
| `NUM_TRANSACCION` | VARCHAR | Número único de transacción | TXN-2020-000001 |
| `PERIODO` | VARCHAR | Periodo del cargo | 2022CA |
| `TIPO_CARGO` | VARCHAR | Inscripción / Reinscripción / Colegiatura | Inscripción |
| `MONTO` | DECIMAL | Cantidad del cargo en pesos MXN | 3404.67 |
| `SALDO` | DECIMAL | Saldo acumulado pendiente | 0.0 |
| `FECHA_ACT` | DATE | Fecha de registro | 2020-03-26 |

**Sobre los costos:** Cada programa académico maneja su propio valor de colegiatura, con un incremento aproximado del 5% anual para las generaciones de nuevo ingreso. Esto significa que dos estudiantes del mismo programa pero de diferente generación pueden pagar montos distintos — un detalle realista que los participantes descubrirán al explorar los datos.

**Sobre el saldo:** Cuando un estudiante no liquida completamente una colegiatura, el saldo pendiente se va acumulando en las transacciones posteriores. Un saldo de `0.0` significa que el estudiante está al corriente; un saldo positivo creciente es una bandera roja de deserción inminente.

---

## 🔗 Mapa de Relaciones

```
HISTACA ──────┐
  (SID,       │     PLANACA
   CVE_PROG,  │     (CVE_PROG,
   PERIODO)   │      CVE_MATERIA)
      │       │          │
      │       ▼          │
      ├──► CALFIN ◄──────┘
      │    (SID, CVE_PROG,
      │     CVE_MATERIA,
      │     CVE_PROFESOR,
      │     PERIODO)
      │
      ├──► PAGFIN
      │    (SID, PERIODO)
      │
      ├──► INFCOM
      │    (SID)
      │
      ├──► DOMICOM ·····► SEPOMEX / CONEVAL
      │    (SID, CP)       (fuentes externas)
      │
      ├──► CORCOM
      │    (SID)
      │
      └──► TELCOM
           (SID)
```

Todas las tablas se conectan a través del **`SID`**. Los cruces más ricos para el análisis de deserción y asignación de becas son:

- **`HISTACA` + `CALFIN`** → Trayectoria académica completa: promedio por periodo, materias reprobadas, tendencia de rendimiento.
- **`HISTACA` + `PAGFIN`** → Relación entre estatus académico y comportamiento de pago. Un estudiante con saldo creciente y estado "Activo" es un candidato claro a deserción.
- **`DOMICOM` + SEPOMEX** → Distancia a la escuela para beca de transporte.
- **`DOMICOM` + CONEVAL** → Índice de rezago social del municipio del estudiante.
- **`INFCOM` + `HISTACA`** → Perfilamiento demográfico por programa, edad, género.

---

## 🧹 Resumen de Ruido por Tabla

| Tabla | Tipo de ruido | Intensidad |
|---|---|---|
| `HISTACA` | Variaciones en `ESTADO`, registros duplicados (~2%), formatos de fecha | Medio |
| `INFCOM` | `GENERO` con múltiples formatos, CURPs malformados, SIDs vacíos (~1.5%) | Alto |
| `DOMICOM` | CPs truncados o con "O" en vez de "0", espacios en blanco | Medio |
| `CORCOM` | `TIPO_CORREO` con variaciones de caso y espacios | Bajo |
| `TELCOM` | `TIPO_TEL` con variaciones, teléfonos con formato irregular | Bajo |
| `PLANACA` | Sin ruido (catálogo controlado) | Ninguno |
| `CALFIN` | Calificaciones no numéricas (`N/A`, `NULL`, vacíos), fechas inconsistentes | Medio |
| `PAGFIN` | Fechas con formato mixto | Bajo |

---

## 📁 Archivos del Módulo

```
base_institucional/
├── HISTACA.csv     # 10,568 registros — Historial académico
├── PLANACA.csv     #    260 registros — Catálogo de materias
├── INFCOM.csv      # 10,509 registros — Información personal
├── DOMICOM.csv     # 10,369 registros — Domicilios
├── CORCOM.csv      # 18,626 registros — Correos electrónicos
├── TELCOM.csv      # 14,493 registros — Teléfonos
├── CALFIN.csv      #205,442 registros — Calificaciones finales
└── PAGFIN.csv      # 53,263 registros — Pagos financieros
```

---

## 💡 Ejemplo de Perfil Completo de un Estudiante

Para ilustrar cómo se ve la información de un solo estudiante cuando se cruzan todas las tablas — el SID 1:

```
HISTACA:  SID=1, MATRICULA=2022A00001, CVE_PROGRAMA=LIME2020MX, ESTADO=Egresado, PERIODO=2024CC
INFCOM:   SID=1, NOMBRE=Saúl Trejo García, FECHA_NAC=1993-08-02, GENERO=M, CURP=TEGS930802HDFRRL61
DOMICOM:  SID=1, CALLE=Eje 6 Sur 649, COLONIA=Escandón, MUNICIPIO=Coyoacán, ENTIDAD=Ciudad de México, CP=03100
CORCOM:   SID=1, CORREO=saul.trejo@universidad-sysac.edu.mx (Institucional)
          SID=1, CORREO=saul_trejo90@icloud.com (Personal)
TELCOM:   SID=1, TELEFONO=5515879193 (Celular)
          SID=1, TELEFONO=5512776658 (Casa)
CALFIN:   SID=1, múltiples registros con calificaciones por materia y periodo
PAGFIN:   SID=1, múltiples transacciones desde 2020 hasta 2023
```

Perfil: Saúl es egresado de Mercadotecnia, vive en Coyoacán (Ciudad de México, CP 03100), lo que implica un traslado considerable hasta Tecámac. Su historial de pagos muestra saldos pendientes acumulados en algunos periodos. Todo esto se puede reconstruir a partir de los CSV — pero solo si cruzas las tablas correctamente.

---

## 📜 Nota sobre los Datos

Todos los datos son **100% sintéticos**. No corresponden a personas reales. Los nombres, matrículas, CURPs, correos, teléfonos, direcciones y códigos postales fueron generados algorítmicamente con distribuciones que reflejan patrones demográficos mexicanos, pero no contienen información personal identificable.

---

> *"Los datos de una institución educativa son como un rompecabezas: cada tabla es una pieza. El análisis comienza cuando alguien decide armar la imagen completa."*
