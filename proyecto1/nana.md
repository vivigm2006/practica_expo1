# Diccionario de datos: Modelo Desnormalizado (Esquema Estrella)
**Proyecto:** Análisis de Incendios Forestales en EE. UU. (2000-2015)

Este documento detalla la estructura del modelo dimensional (Esquema en Estrella) construido para optimizar el rendimiento en Power BI y Streamlit. Se compone de una tabla de hechos central y seis tablas de dimensiones que la rodean.

### Convención de la columna "Origen":
*   **FUENTE**: Proviene directamente de la tabla `fires` (ya limpia y normalizada).
*   **DERIVADO**: Calculado durante el proceso de desnormalización (SQL) a partir de la fuente.
*   **CONSTRUIDO**: Clasificación o regla de negocio elaborada por el equipo.

---

## 1. Tabla de Hechos: `fact_incendio`
**Granularidad:** Un registro por incendio detectado.
**Filtro:** Sólo incluye registros con fecha de descubrimiento entre el año 2000 y 2015.

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `fod_id` | INTEGER PK | FUENTE | Identificador único del incendio en la FPA FOD. |
| `fecha_key` | INTEGER FK | DERIVADO | Llave foránea hacia `dim_fecha` (formato AAAAMMDD). |
| `region_key` | INTEGER FK | DERIVADO | Llave foránea hacia `dim_region`. |
| `causa_key` | INTEGER FK | FUENTE | Llave foránea hacia `dim_causa` (STAT_CAUSE_CODE). |
| `administracion_key`| INTEGER FK | DERIVADO | Llave foránea hacia `dim_administracion` (OWNER_CODE + 1). |
| `clase` | VARCHAR(1) FK| FUENTE | Llave foránea hacia `dim_tamano` (Clase de tamaño A-G). |
| `ubicacion_key` | INTEGER FK | DERIVADO | Llave foránea hacia `dim_ubicacion`. |
| `nwcg_unit_id` | VARCHAR | FUENTE | Atributo degenerado: código de la unidad que reportó el incendio. |
| `incendios` | INTEGER | DERIVADO | Métrica base. Siempre vale 1 (se usa para hacer `SUM` y contar). |
| `acres` | DOUBLE | FUENTE | Métrica de superficie quemada (FIRE_SIZE). |
| `tiene_contencion` | INTEGER | DERIVADO | Bandera (1/0). Vale 1 si el incendio tiene fecha de control, 0 si no. |
| `dias_contencion` | INTEGER | DERIVADO | Días entre la detección y la contención. Nulo si no hay fecha de fin. |

---

## 2. Tablas de Dimensión

### 2.1 `dim_fecha` (Responde al Objetivo Específico 5)
**Granularidad:** Una fila por cada fecha única de descubrimiento.

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `fecha_key` | INTEGER PK | DERIVADO | Llave primaria (formato AAAAMMDD). |
| `fecha` | DATE | FUENTE | Fecha de descubrimiento completa. |
| `anio` | INTEGER | DERIVADO | Año extraído de la fecha. |
| `trimestre` | INTEGER | DERIVADO | Trimestre del año (1 al 4). |
| `mes` | INTEGER | DERIVADO | Mes del año (1 al 12). |
| `nombre_mes` | VARCHAR | DERIVADO | Nombre del mes (Ej: January). |
| `estacion` | VARCHAR | CONSTRUIDO | Invierno, Primavera, Verano u Otoño (según el mes). |

### 2.2 `dim_region` (Responde al Objetivo General y Obj. 1, 2, 3, 4)
**Granularidad:** Una fila por cada Región NIFC.

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `region_key` | INTEGER PK | DERIVADO | Llave subrogada generada con `DENSE_RANK()`. |
| `region_code` | VARCHAR(2) | FUENTE | Código NIFC (Ej: GB, NW, CA). |
| `region_name` | VARCHAR | FUENTE | Nombre del área geográfica (Ej: Great Basin Area). |

### 2.3 `dim_causa` (Responde al Objetivo Específico 2)
**Granularidad:** Una fila por cada causa específica.

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `causa_key` | INTEGER PK | FUENTE | Código original de causa (STAT_CAUSE_CODE). |
| `causa_especifica` | VARCHAR | FUENTE | Descripción de la causa (Ej: Lightning, Arson). |
| `origen_causa` | VARCHAR | CONSTRUIDO | Agrupación: "Natural" (solo rayos) o "Humana" (el resto). |

### 2.4 `dim_administracion` (Responde al Objetivo Específico 4)
**Granularidad:** Una fila por cada propietario del terreno.

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `administracion_key` | INTEGER PK | DERIVADO | Código original + 1. |
| `propietario` | VARCHAR | FUENTE | Nombre del dueño original (OWNER_DESCR). |
| `tipo_administracion`| VARCHAR | CONSTRUIDO | Agrupación: Federal, Estatal/Local o Privada. |

### 2.5 `dim_tamano` (Responde al Objetivo Específico 3)
**Granularidad:** Una fila por cada clase de tamaño (A-G).

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `clase` | VARCHAR(1) PK | FUENTE | Letra de la clase (A a la G). |
| `rango` | VARCHAR | CONSTRUIDO | Texto descriptivo del rango (Ej: >= 5000 acres). |
| `orden` | INTEGER | CONSTRUIDO | Número del 1 al 7 para ordenar gráficos. |
| `es_megaincendio` | INTEGER | CONSTRUIDO | Bandera (1/0). Vale 1 solo para la clase G (>= 5000 acres). |

### 2.6 `dim_ubicacion` (Responde al Objetivo Específico 1)
**Granularidad:** Una fila por cada combinación única de Estado y Condado.

| Campo | Tipo | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| `ubicacion_key` | INTEGER PK | DERIVADO | Llave subrogada generada con `DENSE_RANK()`. |
| `state_code` | VARCHAR(2) | FUENTE | Código del estado (Ej: CA). |
| `state_name` | VARCHAR | FUENTE | Nombre del estado (Ej: California). |
| `fips_5` | VARCHAR(5) | DERIVADO | Concatenación: state_code + fips_code. |
| `county_name` | VARCHAR | FUENTE | Nombre del condado. Si es nulo, dice "Desconocido". |
| `land_sq_mi` | DOUBLE | FUENTE | Extensión de tierra del condado (excluye ríos y lagos). |