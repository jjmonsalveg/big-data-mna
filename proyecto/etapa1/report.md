# Etapa 1. Selección y caracterización del dataset

**Materia**: Análisis de grandes volúmenes de datos
**Institución**: Tecnológico de Monterrey
**Trimestre**: 3
**Estudiante**: Jonathan Monsalve
**Fecha**: 2026-05-03

## 1. Tema del proyecto

Análisis exploratorio del dataset NYC TLC Yellow Taxi (años 2024 y 2025) con vistas al modelado supervisado en etapas posteriores del proyecto. La caracterización busca documentar volumen, esquema, distribuciones y problemas de calidad del dataset antes de cualquier limpieza o entrenamiento.

## 2. Selección y justificación del dataset

Se eligió el dataset público **NYC TLC Yellow Taxi Trip Records**, publicado por la New York City Taxi and Limousine Commission. Razones de la elección:

- **Tamaño**: 1.42 GB en disco (24 archivos Parquet mensuales más el catálogo de zonas en CSV). Cubre el requisito de la rúbrica (>1 GB). Un solo año pesa ~0.65 GB, motivo por el cual se incluyeron 2024 y 2025 completos.
- **Volumen**: 89,892,322 registros, suficiente para evaluar distribuciones, agregaciones y modelos sobre datos genuinamente masivos en una sesión local de PySpark.
- **Heterogeneidad de tipos**: enumeraciones (`VendorID`, `RatecodeID`, `payment_type`), identificadores zonales, marcas de tiempo, distancias y montos monetarios en distintas escalas. Esto permite ejercitar todos los módulos del curso (estadística descriptiva, calidad de datos, muestreo, visualización, modelado).
- **Dimensión temporal natural**: dos años permiten análisis comparativo interanual; el año 2026 (parcialmente publicado) queda reservado como conjunto de validación temporal (out-of-time) para la Etapa 4.
- **Reproducibilidad**: la fuente es un CDN público sin autenticación, lo que permite que el notebook corra en Google Colab y en computadoras de compañeros sin configuración adicional.

## 3. Caracterización del dataset

### 3.1 Inventario y formato

- 24 archivos Parquet mensuales (`yellow_tripdata_YYYY-MM.parquet`), aproximadamente 60 MB cada uno.
- 1 archivo CSV: `taxi_zone_lookup.csv` con 265 zonas TLC.
- Total: 1.42 GB en disco, 89.9 millones de filas.

### 3.2 Esquema y refinamiento de tipos

Los archivos de 2025 incorporan la columna `cbd_congestion_fee` (ausente en 2024). La unificación se resolvió con `mergeSchema=True` en la lectura, que rescata la columna y la deja como nula en los registros 2024.

Tras validar empíricamente los rangos efectivos de cada columna numérica, se aplicó un refinamiento de tipos (downcast) sobre el DataFrame:

- Identificadores enumerados (`VendorID`, `RatecodeID`, `payment_type`): de `int` a `tinyint` (1 byte).
- Identificadores de zona (`PULocationID`, `DOLocationID`): de `int` a `smallint` (2 bytes).
- Variables monetarias y `trip_distance`: de `double` a `float` (4 bytes).

El ahorro estimado es de ~72 bytes por fila, equivalente a ~6 GB de presión de memoria sobre el dataset completo. Este refinamiento es metodológicamente importante porque mantiene el notebook portable a entornos con memoria limitada (Colab, laptops de estudiantes) sin tener que tunear `spark.driver.memory`.

## 4. Hallazgos clave del análisis exploratorio

### 4.1 Patrones temporales de demanda

**Figura 1 (notebook sección 9.2)**: Heatmap 24x7 de conteo de viajes. Eje Y: hora del día (0 = medianoche, ascendente hasta 23). Eje X: día de la semana (lunes a domingo). Paleta coolwarm (azul = baja demanda, rojo = alta demanda). Cada celda anota su conteo en miles.

El heatmap revela un pico claro de viajes entre las 17:00 y las 19:00 de martes a jueves (más de 1 millón de viajes por celda en la combinación más cargada). Las madrugadas de lunes a jueves (2:00-5:00) son las celdas de menor actividad, mientras que los viernes y sábados muestran un segundo pico nocturno entre 22:00 y 1:00 ausente entre semana, consistente con tráfico de ocio.

### 4.2 Relación distancia-tarifa

