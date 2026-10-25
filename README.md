## Arquitectura de datos

El procesamiento de los datos brutos recopilados desde Kaggle hasta su consumo analítico se estructuró en tres fases metodológicas rigurosas para garantizar la consistencia estadística y el rendimiento del aplicativo.

---

### 1. Limpieza y preparación de datos

El conjunto de datos original presentaba vacíos operativos y formatos no estandarizados que requerían corrección previa antes de cualquier modelado:

* **Imputación geoespacial:** El 36% de los registros carecían de asignación de condado y estado. Para corregirlo de forma automatizada, se cruzaron las coordenadas de latitud y longitud con los contornos vectoriales del censo de Estados Unidos, asignando el código correspondiente mediante intersección espacial.
* **Estandarización temporal:** Todos los campos de fechas originales fueron transformados de formato Juliano a formato estándar de fecha.
* **Tratamiento de la ausencia de contención:** Se identificó que un porcentaje elevado de incendios no registraba fecha de contención. En lugar de descartar los eventos, se optó por conservarlos en el universo de registro incorporando una bandera binaria de control, lo que permite medir con precisión la cobertura operativa de las agencias sin alterar los cálculos métricos.

---

### 2. Normalización 

A partir de la tabla masiva de la fuente original, se diseñó una primera base de datos relacional normalizada para asegurar la integridad referencial y aislar los dominios categóricos. 

* **Reducción de dominios:** Se depuró el esquema inicial para conservar únicamente las entidades estrictamente relevantes para la investigación, estructurando el modelo en torno a tablas clave compuestas por la tabla transaccional base y catálogos normalizados para áreas geográficas, propietarios, causas, clases de tamaño y ubicaciones.

---

### 3. Desnormalización 

Para optimizar el rendimiento de las consultas analíticas del aplicativo y los tableros ejecutivos, el modelo relacional se desnormalizó transformándose en un Esquema en Estrella:

* **Tabla de Hechos:** Almacena las métricas cuantitativas del estudio (como los acres quemados y los días de duración) junto al código de la unidad reportora como atributo degenerado.
* **Claves Subrogadas:** Para acelerar el procesamiento y reducir el peso del almacenamiento, se utilizó la función analítica de SQL para asignar identificadores numéricos secuenciales a los contextos geográficos y temporales.
* **Dimensiones analíticas:** Se construyeron seis tablas de contexto enriquecidas aplicando reglas de negocio específicas para estacionalidades, macro-categorías de causas y detección de incendios de gran magnitud.
