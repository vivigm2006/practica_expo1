## Arquitectura de datos: Limpieza, Normalización y Modelado

El procesamiento de los datos brutos recopilados desde Kaggle hasta su consumo analítico se estructuró en tres fases metodológicas rigurosas para garantizar la consistencia estadística y el rendimiento del aplicativo.

---

### 1. Limpieza y preparación de datos
El conjunto de datos original presentaba vacíos operativos y formatos no estandarizados que requerían corrección previa antes de cualquier modelado:
* **Imputación geoespacial:** El 36% de los registros carecían de asignación de condado y estado. Para corregirlo de forma automatizada, se cruzaron las coordenadas de latitud y longitud con los contornos vectoriales (*ShapeFiles*) del censo de EE. UU., asignando el código FIPS correspondiente mediante intersección espacial[cite: 1].
* **Estandarización temporal:** Todos los campos de fechas originales fueron transformados de formato Juliano a formato estándar `DATE`[cite: 1].
* **Tratamiento de la ausencia de contención:** Se identificó que un porcentaje elevado de incendios no registraba fecha de contención. En lugar de descartar los eventos, se optó por conservarlos en el universo de registro incorporando una bandera binaria (`tiene_contencion`), lo que permite medir con precisión la cobertura operativa de las agencias sin alterar los cálculos métricos[cite: 5].

---

### 2. Normalización (Esquema Relacional 3FN)
A partir de la tabla masiva de la fuente original, se diseñó una primera base de datos relacional normalizada para asegurar la integridad referencial y aislar los dominios categóricos[cite: 2]. Se depuró el esquema inicial para conservar únicamente las entidades estrictamente relevantes para la investigación, estructurando el modelo en torno a la tabla transaccional base `tabla_incendios` y catálogos normalizados de áreas geográficas, propietarios, causas, clases de tamaño y ubicaciones[cite: 2].

---

### 3. Desnormalización (Modelo Dimensional en Estrella)
Para optimizar el rendimiento de las consultas analíticas, el modelo relacional se desnormalizó transformándose en un **Esquema en Estrella**:
* **Tabla de Hechos (`fact_incendio`):** Almacena las métricas cuantitativas del estudio (acres quemados y días de duración) junto al código de la unidad reportora (`nwcg_unit_id`) como atributo degenerado[cite: 3].
* **Claves Subrogadas:** Para acelerar el procesamiento y reducir el peso del almacenamiento, se utilizó la función analítica `DENSE_RANK()` en SQL para asignar identificadores numéricos secuenciales a los contextos geográficos y temporales[cite: 3].
* **Dimensiones analíticas:** Se construyeron 6 tablas de contexto enriquecidas (`dim_fecha`, `dim_region`, `dim_causa`, `dim_administracion`, `dim_ubicacion` y `dim_tamano`) aplicando reglas de negocio para estacionalidades, macro-categorías de causas y detección de mega-incendios[cite: 3].