**Figura 2 (notebook sección 9.4)**: Diagrama de dispersión de `trip_distance` (eje X, millas, rango 0-50) contra `fare_amount` (eje Y, USD, rango 0-200). Muestra de 100,000 viajes con baja opacidad (alpha=0.05) para visualizar densidad. Color azul único, puntos pequeños.

El scatter revela tres hallazgos en una sola gráfica:

1. **Correlación lineal estructural**: la nube principal sigue una pendiente cercana a USD 2.50 por milla, consistente con la tarifa por milla del medidor TLC.
2. **Banda horizontal en USD 70**: corresponde a la tarifa plana JFK-Manhattan, claramente identificable como una "moda" visual.
3. **Cluster vertical en `trip_distance = 0` con `fare_amount` > 0**: alineación densa de puntos hasta USD 100, indicador de fallas del medidor de odómetro o de viajes cancelados mal facturados. Este problema no era detectable en los estadísticos descriptivos univariados.

### 4.3 Estructura de propinas por método de pago

**Figura 3 (notebook sección 9.3)**: Boxplots comparativos de `tip_amount` (eje Y, USD) por categoría de `payment_type` (eje X). Cuatro categorías visibles: Tarjeta de crédito, Efectivo, Sin cargo, Disputa. Bigotes p5-p95, datos acotados al rango [0, USD 100]. El recuento `n` aparece debajo de cada etiqueta.

Solo los viajes con `payment_type = 1` (tarjeta de crédito) registran propinas reales: mediana de USD 3.30, rango intercuartil USD 2-5, percentil 95 ~USD 13. Las categorías Efectivo, Sin cargo y Disputa registran mediana cero por diseño del medidor: las propinas en efectivo no entran al sistema. Este es un sesgo estructural que cualquier análisis posterior de propinas debe respetar (filtrar a `payment_type = 1` o reconocer la limitación).

### 4.4 Hallazgo metodológico crítico: distorsión de Pearson por outliers

La matriz de correlación de Pearson sobre datos crudos arroja resultados contraintuitivos: `trip_distance` vs `fare_amount` r = 0.01, cuando estructuralmente debería superar 0.8. La causa es la presencia de outliers extremos (`trip_distance` máximo 398,608 millas, `fare_amount` máximo USD 863,372) que inflan la varianza y enmascaran las relaciones lineales reales. Las únicas correlaciones que sí emergen sobre datos crudos corresponden a viajes en zona de congestión central (r entre 0.47 y 0.74 entre los recargos `improvement_surcharge`, `congestion_surcharge` y `cbd_congestion_fee`) y al cluster de viajes a aeropuerto (r entre 0.41 y 0.45 entre `Airport_fee`, `tolls_amount` y `tip_amount`). Este resultado es prueba directa de que la limpieza de outliers en la Etapa 2 es indispensable antes de cualquier modelado supervisado.

### 4.5 Patrón estructural de nulos: Flex Fare

**Figura 4 (notebook sección 9.5)**: Barras horizontales de porcentaje de nulos por columna (solo columnas con nulos > 0), ordenadas de mayor a menor. Eje X: porcentaje. Eje Y: nombre de columna. Cada barra muestra su porcentaje exacto al final.

Cinco columnas (`passenger_count`, `RatecodeID`, `store_and_fwd_flag`, `congestion_surcharge`, `Airport_fee`) presentan exactamente el mismo porcentaje de nulos (17.47%, 15,703,126 filas). Una verificación explícita confirma que se trata de las mismas filas: el conteo de filas con las cinco columnas simultáneamente nulas coincide con el conteo individual de cualquiera de ellas. Si fueran nulos independientes, el conteo conjunto sería mucho menor.

Estas filas corresponden a la modalidad **Flex Fare**, introducida por TLC para alinear los taxis amarillos con plataformas de viaje por aplicación (Curb, Arro). El viaje se cotiza desde la app con tarifa acordada por adelantado y el taxímetro tradicional no se activa, por lo que los campos que produce el medidor quedan nulos por diseño operativo (no por fallo de captura). Se identifican con `payment_type = 0`, fuera del rango 1-6 del manual TLC clásico.

La consecuencia para etapas posteriores es que **no corresponde imputar** esas cinco columnas, porque inventaría telemetría que nunca existió. La estrategia es marcar una bandera `is_flex_fare = (payment_type == 0)` y dejar que cada análisis decida según la matriz siguiente:

