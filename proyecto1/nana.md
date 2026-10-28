# Diccionario de Datos - Modelo Desnormalizado

Este documento detalla el significado de las columnas creadas en la capa analítica (Modelo Estrella) a partir de la data normalizada.

## Tabla de Hechos (`fact_incendio`)
*   **fod_id**: Identificador único del incendio.
*   **fecha_key**: Clave foránea a la dimensión de tiempo (formato AAAAMMDD).
*   **region_key**: Clave foránea a la región NIFC.
*   **causa_key**: Clave foránea a la causa de origen.
*   **administracion_key**: Clave foránea al tipo de propietario del terreno.
*   **clase**: Clave foránea al tamaño del incendio (A-G).
*   **ubicacion_key**: Clave foránea a la ubicación geográfica (Estado/Condado).
*   **nwcg_unit_id**: Atributo degenerado (código de la unidad reportante).
*   **incendios**: Métrica que vale 1 (se usa para hacer conteos con SUM).
*   **acres**: Métrica de superficie quemada.
*   **tiene_contencion**: Bandera (1 o 0). Indica si el incendio tiene fecha de fin registrada.
*   **dias_contencion**: Métrica del tiempo en días que tardó en contenerse el incendio.

## Tablas de Dimensión
*   **`dim_fecha`**: Incluye año, trimestre, mes y la estación del año (Invierno, Primavera, Verano, Otoño).
*   **`dim_region`**: Contiene el código y nombre de las 10 regiones del NIFC.
*   **`dim_causa`**: Divide la causa específica (ej. rayo) en origen "Humana" o "Natural".
*   **`dim_administracion`**: Agrupa el propietario original en Federal, Estatal/Local o Privada.
*   **`dim_tamano`**: Define los rangos A-G y marca con "1" los mega-incendios (Clase G >= 5000 acres).
*   **`dim_ubicacion`**: Contiene el nombre del estado, condado y la superficie de tierra (`land_sq_mi`) en millas cuadradas.