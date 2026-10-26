## Limpieza, Normalización y Desnormalización de Datos

Para poder analizar la base de datos de incendios forestales en nuestro aplicativo y en los tableros de control, organizamos el procesamiento de la información en tres pasos principales:

---

### 1. Limpieza de Datos

La base de datos original traía algunos campos con información faltante y formatos que necesitaban ajustarse antes de trabajar:

* **Completar datos geográficos faltantes:** Aproximadamente un 36% de los registros no tenían el condado ni el estado asignado. Para solucionarlo, usamos las coordenadas de latitud y longitud de cada incendio y las cruzamos con el mapa vectorial de contornos de condados del censo de Estados Unidos. De esta forma pudimos identificar de manera automática a qué estado y condado pertenecía cada evento.
* **Formato de fechas:** Las fechas de la base original venían en formato Juliano, por lo que las convertimos todas al formato estándar de fecha (`DATE`).
* **Manejo de fechas de contención vacías:** Notamos que muchos incendios no tenían registrada la fecha de contención. En lugar de borrar esos registros, decidimos conservarlos e identificar los incendios que sí tenían fecha mediante una columna o bandera de control (`tiene_contencion` con valores 1 o 0), evitando perder datos importantes.

---

### 2. Normalización (Tercera Forma Normal - 3FN)

Partiendo de la tabla masiva original, organizamos la información en una base de datos relacional aplicando las reglas de la Tercera Forma Normal (3FN):

* **Eliminación de datos repetidos:** Separamos la información en tablas independientes de catálogos para evitar redundancias en datos categóricos como causas, propietarios y ubicaciones.
* **Selección de tablas relevantes:** Simplificamos el sistema conservando únicamente las tablas y campos que realmente eran útiles para nuestra investigación. El modelo normalizado quedó estructurado por la tabla principal `tabla_incendios` conectada a las tablas de `tabla_geographic_area`, `tabla_propietario`, `tabla_causa_incendio`, `tabla_clase_tamano`, `tabla_condado` y `tabla_estado`.

---

### 3. Desnormalización (Modelo en Estrella)

Para que las consultas en el aplicativo y en las herramientas de visualización fueran mucho más rápidas y eficientes, transformamos el modelo relacional en un **Modelo Dimensional en Estrella**:

* **Tabla de Hechos (`fact_incendio`):** Es la tabla central del modelo y contiene las métricas numéricas principales que vamos a analizar (como los acres quemados y los días de duración), manteniendo además el código de la unidad que reportó el incendio (`nwcg_unit_id`) como un atributo directo.
* **Columnas de control en la tabla de hechos:** Para no alterar los cálculos matemáticos y promedios, colocamos la columna `tiene_contencion` directamente en la tabla de hechos (asignando un "1" si el incendio tiene fecha de fin y un "0" si no la tiene).
* **Uso de IDs numéricos:** Para no repetir textos largos dentro de la tabla de hechos y hacer que las consultas funcionen más rápido, creamos códigos numéricos (claves subrogadas) para conectar con cada tabla de contexto.
* **Tablas de Dimensiones:** Creamos 6 tablas alrededor de la tabla de hechos para dar contexto a los datos y agrupamos la información según nuestras reglas de análisis:
  * `dim_fecha`: Calcula la estación del año (Invierno, Primavera, Verano, Otoño) según el mes de inicio del incendio.
  * `dim_region`: Contiene las 10 regiones operativas de atención a incendios.
  * `dim_causa`: Agrupa los motivos del incendio en "Natural" (rayos) y "Humana" (el resto de las causas).
  * `dim_administracion`: Clasifica los dueños de los terrenos en tres categorías: Federal, Estatal/Local y Privada.
  * `dim_ubicacion`: Guarda el estado y el condado del incendio.
  * `dim_tamano`: Define los rangos oficiales del tamaño del incendio (de la A a la G) e incluye un campo que marca con un "1" si el evento corresponde a un mega-incendio (Clase G).