| Tipo de análisis | Tratamiento de filas Flex Fare |
|---|---|
| Demanda por hora, día o zona | Incluir (representan demanda real) |
| Análisis del medidor (`passenger_count`, `RatecodeID`, recargos) | Excluir filtrando por `is_flex_fare = false` |
| Modelado supervisado de tarifa o duración | Incluir con la bandera como variable predictora |
| Análisis de propinas | Ortogonal: el filtro `payment_type = 1` ya excluye Flex Fare |

## 5. Problemas de calidad y correcciones propuestas

La siguiente tabla sintetiza los hallazgos consolidados en la sección 8 del notebook (10 problemas detectados, presentados aquí en versión ejecutiva).

| # | Problema | Corrección propuesta |
|---|---|---|
| 1 | `trip_distance` con valores físicamente imposibles (máx. 398,608 millas) | Filtrar al rango [0, p99.9] o tope físico de 200 millas |
| 2 | Variables monetarias con outliers de seis cifras (máx. USD 863,372) | Filtrar `fare_amount` y `total_amount` al rango [0, p99.9] |
| 3 | `passenger_count` con valores fuera del rango documentado (0, 7, 8, 9) | Imputar a 1 (moda y mediana del dataset) |
| 4 | 59 timestamps fuera de 2024-2025 (años 2002, 2007-2009, 2023, 2026) | Filtrar al rango temporal declarado |
| 5 | 21 registros 2024 con `cbd_congestion_fee` no nulo (cargo vigente desde 2025-01-05) | Imputar a 0.0 |
| 6 | Nulos estructurales en `cbd_congestion_fee` (46.4% del total, columna ausente en 2024) | Imputar a 0.0 para registros 2024 |
| 7 | Patrón sincronizado de nulos Flex Fare en cinco columnas (17.47%, 15.7 M filas verificadas como las mismas) | Marcar bandera `is_flex_fare = (payment_type == 0)`; no imputar; tratamiento por análisis (ver sección 4.5) |
| 8 | Zonas placeholder 264 y 265 (Unknown, N/A) en el top-10 de origen y destino | Excluir de análisis geográficos; conservar en agregados globales |
| 9 | Matriz de Pearson distorsionada por outliers (r ≈ 0.01 entre distancia y tarifa) | Recalcular tras correcciones 1, 2, 3 y 10; comparar con Spearman |
| 10 | Viajes con `trip_distance = 0` y `fare_amount > 0` (cluster vertical en scatter) | Filtrar o imputar distancia desde tarifa (~USD 2.50/mi) |

Las correcciones 1, 2, 4 y 10 son destructivas (filtran filas) y se aplicarán de forma conjunta para verificar que la pérdida no exceda el 1% del dataset. Las correcciones 3, 5, 6 y 7 son imputaciones o banderas y no afectan el tamaño del dataset.

## 6. Conclusión y siguientes pasos

El dataset es viable para el proyecto: cumple holgadamente el requisito de tamaño, ofrece heterogeneidad de tipos suficiente para los módulos posteriores, y los problemas de calidad detectados son tratables con correcciones documentadas. El año 2026 queda reservado para validación temporal en la Etapa 4.

El siguiente paso es la Etapa 2 (Preprocesamiento), donde se aplicarán las correcciones de la sección 5 y se cuantificará el porcentaje de filas removidas. Solo después de esa limpieza tiene sentido recalcular la matriz de correlación y proceder al modelado.

## Referencias (APA 7)

- Apache Software Foundation. (2025). *Apache Spark 4.1.1 Documentation*. https://spark.apache.org/docs/latest/
- New York City Taxi and Limousine Commission. (2025). *TLC Trip Record Data*. https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- Wickham, H. (2014). Tidy data. *Journal of Statistical Software*, 59(10), 1-23. https://doi.org/10.18637/jss.v059.i10

## Declaración de uso de inteligencia artificial

En la elaboración de este trabajo se utilizó Claude Code (Anthropic) como asistente para la estructuración del notebook, la redacción de código PySpark y la consolidación del análisis exploratorio en formato de reporte. Todas las celdas del notebook fueron revisadas y ejecutadas por el estudiante en su entorno local; las decisiones metodológicas (selección del dataset, cobertura temporal, criterios de corrección, métricas a reportar) fueron tomadas por el estudiante. La asistencia de la herramienta se limitó a la implementación técnica y a la sugerencia de buenas prácticas; los hallazgos, las interpretaciones y las conclusiones son responsabilidad del autor.
