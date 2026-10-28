# Diccionario de Datos - Capa Desnormalizada 

Este documento detalla la estructura y el significado de las tablas y campos creados en la fase de **Desnormalización**. El objetivo de esta capa es optimizar el rendimiento para las herramientas de visualización, donde el modelo separa el evento (Tabla de Hechos) de su contexto (Tablas de Dimensión) permitiendo consultas analíticas rápidas y precisas para responder a los objetivos de la investigación.

## 1. Tabla de Hechos (`fact_incendio`)
Representa el evento principal, tiene una granularidad de un registro por cada incendio detectado y contiene las llaves que conectan con las dimensiones y las métricas cuantitativas

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **fod_id** | INTEGER (PK) | Identificador único del incendio (heredado de la data normalizada) |
| **fecha_key** | INTEGER (FK) | Llave foránea hacia `dim_fecha`. Formato de fecha AAAAMMDD |
| **region_key** | INTEGER (FK) | Llave foránea hacia `dim_region` |
| **causa_key** | INTEGER (FK) | Llave foránea hacia `dim_causa` |
| **administracion_key** | INTEGER (FK) | Llave foránea hacia `dim_administracion` |
| **clase** | VARCHAR(1) (FK) | Llave foránea hacia `dim_tamano` e indica la clase de tamaño (A-G) |
| **ubicacion_key** | INTEGER (FK) | Llave foránea hacia `dim_ubicacion` |
| **nwcg_unit_id** | VARCHAR | **Atributo degenerado**. Código de la unidad de NWCG que reportó el incendio y se mantiene aquí para evitar crear una dimensión de un solo campo |
| **incendios** | INTEGER | **Métrica**. Siempre vale 1 y permite hacer conteos (SUM) en herramientas BI |
| **acres** | DOUBLE | **Métrica**. Superficie quemada en acres |
| **tiene_contencion** | INTEGER | **Bandera (1/0)**. Vale 1 si el incendio tiene fecha de fin registrada, 0 si no. Vital para calcular promedios reales de días de contención sin sesgos |
| **dias_contencion** | INTEGER | **Métrica**. Diferencia en días entre la fecha de detección y la fecha de contención y nula si no tiene contención |

## 2. Tablas de Dimensión 

### 2.1 `dim_fecha` (Responde al Objetivo Específico 5)
Contexto temporal para análisis de estacionalidad
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **fecha_key** | INTEGER (PK) | Fecha en formato AAAAMMDD |
| **fecha** | DATE | Fecha real de descubrimiento |
| **anio** | INTEGER | Año de detección (2000-2015) |
| **trimestre** | INTEGER | Trimestre del año (1-4) |
| **mes** | INTEGER | Mes del año (1-12) |
| **nombre_mes** | VARCHAR | Nombre del mes en texto |
| **estacion** | VARCHAR | Clasificación de la fecha en Invierno, Primavera, Verano u Otoño |

### 2.2 `dim_region` (Responde al Objetivo General y a los Específicos)
Contexto territorial principal de la investigación
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **region_key** | INTEGER (PK) | Clave subrogada (ID numérico) generada por el sistema |
| **region_code** | VARCHAR(2) | Código del Área Geográfica del NIFC (ej. GB, SW) |
| **region_name** | VARCHAR | Nombre completo de la región (ej. Great Basin Area) |

### 2.3 `dim_causa` (Responde al Objetivo Específico 2)
Contexto de origen del siniestro.
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **causa_key** | INTEGER (PK) | Código original de la causa (STAT_CAUSE_CODE) |
| **causa_especifica** | VARCHAR | Descripción original (ej. Lightning, Arson) |
| **origen_causa** | VARCHAR | **Regla de Negocio**: Agrupación en "Natural" o "Humana" |

### 2.4 `dim_administracion` (Responde al Objetivo Específico 4)
Contexto de propiedad del terreno
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **administracion_key** | INTEGER (PK) | Código original del propietario + 1 |
| **propietario** | VARCHAR | Nombre detallado del dueño del terreno |
| **tipo_administracion** | VARCHAR | **Regla de Negocio**: Agrupación en "Federal", "Estatal/Local" o "Privada" mediante análisis de texto |

### 2.5 `dim_tamano` (Responde al Objetivo Específico 3)
Contexto de magnitud del incendio
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **clase** | VARCHAR(1) (PK) | Letra de la clase de tamaño oficial (A-G) |
| **rango** | VARCHAR | Descripción del rango en acres (ej. >= 5000 acres) |
| **orden** | INTEGER | Valor del 1 al 7 para ordenar los gráficos correctamente |
| **es_megaincendio** | INTEGER | **Bandera (1/0)**: Vale 1 si pertenece a la Clase G (mega-incendios), 0 para el resto |

### 2.6 `dim_ubicacion` (Responde al Objetivo Específico 1)
Contexto geográfico de alta precisión
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| **ubicacion_key** | INTEGER (PK) | Clave subrogada |
| **state_code** | VARCHAR(2) | Código del estado (ej. CA) |
| **state_name** | VARCHAR | Nombre del estado |
| **fips_5** | VARCHAR(5) | Concatenación de State_Code + Fips_Code |
| **county_name** | VARCHAR | Nombre del condado |
| **land_sq_mi** | DOUBLE | Extensión territorial del condado en millas cuadradas (excluyendo agua). Útil para calcular densidad de incendios |